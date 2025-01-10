+++
title="GSoC Logs: aiida_core"
date=2020-05-06
Tags=["GSoC", "AiiDA"]
Category=["Note"]
lastmod=2020-06-09
+++

## Description
Here I record the logs of exploring, changing of aiida_core code base.
The post is organized in date.

## May-06

#### The project entry point
I explore the code base of aiida_core, to decide where to kick off gsoc project.
There are lots of code involved with tornado, and I divide them into following several categories.

1. The code interact with `plum`
2. The component that need asynchronous supported natually, e.g. `aiida/engine/transports.py`
3. coroutines are running at the event loop of runner object.

#### transports
The `engine/transports.py` could be the good starting point.

The transports is a component in aiida_core used for handle the connection with
to the client computer. In principle, it may cost some time to open the link to
the computer, therefore it should not be blocked. That is why asynchronous programming
intervened.

`TransportQueue` object has its own loop, if not set, use the event loop of the main thread
i.e. `IOLoop.current()` . Then all the coroutines of transport tasks are running in this loop.

#### improved.
In test `tests/engine/test_work_chain.py`, the `inspect.stack` is now support nametuple to
inspect the function name. Therefore, can be improved for `python>=3.5`

#### `add_callback` v.s. `ensure_future` in process async test
Now I change process test in plum by launching a process throughly asynchronous
with `asyncio.ensure_future`. The previous implementation is adding a callback
function in the loop and then launch the loop. Here with pytest asyncio, I
have the whole test function running in a loop by decorate the test with
`@pytest.mark.asyncio`. As a result, I cannot add callback in the current loop, while
have to launch the coroutine by `ensure_future`. This is not wrong in the situation, but
this implementation is not compatible with the process test case in aiida_core which
add the function by `loop.add_callback` and then running the loop.

