---
title: DOM manipulation for bevy app run on web
date: '2025-11-07'
categories:
    - post
tags:
    - rust 
    - bevy
    - web
draft: true
---

Working with DOM manipulation in Rust turned out to be surprisingly fun.
As a result, I was able to write zero JavaScript code and make my Bevy app bypass the browser's default behaviors and behave exactly the same as its desktop counterpart.

Under the hood, this approach relies on `web-sys`, which provides low-level bindings to the Web APIs exposed by the browser. 
On top of that, `gloo` offers an ergonomic interface, smoothing over many of the rough edges and making common tasks like event handling and DOM interaction feel idiomatic in Rust.
In this post, I walk through a real-world problem (add drop-and-drag for my bevy app) I ran into and show how combining `web-sys` made the solution not only possible, but pleasant to implement.

## why does my Bevy app behaves differently in browser?

I'm building [vizmat](https://rs4rse.github.io/vizmat/), a molecule and crystal visualizer written with [Bevy](https://bevy.org/). 
One of the big promises of Bevy and Rust + WASM in general is "code once, run everywhere." 
In practice, this mostly holds true: the same codebase runs on linux/macos/windows AND **in the browser** with very little friction.

Until it doesn’t.

A concrete example I ran into was drag-and-drop. 
The exact same interaction that worked perfectly on desktop simply refused to behave as expected in the browser build. 
When I drag a crystal structure file into the canvas, instead of loading it, the browser download the file.

At first glance, this feels like a Bevy or WASM bug, but the real culprit is the browser itself.
Unlike native environments, browsers come with [a lot of built-in default behaviors](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Events). 
Mouse events, drag-and-drop, file handling, and focus management are all pre-wired by the browser, often in ways that conflict with how a game engine like Bevy expects to receive input. 
If these defaults aren't explicitly disabled or intercepted, they can swallow events before Bevy ever sees them.

In other words, the behavior difference isn't coming from Bevy but from the browser's DOM event system.

## How a javascript web app solves this?

Javascript is the native language of browser, it is easy to find solution to do it in javascript.
Follow the [MDN API doc](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API#drop_target) I find it can simply do following for `dragover` event.

```javascript
const element = document.getElementById("bevy-canvas");

element.addEventListener(
  "dragover",
  (event) => {
    event.stopPropagation();
    event.preventDefault();

    const dt = event.dataTransfer;

    dt.dropEffect = "copy";
    dt.effectAllowed = "all";
  },
  { passive: false }
);
```

Same for the drop event, and it can get the data in bytes for the file dropped in.

```javascript
const element = document.getElementById("bevy-canvas");
element.addEventListener(
  "drop",
  (event) => {
    event.stopPropagation();
    event.preventDefault();

    const dt = event.dataTransfer;
    const items = dt.items;

    for (let idx = 0; idx < items.length; idx++) {
      const item = items[idx];
      const file = item.getAsFile();
      const reader = new FileReader();

      reader.addEventListener("load", () => {
        const buffer = reader.result;
        const data = new Uint8Array(buffer);

        // ??? there is no sendEvent function, I need define it in rust and provide JS binding.
        sendEvent({
          type: "Drop",
          name: file.name,
          data: data,
          mime_type: file.type,
        });
      });

      reader.readAsArrayBuffer(file);
    }
  },
  { passive: false }
);
```

I can partially test this javascript code in following html.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>vizmat: a cross platform crystal visualizer</title>
    <style>
      body {
          margin: 0;
          padding: 0;
          overflow: hidden;
          background-color: #1a1a1a;
      }
      canvas {
          display: block;
          width: 100vw;
          height: 100vh;
      }
    </style>
  </head>
  <body>
    <canvas id="bevy-canvas"></canvas>
    <script type="module" src="./index.js"></script>
  </body>
</html>
```

The file is not downloaded anymore but be read and content is extract out.

However, you may have noticed that `sendEvent` is not implemented. 
Its purpose is to send data to somewhere the Rust WASM app can understand—in other words, into a Rust-side data structure.

The way to achieve this is by creating a JS binding on the Rust side. 
`serde` is needed here to serialize the data for transfer.

This is certainly doable, but it’s not my goal here. What I want is for everything to be done in Rust, so the data is already in Rust’s layout when received.

You can imagine this as having code on the frontend javascript side that retrieves data from the browser and then sends it to the Rust WASM side.

In the next section, I'll move this logic entirely to Rust, where the code lives on the Rust side and fetches data directly from the Web API, already in Rust's native format.

### direct DOM control in rust using `gloo` and `web-sys`

The caveat of using javascript is that it requires a binding implementation of `sent_event` so the data can go from browser to rust side.
Is there a way to get this data in rust layout directly? 
Yes, the solution is using `web-sys` -- the bindings for the Web APIs to directly talk to browser in rust.


```rust
let doc = gloo::utils::document();
let element = doc.get_element_by_id("bevy-canvas")?;

EventListener::new_with_options(
    &element,
    "dragover",
    EventListenerOptions::enable_prevent_default(),
    move |event| {
        let event: DragEvent = event.clone().dyn_into().expect("dynamic cast fail");
        event.stop_propagation();
        event.prevent_default();

        event
            .data_transfer()
            .expect("invalid data transfer")
            .set_drop_effect("copy");
        event
            .data_transfer()
            .expect("invalid data transfer")
            .set_effect_allowed("all");

        info!("dragover event");
    },
)
.forget();

EventListener::new_with_options(
    &element,
    "drop",
    EventListenerOptions::enable_prevent_default(),
    move |event| {
        let event: DragEvent = event.clone().dyn_into().expect("dynamic cast fail");
        event.stop_propagation();
        event.prevent_default();

        info!("drop event");

        let transfer = event.data_transfer().expect("invalid data transfer");
        let files = transfer.items();

        for idx in 0..files.length() {
            let file = files.get(idx).expect("invalid item");
            let file_info = file
                .get_as_file()
                .ok()
                .flatten()
                .expect("cannot flatten fileinfo");

            info!(
                "file[{idx}] = '{}' - {} - {} b",
                file_info.name(),
                file_info.type_(),
                file_info.size()
            );

            let file_reader = FileReader::new().unwrap();

            {
                let file_reader = file_reader.clone();
                let file_info = file_info.clone();
                EventListener::new(&file_reader.clone(), "load", move |_event| {
                    let result = file_reader.result().expect("result invalid");
                    let result = web_sys::js_sys::Uint8Array::new(&result);
                    let mut data: Vec<u8> = vec![0; result.length() as usize];
                    result.copy_to(&mut data);

                    info!("drop file read: {}", file_info.name());

                    send_event(WebEvent::Drop {
                        name: file_info.name(),
                        data,
                        mime_type: file_info.type_(),
                    });
                })
                .forget();
            }

            file_reader.read_as_array_buffer(&file_info).unwrap();
        }

        info!("dragover event");
    },
)
.forget();
```

TODO: Explain the code and mention the web-sys feature adding.
- explain why forget is needed.

The advantage is not only you don't need to write any javascript code, but you can get RAII and native asynchronous without, which I'll show as an example in appendix.

### bevy-channel

### appendix

#### RAII

#### native fearless asynchronous
