---
title: Practical state-machine runs on tokio
date: '2025-03-14'
categories:
    - post
tags:
    - rust 
    - tokio
    - oxiida
    - aysnc
---

For the scientific workflow engine my team is developing, one of the core component is an abstraction for creating a state machine that operates on Python's asyncio event loop to interact with remote HPC resources.
While there are several design issues, they are not the focus of this post. 
Here, I'm simply sharing a practical implementation of a state machine that runs asynchronously on the Rust tokio runtime.

Interestingly, this turned out to be a fairly general pattern across different fields. 
I was working on it last week, and coincidentally, people were discussing a similar topic at the berline.rs meetup, where I shared this example. 
Since it's just a prototype with no concrete implementation yet, I haven’t set up a repository but instead provided some code snippets along with explanations.

There are two nice articles worth to check about what can be a good state machine pattern in rust:

- https://tmpfs.org/blog/posts/rust-async-fsm/
- https://hoverbear.org/blog/rust-state-machine-pattern/

The Linux system itself operates as a state machine, where processes are scheduled and executed asynchronously. 
The tokio runtime can be seen as a state machine abstraction of this system-level state machine, managing task scheduling and the runtime calling `poll` to advance state transitions. 
For real-world applications, however, more specialized abstractions are required to effectively model domain-specific behavior.

The `Process` here is a state machine hold the state, and can change its state by `transition_to` call.
Most importantly, every process has its own fate when things goes well it `advanced` to the final state.
Life of the process may not always go smoth, the process may want to pause or even kill himself which is totally fine. 


```rust
pub trait Process<S: State> {
    fn state(&self) -> &S;
    fn transition_to(&mut self, state: S);
    fn is_done(&self) -> bool {
        self.state().is_done()
    }
    fn is_paused(&self) -> bool {
        self.state().is_paused()
    }
    fn kill(&mut self) -> impl std::future::Future<Output = ()>;
    fn pause(&mut self) -> impl std::future::Future<Output = ()>;
    fn advance(&mut self) -> impl std::future::Future<Output = ()>;
}
```

To run it in the tokio runtime, calling the `spawn` function will return a handler that can be used to launch, pause and kill the process.
Here the `tokio::task:LocalSet` is for ensure the process will not be stolen by other threads, because it then requires the `Process` needs to be `Send`. 
Which I don't want because I want it to be more flexible that `Process` implementation can have input, output type that do not need to be `Send`.

In the function signature, you can find the proc is passed as `&Rc<Mutex<P>>` because during run, the state of the process is mutated during run. 
I feel this is not very elegent to have lots of lock aquire statement here but let's see if there is better way to remove them.
For the prototype, fine for me.

```rust
pub fn spawn<S, P>(
    local: &LocalSet,
    proc: &Rc<Mutex<P>>,
    drive_mode: DriveMode,
) -> ProcessHandler<S>
where
    S: State,
    P: Process<S> + 'static,
{
    // channel for events
    let (event_tx, mut event_rx) = mpsc::channel::<HandlerEvent>(10);
    // channel for nudging
    let (nudge_tx, mut nudge_rx) = mpsc::channel::<()>(1);
    // channel for send state info out
    let (state_tx, state_rx) = tokio::sync::watch::channel(S::default());
    // notifier to resume
    let resume_notifier = Arc::new(Notify::new());
    let resume_notifier_cloned = resume_notifier.clone();

    let proc_clone = proc.clone();
    let join_handle = local.spawn_local(async move {
        // await for launch trigger in fire and forget modeloop
        // loop will continue to the end
        if matches!(drive_mode, DriveMode::FireAndForget) {
            let _ = nudge_rx.recv().await;
        }

        loop {
            let is_done = {
                let proc_ref = proc_clone.lock().await;
                proc_ref.is_done()
            };
            if !is_done && matches!(drive_mode, DriveMode::Step) {
                let _ = nudge_rx.recv().await;
            }

            while let Ok(event) = event_rx.try_recv() {
                match event {
                    HandlerEvent::Kill => {
                        // call proc `kill` of proc, proc need has its own implementation on what need to do to kill.
                    }
                    HandlerEvent::Pause => {
                        // call proc `pause`
                    }
                }
            }

            let is_done = {
                let proc_ref = proc_clone.lock().await;
                proc_ref.is_done()
            };
            if is_done {
                // drop receiver so no event signal can send.
                drop(event_rx);
                break;
            }

            // paused and await for the resume signal
            ...

            {
                let mut proc_ref = proc_clone.lock().await;
                proc_ref.advance().await;
                let _ = state_tx.send_replace(proc_ref.state().clone());
            }
        }
    });
    ProcessHandler {
        join_handle,
        event_sender: event_tx,
        nudge_sender: nudge_tx,
        state_receiver: state_rx,
        resume_notifier,
    }
}
```