So my question is which implementation is good? Can I use the same approach in aiida_core?
Check the two types of implementation at [here](https://gist.github.com/unkcpz/c2221120d8b7dca748325ce1391d4b57)

My understanding is that ensure_future is register a coroutine to be run in the future.
`ensure_future` is the asynchronous counterpart of the `call_soon` which for running a
synchronous function in event loop. And to run function in the event loop of other
thread, `call_soon_threadsafe` and its asynchronous counterpart `run_corountine_threadsafe`
are used.

Please check https://cheat.readthedocs.io/en/latest/python/asyncio.html for more information.

## May-07

#### asynchronous of `CalculationFuture`

The `CalculationFuture` object only get set when the calculation is finished.
There are two ways to get the result of `CalculationFuture` set. One is by communicating over
RabbitMQ broadcast, the other is by polling and waiting for poll interval before check whether
is the result set.

`_poll_calculation` is a coroutine since it is counting and check the state of future,
it is asynchronous with process, they are running in the same event loop of runner.

#### investigate `exponential_backoff_retry`

When a function is running by using `exponential_backoff_retry`, the function
is first change to a coroutine if is not. Then the function is retry until it
is successfully run. The `exponential_backoff_retry` itself is a coroutine since
it should not block other steps of process, and it is
run in an event loop. Every time the function is retried, `exponential_backoff_retry`
adds a asynchronous counter to count the time to wait the function to be run.
The time of counter is exponentially increased. Therefore, it is named `exponential_backoff_retry`.

#### Logger configuration

When changing tornado to asyncio, the logger of asyncio should also be implement.
For reference, https://docs.python.org/3/library/asyncio-dev.html#logging
Just setting loggers in file `aiida/common/log.py` and `aiida/manage/configuration/options.py` .

And this seems like the first thing should be done before refactoring from tornado to asyncio.

#### different between `launch_process` and `continue_process` of `ProcessLauncher`

`continue_process` receive pid of process as input and running the process. The pid
is stored in some media of a persist process. `launch_process` directly running
the incoming process, by initialize the process class to corresponding object.

Here, I can try to add a test for redefined `_continue` function in aiida `ProcessLauncher`.

## May-08

#### Why use `raise gen.Return(result)` in process_function

In decorator `process_function` which wrap the turn the function into
a FunctionProcess. If process is executed not in a daemon but in a local runner,
aiida is listen to the signal of interrupt and handle the signal with `kill_process`
function which raise a critical log and kill the running process elegantly. However,
here in the `kill_process`, the result is returned by `raise gen.Return(result)`. I am
confused that this function is not a coroutine, why use such syntax? Can we directly return
the result?  

#### Transport tasks

The mechanism that allowed engine connecting to the remote machine is called transport,
and the transport tasks are the tasks performed in transport stage. The transport tasks
happened in `CalcJob` process. First of all, `CalcJob` overwrite the `run` method of
Process class. It enter the UPLOAD state by `plumpy.Wait` when run by `run` method. Then after UPLOAD
state, it enter the SUBMIT and then UPDATE and then RETRIEVE state step by step.

The state transition in Waiting state of `CalcJob` is done by `create_state` methods which
are defined in function `upload`, `submit`, `update` and `retrieve`. Everytime the process
got its change to run, it will check its current state and tries to move to next state.

#### Question: different between `submit` functions of runner and launch module.

There are two similar `submit` method both in `aiida/engine/launch` module and
`aiida/engine/runner`. What's the different? And when to use which?

Please check: https://aiida.readthedocs.io/projects/aiida-core/en/latest/working/workflows.html#submitting-sub-processes
The runner submit is used in subworkflow submitting specifically. In work chain, the upper process
should wait until the subprocess is surely finished, and the future to resolved, before continue.

Although I get the explanation of their difference in design, I still unable to find the
real implementation different between them. They both run with `controller.continue_process`
with `nowait=False` and `no_reply=True`.

#### up bound the number of jobs in the schedule queue

This is the requirement mentioned in the [issue#88](https://github.com/aiidateam/aiida-core/issues/88).
Even the user case is not very common, to extend the usage of the software is always seems
like not a bad thing. After look into the code base, I assumed this functionality should be
added in `aiida/engine/processes/calcjobs/task.py::task_submit_job`. Before actually
submit the uploaded job by `execmanager`, inspect the number of jobs queued at the
scheduler of a specific user. Only submit the job when the number limitation is unreached.

## May-09

#### analyse the code of circus part in aiida_core

One thing to sure, is from start to end, only one circus watcher is
created and used and spawn many processes(system) for running processes(AiiDA).

Here are something I am not fully understanded.
1. Are the daemon processes start by circus independent? Can they communicate with other
processes? And how?
2. `verdi` command launch aiida processes to daemon, how circus balance load them?


## May-19

#### add new feature so that specific type node can be quickly spot in the graph

In order to find the initial configuration of a final most stable structure in my
field research, it is easy to use AiiDA and pick the initial configuration from
the provenance graph. However, sometimes when the workflow is very complex, it is
not very easy to recognize the desired nodes from provenance graph. To overcome
this, I simply add an option `--target-cls` to `verdi node graph generate` to
colored the nodes with expected class and left other types of nodes decolorized.

The details can be found in https://github.com/aiidateam/aiida-core/pull/4081 .
This PR also fix the origin node highlight [feature](https://github.com/aiidateam/aiida-core/issues/3718).

## May-23

#### Some tests are failed locally

Running the tests with the latest version (5e02e164b) of aiida_core, I got some tests
not passed. Some of the are related to the asynchronous programming, therefore,
necessarily to be fixed before head. The failed tests are:

```
=============================================== short test summary info ================================================
FAILED tests/cmdline/commands/test_daemon.py::TestVerdiDaemon::test_daemon_restart - AssertionError: SystemExit(<Exit...
FAILED tests/cmdline/commands/test_daemon.py::TestVerdiDaemon::test_daemon_start - AssertionError: 'daemon-error-not-...
FAILED tests/cmdline/commands/test_import.py::TestVerdiImport::test_import_old_url_archives - AssertionError: SystemE...
FAILED tests/cmdline/commands/test_import.py::TestVerdiImport::test_import_url_and_local_archives - AssertionError: S...
FAILED tests/sphinxext/test_workchain.py::test_workchain_build - assert 2 == 0
FAILED tests/sphinxext/test_workchain.py::test_broken_workchain_build - assert 'The broken workchain says hi!' in "\n...
FAILED tests/transports/test_all_plugins.py::TestBasicFunctionality::test_is_open - tests.transports.test_all_plugins...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_chdir_to_empty_string - tests.transports...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_dir_copy - tests.transports.test_all_plu...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_dir_creation_deletion - tests.transports...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_dir_permissions_creation_modification - ...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_dir_reading_permissions - tests.transpor...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_isfile_isdir_to_empty_string - tests.tra...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_isfile_isdir_to_non_existing_string - te...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_listdir - tests.transports.test_all_plug...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_listdir_withattributes - tests.transport...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_makedirs - tests.transports.test_all_plu...
FAILED tests/transports/test_all_plugins.py::TestDirectoryManipulation::test_rmtree - tests.transports.test_all_plugi...
FAILED tests/transports/test_all_plugins.py::TestPutGetFile::test_put_and_get - tests.transports.test_all_plugins.run...
FAILED tests/transports/test_all_plugins.py::TestPutGetFile::test_put_get_abs_path - tests.transports.test_all_plugin...
FAILED tests/transports/test_all_plugins.py::TestPutGetFile::test_put_get_empty_string - tests.transports.test_all_pl...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_copy - tests.transports.test_all_plugins.run_for_al...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_get - tests.transports.test_all_plugins.run_for_all...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_put - tests.transports.test_all_plugins.run_for_all...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_put_and_get - tests.transports.test_all_plugins.run...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_put_and_get_overwrite - tests.transports.test_all_p...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_put_get_abs_path - tests.transports.test_all_plugin...
FAILED tests/transports/test_all_plugins.py::TestPutGetTree::test_put_get_empty_string - tests.transports.test_all_pl...
FAILED tests/transports/test_all_plugins.py::TestExecuteCommandWait::test_exec_pwd - tests.transports.test_all_plugin...
FAILED tests/transports/test_all_plugins.py::TestExecuteCommandWait::test_exec_with_stdin_filelike - tests.transports...
FAILED tests/transports/test_all_plugins.py::TestExecuteCommandWait::test_exec_with_stdin_string - tests.transports.t...
FAILED tests/transports/test_all_plugins.py::TestExecuteCommandWait::test_exec_with_stdin_unicode - tests.transports....
FAILED tests/transports/test_all_plugins.py::TestExecuteCommandWait::test_exec_with_wrong_stdin - tests.transports.te...
FAILED tests/transports/test_ssh.py::TestBasicConnection::test_auto_add_policy - paramiko.ssh_exception.NoValidConnec...
FAILED tests/transports/test_ssh.py::TestBasicConnection::test_no_host_key - paramiko.ssh_exception.NoValidConnection...
========================= 35 failed, 1438 passed, 16 skipped, 33 warnings in 261.39s (0:04:21) =========================
```

## 1st-June

#### refactoring `InterruptableFuture` and `interrupt_task`

`InterruptableFuture` is the class defined in file `aiida/engine/utils.py`.

#### unittest mock the presubmit of CalcJob

What is `mock.patch` of unittest used for?

#### Getting stuck in WorkChain part

The problem is when running nested WorkChain, the inner workchain is
run but not returned to set Context variable. After several hours debugging, I am sure
the inner workchain is reaching the `Finished` state, but, then the event loop
is block by something. Can not figure out why.

#### Only one event loop bond to the thread

Maybe the same reason which caused the above problem. When outer most process is running,
a process is started and running, then no other event loop is allowed to be run.

To clarify my guess, the following test case passed:

```python
def test_process_nested(self):
    """
    Run multiple and nested processes to make sure the process stack is always correct
    with process in new event loop.
    """
    expect_true = []

    class StackTest(plumpy.Process):

        def run(self):
            # TODO: unexpected behaviour here
            # if assert error happend here not raise
            # it will be handled by try except clause in process
            # is there better way to handle this?
            expect_true.append(self == Process.current())

    class ParentProcess(plumpy.Process):

        def run(self):
            expect_true.append(self == Process.current())
            StackTest(loop=events.new_event_loop()).execute()

    ParentProcess().execute()
```

As we can expected, it is not working for asyncio.
For now, I find `nest_asyncio` patch package can be used to tackle this
problem. But not sure is it the ultimate way?

#### Why process_class build from FunctionProcess pass runner


## 2nd-June

#### Ha! My first code review

Thanks for the chance provided by tutor [Leopold](https://github.com/ltalirz),
this is my first code review in `aiida_core` https://github.com/aiidateam/aiida-core/pull/3977

In this PR, new feature yaml config from online url is added.

#### Details of single event loop problem

To recap the problem I encountered which blocks me to go further. Different from
`tornado<5.0` the event loop of asyncio (`tornado>=5.0` also use `asyncio` event loop)
did not support running nest event loop which will raise a `RuntimeError`:

```
RuntimeError: This event loop is already running in python
```

Please reference of https://stackoverflow.com/questions/46827007/runtimeerror-this-event-loop-is-already-running-in-python
for more details of this problem itself.

In `execute` class function of `Process`, an asynchronous call of `step_until_terminated`
is scheduled in the non-async code here and wait for the result until it is done.
There seems not way to schedule more asynchronous calls here and wait, since the main
thread is block by the last `loop.run_until_complete`.
As mentioned above, `plumpy` uses this feature for nesting and execute process.
This pattern also occurs in simple workflows and workchains of `aiida_core`,
where the calcfunction `add` and `mul` are executed inside the `add_mul_wf` workflow:

```python
@calcfunction
def add(data_a, data_b):
    return data_a + data_b

@calcfunction
def mul(data_a, data_b):
    return data_a * data_b

@workfunction
def add_mul_wf(data_a, data_b, data_c):
    return mul(add(data_a, data_b), data_c)

result, node = add_mul_wf.run_get_node(orm.Int(3), orm.Int(4), orm.Int(5))

self.assertEqual(result, (3 + 4) * 5)
self.assertIsInstance(node, orm.WorkFunctionNode)
```

Patch package `nest-asyncio` can be used to overcome this, but I am not sure about it:

- Is there good way to do this without using `nest-asyncio`?
- The nested process should be running in the same event loop of its parent process
or a new event loop?

As for the second question, I did some experiment, and here are some ideas
and my understanding of the code base. In the `add_mul_wf` above, when a new
`FunctionProcess` is created and run which `run_get_node`, a new runner is
created from scratch with new_event_loop by `manager.create_runner`. It shows that in old code nested
process is running in a different event loop. With the help of `nest-asyncio`,
the processes can also run in a single event loop say event loop got by `asyncio.get_event_loop()`,
only thing should notice is when all tasks are done in the loop, it will be closed automatically.

If I am missing something please point it out.

## 8th-June

#### Replace tornado with asyncio in rmq module

after that in engine test cases only:L

```
FAILED tests/engine/test_manager.py::TestJobManager::test_request_job_info_update - TypeError: coroutines cannot be u...
FAILED tests/engine/test_runners.py::TestWorkchain::test_call_on_calculation_finish - AttributeError: '_UnixSelectorE...
FAILED tests/engine/test_utils.py::TestInterruptable::test_interrupt - TypeError: a callable object was expected by c...
```
left.

#### communicator hang up when disconnected

The problem can be reproduce with aiida_core  https://github.com/unkcpz/aiida_core/tree/gsoc-rmq and plumpy https://github.com/unkcpz/plumpy/tree/gsoc-asyncio (and asyncio kiwipy).  By running test case

```
$ pytest tests/engine/test_rmq.py::TestProcessControl::test_kill -sv
```

And also same problems for `TestProcessControl::test_pause` .. But with process finished without exception for example `TestProcessControl::test_launch_with_inputs`  the `tearDown`  will not timeout.

Now, I also test with following dependent matrix: `aiida_core` https://github.com/unkcpz/aiida_core/tree/gsoc-rmq and plumpy https://github.com/unkcpz/plumpy/tree/gsoc-asyncio and `kiwipy<0.6.0`.  And there is no exception no hang up.

it is more certain that the new version kiwipy may have some incompatible features

#### call_at only receive callback

How to add coroutine to `call_at` (and `call_soon`, `call_later`)?

just by pass the coroutine to `asyncio.create_task` (ensure_future for py<3.7)

```python
async def coro():
  await asyncio.sleep(1.)

delay = 5.
call_at(delay, asyncio.ensure_future, coro())
```

## 9th-June

#### Tutorial test with new aiida_core

Now the aiida_core is totally migrated to asyncio. Tests beyond unittest is necessary.

--:

- Process status not updated when process finished. (Only CalcJob has this issue)
- daemon stop hang up. But after timeout, it is stopped.
