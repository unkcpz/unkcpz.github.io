+++
title="GSoC Logs: plumpy"
date=2020-04-29
Tags=["GSoC", "AiiDA"]
Category=["Note"]
lastmod=2020-06-05
+++

## Description
This page contains logs about difficulties and short details of each PR in plumpy
migration part of project. Each sections are tagged by date, include the short descriptions
of the PR or the difficulties I faced in the project.

## 28th-April
The first [PR](https://github.com/aiidateam/plumpy/pull/151) which attempt to drops all the
py2 dependencies.

I did it with first remove all `six` package dependencies and all `from __future__` clause.
As tutor [suggested](https://github.com/aiidateam/plumpy/pull/151#pullrequestreview-401765724),
there are following codes also need to be modified to give up compatible
with py2:

- Classes without parent other than `object` do not need to be inherited from `object` anymore,
i.e. change  `class Sub(object):` to `class Sub:`
- `types.SimpleNamespace` is built in py3, therefore no need to implement it by ourselves.
- Get rid of `SavableFuture._tb_logger` in `plumpy.persistence.py` and related functionality

#### What is `types.SimpleNamespace`?
`types.SimpleNamespace` is just a simple `object` subclass but provides attriute access to
its namespace.
It has no much different from `class NS: pass` but more versatile.
This provides the following advantages over an empty class[^simpleNamespace]:
- It allows you to initialize attributes while constructing the object: sn = SimpleNamespace(a=1, b=2)
- It provides a readable repr(): eval(repr(sn)) == sn
- It overrides the default comparison. Instead of comparing by id(), it compares attribute values instead.

[^simpleNamespace]: https://stackoverflow.com/questions/37161275/what-is-the-difference-between-simplenamespace-and-empty-class-definition

#### Can not get rid of `SavableFuture._tb_logger`
The class `SavableFuture` is inherited from `tornado.concurrent.Future`
The `_tb_logger` still be used in tornado's Future. So It can only be
get rid of after remove the need of tornado Future.

## May-16

#### The idea behind communicator and process event loop

The bird-view review is based on the discussion https://github.com/aiidateam/plumpy/pull/150#issuecomment-621705277

In this section, code related examples in plumpy package and aiida_core is given for clarification.
The processes are running at the event loop of what we call worker thread.
Theoretically, in asyncio context, one thread only has one event loop running, and
processes running in that event loop asynchronously, aka non-blocking. When one process
running to a step asynchronously, it is running and other control signals are
still able to send to and received from the running process.

The reason that process working this way is that the process in plum did not do
computational resource cost jobs, it just serves as a record and state machine
of processes. All steps running in the process are IO based actions.

The communicator provided by kiwi is used to control the plum process.
It's OK if one process blocks all the others for quite some time while it is in a step,
however it should not block communications with RabbitMQ because this may
result in RabbitMQ thinking that this client (and all its processes) have died.
Therefore, to get rid of this, the communicator should be run at a different
thread from the process one, and RmqThreadCommunicator provided in kiwi API
work for this purpose. However, this cause another question which the callback
is invoked in though the scheduler in the communicator lead to the callback function
therefore is running in the communicator's event loop. We need schedule this
callback task in the loop of process' worker thread. So I use `asyncio.run_coroutine_threadsafe`
to schedule it in the correct loop.

There are three kinds of controller case, one uses rpc subscriber to pause, play, and kill
the process. This rpc subscriber is added in the communicator when the process initialized.
Similarly, the broadcast subscriber is added when processes initialized. The broadcast subscriber
is used when we need pause, play, and kill process one command for all.
The task actions such as launch, execute and continue processes are invoked without
initialize the process. That means we have no subscriber with over threads scheduler.
The subscriber has to be added manually before the process initialized.

Let's see the code of plumpy and aiida_core. In plum, the action invoke callback
is scheduled by method `_schedule_rpc` of Process class. Therefore, when process
initialized and running in the even loop, it can be controlled by the process_controller
over thread communicator. While as for the task actions, aiida_core add this subscriber
with `ProcessLauncher` in method `create_daemon_runner` of `manager` class, right after
the runner is created. Corresponding to that, to testing this behavior in plum,
a `LoopCommunicator` is used to converting a callback being scheduled to the right thread.
It should be noticed that all thread transition procedure is done by `plumpy.futures.create_task`.

As summary, there are only to place where the code call `run_coroutine_threadsafe` to correctlly
schedule the callback to event loop. One in `_schedule_rpc` and one in `create_task`.

## 26th-May

#### Adding doc to plum

The branch named `gsoc-doc` will include the commits of docstring and documentation
change of plum.
The documentation is plan to consist by three part. a) The docstring of often used
functions, which will show up in the API doc, primary for developers. b) The self-explain examples. These
examples are primary for ultimate users. By learning from these examples, users can
quickly know how to create their process and control the processes by process_controller.
c) The documentation is for both developers and users. Held at readthedocs, the
documentation will contain the content of how to install the plum, how to use plum
to create and control the process, and how to embed the plum into your own project,
and for developers, give some important concepts about why the kiwi was needed to
control the process from another thread etc. For users, the concepts part will contains
some statement about the properties of the processes in plum, such as how to save
and load a persist process, how to create process with desired namespace and the
properties of namespace feature. Last but not least, tell user how to create the
workflow which is also a process but has the abilities to define the logical running flow
in it.

