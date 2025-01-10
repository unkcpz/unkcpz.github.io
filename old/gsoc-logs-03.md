+++
title="GSoC Logs: circus"
date=2020-05-08
Tags=["GSoC", "AiiDA", "circus"]
Category=["Note"]
lastmod=2020-05-11
+++

## Description
This is the post of logs recording the questions and barriers encountered
when look into the code base of `circus`.

## May-11

#### `gen.Task` equality in high version tornado

Some function in circus depend on outdated `gen.Task`, to run
callback-based asynchronous function. The function has an argument `callback`
and when the function run with `gen.Task`, the callback is then run after the
execution of the function and the result of the callback function if returned as
`Future` object.

In higher version of tornado, the concept of callback is deprecated. Some the
programming pattern here are changed to the following code according to the
reference from https://stackoverflow.com/questions/57103331/how-to-replace-yield-gen-taskfn-argument-with-an-equivalent-asyncio-express :
```python
response = yield gen.Task(fn, request)

# is equivalent to
future = tornado.concurrent.Future()
fn(request, callback=future.set_result)
response = yield future
```

The `fn` function mentioned above is functions of `zmq.eventloop.zmqstream.ZMQStream`,
which send and receive from a non-blocking socket, using tornado. The relevant
methods are `send(msg, callback)` and `on_recv(callback)`. The callback argement here
used for assign the task to run when everytime the function `send` or `on_recv` is called.
The functions locate in file `circus/client.py::AsyncCircusClient::call` which call
the command set in the `watcher` of circus arbiter.  

#### The call timing of the method `add_done_callback`

In new version of tornado (>=5.0), the `Future` in tornado is implemented from
asyncio when available, so the behavior of method `add_done_callback` is slightly
changed. If the Future is already done, this function makes no guarantee that the
callback will be called immediately. So, use `future_add_done_callback` instead.
```python
def future_add_done_callback(  # noqa: F811
    future: "Union[futures.Future[_T], Future[_T]]", callback: Callable[..., None]
) -> None:
    """Arrange to call ``callback`` when ``future`` is complete.

    ``callback`` is invoked with one argument, the ``future``.

    If ``future`` is already done, ``callback`` is invoked immediately.
    This may differ from the behavior of ``Future.add_done_callback``,
    which makes no such guarantee.

    .. versionadded:: 5.0
    """
    if future.done():
        callback(future)
    else:
        future.add_done_callback(callback)
```

## May-21

#### `RuntimeError: Event loop is closed` for tornado < 5.0.2

Tutor Sebastiaan report that some tests failed when tornado==5.0.0.
By check that, I assume the problem was caused from the bug of tornado<5.0.2
itself. From tornado [release note 5.0.2](https://www.tornadoweb.org/en/stable/releases/v5.0.2.html),
there is a bug that will failed in close the event loop in `tearDown` setup.
Update the dependency tornado to higher version fix this. And to make sure this
is not the fluke, I run the failed test independently, so the loop close procedure
in `tearDown` setup will not take effect. Fortunately, the test passed as expected.