The handler implementation is fairly simple, it send some signal using tokio channels to the interface the process exposed. 
I try to mimic the tokio interface to have also non-blocking call of an operation with `try_` as the prefix.

```rust
pub struct ProcessHandler<S>
where
    S: Clone,
{
    join_handle: JoinHandle<()>,
    event_sender: mpsc::Sender<HandlerEvent>,
    nudge_sender: mpsc::Sender<()>,
    state_receiver: tokio::sync::watch::Receiver<S>,
    resume_notifier: Arc<Notify>,
}

impl<S> ProcessHandler<S>
where
    S: State,
{
    // make sure there is way to control that the spawned task can reach to end
    pub async fn until_completion(self) {
        let _ = self.join_handle.await;
    }

    pub fn try_nudge(&self) -> Result<(), mpsc::error::TrySendError<()>> {
        self.nudge_sender.try_send(())
    }

    pub async fn nudge(&self) {
        let _ = self.nudge_sender.send(()).await;
    }

    pub async fn kill(&self) -> Result<(), Error> {
        // try to resume first to break the pauser
        self.resume_notifier.notify_one();

        self.event_sender
            .send(HandlerEvent::Kill)
            .await
            .map_err(|send_error| Error::EventChannelClosed(send_error.0))
    }

    pub async fn pause(&self) -> Result<(), Error> {
        self.event_sender
            .send(HandlerEvent::Pause)
            .await
            .map_err(|send_error| Error::EventChannelClosed(send_error.0))
    }

    pub fn resume(&self) {
        self.resume_notifier.notify_one();
    }

    pub fn is_done(&self) -> bool {
        self.join_handle.is_finished()
    }
}
```

The `Process` trait is for constructing more detailed process, with more bundled operations required for certain type of tasks.
For me, I'll try to support following types.

- `SchedulerProcess`: similar to AiiDA's `CalcJob`, bundle the remote file operations through transport and perform scheduler operations through SSH.
- `LocalShellProcess`: no remote communication required, but use the resources where the service is running. The process spawn using `tokio::process::Child` so it automatically support running asynchronously, but I believe it requires tense monitoring to avoid draining the resource and starving the service.
- `CloudProcess`: communicate to a k8s cluster and start customized pod on demand, i.e. cloud processes.

I provide here a dummy implementation of an arithmetic add operation process.

```rust
        #[derive(Default, PartialEq, Clone, Debug)]
        enum DummyState {
            #[default]
            Begin,
            End,
        }

        #[derive(Debug)]
        pub enum Event {
            Run,
            Pause,
            Terminate,
        }

        impl State for DummyState {
            fn is_done(&self) -> bool {
                self == &DummyState::End
            }

            // never paused
            fn is_paused(&self) -> bool {
                false
            }
        }

        impl DummyState {
            fn next(self, event: Event) -> Self {
                match (self, event) {
                    (DummyState::Begin, Event::Run | Event::Terminate) => DummyState::End,
                    (s, e) => {
                        panic!("Wrong state, event combination: state - {s:#?}, event - {e:#?}")
                    }
                }
            }
        }

        #[derive(Debug)]
        struct AddProc {
            state: DummyState,
            inp1: i32,
            inp2: i32,
            output: Option<i32>,
        }

        impl Process<DummyState> for AddProc {
            fn state(&self) -> &DummyState {
                &self.state
            }
            fn transition_to(&mut self, state: DummyState) {
                self.state = state;
            }
            fn is_done(&self) -> bool {
                self.state().is_done()
            }
            fn is_paused(&self) -> bool {
                self.state().is_paused()
            }
            async fn kill(&mut self) {
                self.state = self.state.clone().next(Event::Terminate);
            }
            async fn pause(&mut self) {
                self.state = self.state.clone().next(Event::Pause);
            }
            async fn advance(&mut self) {
                self.state = match self.state {
                    DummyState::Begin => {
                        self.output = Some(self.inp1 + self.inp2);
                        println!(
                            "Computed {} + {} = {}",
                            self.inp1,
                            self.inp2,
                            self.output.unwrap()
                        );
                        self.state.clone().next(Event::Run)
                    }
                    DummyState::End => unreachable!(),
                }
            }
        }

        impl AddProc {
            fn new(inp1: i32, inp2: i32) -> Rc<Mutex<AddProc>> {
                Rc::new(Mutex::new(AddProc {
                    state: DummyState::Begin,
                    inp1,
                    inp2,
                    output: None,
                }))
            }
        }

        let local = tokio::task::LocalSet::new();
        let proc = AddProc::new(6, 4);

        local
            .run_until(async {
                let handler = spawn(&local, &proc, DriveMode::FireAndForget);
                assert!(handler.try_nudge().is_ok());
                handler.until_completion().await;
            })
            .await;

        assert_eq!(proc.lock().await.state(), &DummyState::End);
        assert_eq!(proc.lock().await.output, Some(6 + 4));
```