Today, I writing the introduction section of plumpy, and re-organized the structure
of the documentation. Adding and independent the section 'Controller'.

#### Some concepts

- INPUTS_RAW v.s. INPUTS_PARSE in BundleKeys.

#### Some minor changes

- remove event from state running in resume method of `Process`


## 29th-May

#### Accurate use the event loop

When should I create a new event loop? When should I use `set_event_loop`. Make
a summary here, and describe the whole story use my own words.

...


## 3rd-June

#### loop.close() v.s. loop.stop() of asyncio event loop

You can image that `loop.close()` and `loop.stop()`, they are similar to next song and stop this song. If you don't need this song, so next one.

`loop.stop()` is used with `run_forever()` in order to stop the forever running loop.

#### Control the loop after using `nest_asyncio`

Have to reset loop to the original loop after `run_until_complete()`. Maybe
a bug in `nest_asyncio`

#### `nest_asyncio` conflict with ContextVar

Here, I test contextvar working with `nest_asyncio`, and find
contextvar save the variable in a same loop. If I running the nested process
in the same loop:

```python
def test_process_nested(self):
    """
    Run multiple and nested processes to make sure the process stack is always correct
    """
    loop = asyncio.get_event_loop()
    class StackTest(plumpy.Process):

        def run(self):
            pass

    class ParentProcess(plumpy.Process):

        def run(self):
            StackTest(loop=loop).execute()

    ParentProcess(loop=loop).execute()
```

the stacks is correctly handled. However, when I running the child process in a new
event loop:

```python
def test_process_nested(self):
    """
    Run multiple and nested processes to make sure the process stack is always correct
    """
    loop = asyncio.get_event_loop()
    class StackTest(plumpy.Process):

        def run(self):
            pass

    class ParentProcess(plumpy.Process):

        def run(self):
            StackTest(loop=asyncio.new_event_loop()).execute()

    ParentProcess(loop=loop).execute()
```
When entering the inner `StackTest` process, the `PROCESSS_STACK` is reset to default
value, which is not expected behavior. Need to find a right way to use contextvar
over different event loops, I think it is supposed to do this.
Otherwise, have to roll back to `_thread_local` approach in the old version code.


## 4th-June

#### a summary of event loop problem

I make some experiment with `nest_asyncio` yesterday, here is the report.
Before I move forward, these problems deserve more discussion.

I have create some [tests](https://github.com/unkcpz/plumpy/blob/after-nest-asyncio/test/test_nest_asyncio.py) to reproduce and point out the problems after implementing
`nest_asyncio` to enable nest process in `plumpy`.

Firstly, the event loop policy is different from the code without `nest_asyncio`.
You can check the `TestProcess_00::test_execute`, after entering and leave the process
loop, the current instance of event loop is changed to the process one (created with `new_event_loop`).
This not happened if you turn off the `nest_asyncio.apply()` in the beginning of this file.
I don't know this is a design flaw or `nest_asyncio` has to be working in this way,
some relevant discussion can be found in https://github.com/erdewit/nest_asyncio/pull/25 .

Secondly, when `nest_asyncio` comming, there seems to be some flickering running
event loops. It is also report by Martin when he running my fork code two days before.
You can check and reproduce the error with `TestProcess_01` and `TestProcess_02` for
comparison. The error says there is some event loop is running, but I turned the code
upside down and can not find which loop is running.

Thirdly, for py3.5 and py3.6 contextvar seems not working properly with `nest_asyncio`.
When entering the nested loop, the variable set be contextvar is wipe out in the
inner loop. The `contextvar` is not supported for py<3.7, so may since I use
 the `aiocontextvar` to provide ContextVar.

 Finally, I have to admit that I make a big mistake about using `@pytest.mark.asyncio`
 under the class of `unittest.TestCase`, the test wrapped inside will always passed.
 So I am not sure the rmq part is working as expected since the tests is not 'actually' passed.


## 5th-June

#### poll process interval problem and as a backup strategy
 I am not very sure but I think _poll_calculation is the only mechanism here rather than the backup for the rmq broadcast. Since the on_process_finished callback is only registered in _poll_calculation but not to rmq.
But for sure, get_calculation_futuire uses rmq broker and sets poll mechanism as backup, however this method is not called elsewhere in the code base.

Or did I get it wrong?

#### Discuss and temporary solve the loop reentrant problem

The event loop in asyncio is not designed to be reentrant, ref https://bugs.python.org/issue33523

With the help of `nest_asyncio` I am allowed to be running the loop inside the loop,
but

1. I need at the same loop.
2. Event loop need to set back after the nest one is done(this seems like a bug of `nest_asyncio`)

## 17th-June

#### Savable future inherit asyncio future

by override the `load_members` method

#### asyncio future has is own loop

Describe the issue:

how to:

## 24th-June

#### how to get the exception from `kiwipy.capture_exceptions`?

```python
async def run_task():
    with kiwipy.capture_exceptions(future):
        assert asyncio.iscoroutine(coro())
        future.set_result(await coro())
```

When does exception occurs and how it get captured then displayed?
