+++
title="GSoC week summary II"
date=2020-06-23
Tags=["GSoC"]
Category=["Note"]
+++

## Progress

After merge the new develop branch of `aiida_core` and new develop `plumpy` the previous problem that when closing
the communicator, the program hangs without respond is no longer show up again.
I can not reproduce the exception, so just ignore the question for the time being.

But since the starting of using `LoopCommunicator` as communicator, there are more problems bubble up to the desktop.

#### Issue 1: using `asyncio.Future` as `plumpy.Future`

Because `asyncio.Future` is used with associated loop, it can not be await in the other loop.
It is not hard to just fix it if we need the Process initiated with `SavableFuture` created
by specifying the loop:

```python
@persistence.auto_persist('_pid', '_creation_time', '_future', '_paused', '_status', '_pre_paused_status')
class Process(StateMachine, persistence.Savable, metaclass=ProcessStateMachineMeta):

    ...

    def __init__(self, inputs=None, pid=None, logger=None, loop=None, communicator=None):
        ...

        # Runtime variables
        self._future = persistence.SavableFuture(loop=self._loop)
        ...
```

However, when recreating the `SavableFuture` from saved, the loop is supposed to be
designated. Otherwise will get the exception 'RuntimeError: Non-thread-safe operation invoked on an event loop other than the current one'

Does loop need to be stored as the persister or just passed as context of `SavableFuture`?
If the loop in the `load_context` is desired, how to save and load it explicitly?

#### Issue 2: callback in `call_on_process_finish` can be any callable

Since `LoopCommunicator` is used as the communicator now, all communications over RabbitMQ
will go through `plumpy.create_task`. In `create_task` plumpy expect a coroutine to be
scheduled in the event loop, but it always not the case, the callback passed to the
method can be any callable functions coroutines, regular functions, `functools.partial`, lambda,
or even classes with `__call__` method. That is to say, a way to ensure the coroutine is needed.

By google it, find in asyncio programming, it is encourage to explicitly distinguish
the coroutine and regular functions.

```python
async def async_gettter():
    return (await http_client.get("http://example.com"))

def sync_getter():
    return asyncio.get_event_loop().run_until_complete(async_getter())

```

But not easily to convert to a coroutine with deep function calls.
For example, `inline_callback` wrap the `callback`, if callback is not a coroutine,
we need first convert it to the coroutine and await it in the `inline_callback` and then
convert `inline_callback` into a coroutine either.
The question is when porting to code to asyncio, do I need to make sure all the callbacks
passed to the `create_task` is coroutine, even if it not did any network/IO operations?
