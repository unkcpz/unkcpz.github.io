+++
title="GSoC Logs: Proposal"
date=2020-04-01
lastmod=2020-05-26
Tags=["GSoC", "AiiDA"]
Category=["Note"]
+++

# AiiDA: Full support of `asyncio` in `aiida-core`

## Abstract
`aiida-core` uses `plumpy` as its workflow backend and uses `circus` to daemonize its workflow manager process. However, the `plumpy` workflow library, and the circus process & socket manager have not kept up with recent developments, forcing libraries of AiiDA ecosystem to run with outdated versions of `tornado`, and making it incompatible with the latest python web technology. In this project, I will replace `tornado` dependencies of `plumpy` and `aiida-core` by `asyncio` to  enable full support of `asyncio` in `aiida-core`. If the goals and deliverables are reached before the end of the project, I will also migrate `circus`, which is also used by many other open-source projects besides AiiDA, from `tornado` to `asyncio`.

## Technical Details
Coroutines, and asynchronous programming in general are used in many python web technologies, such as jupyter notebooks, volia, bokeh etc. As web technologies evolve, so do libraries for asynchronous programming, such as tornado or the asyncio module of the python standard library(available since python 3.4).
`plumpy`[^plumpy] is a python workflow library that supports writing Processes with a well defined set of inputs and outputs that can be strung together.
`circus`[^circus] is a Mozilla Foundation python library that runs and watches processes and sockets. It can be used as a library or through the command line.
`aiida-core`[^aiida-core] uses `circus` to daemonize its workflow manager process and uses `plumpy` as its workflow backend. However, `plumpy` and `circus` (both dependencies of `aiida-core`) have not kept up with recent developments, forcing AiiDA to run with outdated versions of `tornado`, and making it incompatible with the latest python web technology. The growing use of AiiDA in jupyter notebooks and web applications on platforms like the AiiDA lab and the Materials Cloud make it increasingly important to resolve this issue.
Meanwhile, `circus` is used by many other open-source projects besides AiiDA, which are all to benefit from this development.

[^plumpy]: repository `plumpy`, https://github.com/aiidateam/plumpy
[^circus]: repository `cicus`, https://github.com/circus-tent/circus
[^aiida-core]: repository `aiida-core`, https://github.com/aiidateam/aiida-core

### Code bases involved

#### `plumpy`
Start from `plumpy/processes.py` and `tests/test_processes.py`. Re-designing the `Future` class used of `plumpy` library. Unlike `Future` object in `tornado`, the `Future` instance in `asyncio` (or maybe will using `concurrent.futures.Future`) does not have attribute `_done`. Therefore the `SavableFuture` class should auto persist with `_state` rather than `_done`. Using `ContextVar` to store and access context-local context variable `_process_stack`, instead of using `_thread_local`. The file `plumpy/test_utils` which stores the process demos used in unittest would be better removed under the test module from plumpy module.

#### `aiida-core`
Two components of `aiida-core` are involved with asynchronous programming. First one is `aiida/manager`. It creates a runner when daemon is running and sends a task which is the instance of class ProcessLauncher. The aiida engine is the other component that needs asynchronous programming. In fact the runner used in manager mentioned above is actually created in `aiida/engine/runner.py` as the instance of class Runner. An event loop is launched here and processes are subscribed to the runner and processed asynchronously. There is also a `TransportQueue` class that yields a future result. This class is used in `aiida/engine/processes/calcjobs` and it allows clients to register their interest in a transport object which will be provided at some point in the future.

#### `circus`
Codes in `aiida-core` calling functions in circus should not be affected, therefore necessary to keep the `circus` API unchanged. Codes in `aiida-core` that call circus libraries serve as a proper start point to get a deep insight of the circus library. In file `aiida/engine/daemon/client.py`, circus client instance is created (in function `DaemonClient.client`) to control the behaviour of daemonized processes. And in file `aiida/cmdline/commands/cmd_daemon.py` users start daemons by calling `start_circus` function which sets up the arbiter config and actually launches the circus daemon.

Circus client is tested in file `circus/tests/test_client.py`. Circus arbiter is tested in file `circus/tests/test_arbiter.py`. The main TestCase classes are inherited from class `TestCircus` in file `circus/tests/support.py`. I will regard these three test files as entry points to start refactoring.
I will replace`tornado` with asyncio progressively until all `tornado` dependencies are totally removed.

### Inplementation details

Developing environment:

- Depend on kiwipy develop branch at the beginning but change to stable release branch to guarantee the asyncio version plumpy also work with tornado version kiwipy.
- Pin on python 3.5 features rather than py3.7 which include lots of new asyncio features to make sure the refactoring works for all py3.
- Test with `pytest-asyncio`.

Both in `aiida-core` and `plumpy`, replace `tornado` with `asyncio` by doing:

- Replace `yield` keyword with `await`
- Replace `@gen.coroutine` decorator with `async def`
- Replace `raise gen.Return` with `return`
- Unittests involved with asynchronous code is constructed with `@pytest.mark.asyncio`
- Convert `Process.step()`  call and similar to use `loop.run_once()`
- Re-implement `Future` class in `plumpy`.
- Using `contextvar` to record the stack of the current process.

After the code being modified, run integration test in `aiida-core` to make sure all code changes are  working flawlessly with `aiida-core>=1.1`. Meanwhile the test coverage in `aiida-core` should not be reduced.

`tornado==5.0` supports native coroutines and wrapper `asyncio` event loop in python3[^tornado-release-note]. And the latest `tornado` has adopted the coroutine implementation and has been compatible with `asyncio`. There is not much difference in writing `asyncio` code or the `tornado` asynchronous code, it is easy to change to the support to `asynio` when updating `tornado` to high version refactoring is done. The follow-up is up to the decision of the circus community.

According to issue[^circus-issue] discussed, `@k4nar` suggested making a PR to remove the support for Python versions before 3.5. This is actually equivalent to updating the tornado version to >6.0 which no longer supports Python 2.7 and 3.4 but to the minimum supported Python version is 3.5.2. Then continue to work on moving to asyncio, so we can always have the solution to fallback to tornado, if completely asyncio replacement proved to be too hard.
My plan is pinning to tornado version >5.0 and <6.0 both support python 2.7.9+ and 3.5+ which can still be used for most of users, even when the upgrading refactoring attempt fails.

The fundamental documentation for developers are the throughout docstring for every class and important functions.
In code phase, `missing-docstring` check should be turn off in `.pylintrc`. Then turn it on in documentation stage and add the necessary docstring.

[^tornado-release-note]: https://www.tornadoweb.org/en/stable/releases/v5.0.0.html
[^circus-issue]: free wish: tornado upgrade or switch to asyncio?, https://github.com/circus-tent/circus/issues/1124#issuecomment-600057407

## Schedule of Deliverables
This project will:

- Replace `tornado` dependencies by `asyncio` in `plumpy` and `aiida-core`
- Replace `tornado` dependencies of `circus` by `asyncio` (or at least by `tornado>=5`).
- Write further developer documentation (mostly the API documentation provided in docstring of useful class and methods) for plumpy and kiwipy.
- Thoroughly test to make sure asyncio version of `plumpy` and `circus` is working flawlessly with `aiida-core>=1.1`
- Deliverables include one PR for plumpy and aiida-core each, and two PRs for circus one for removing py2 compatibles and the other (stretch goal) for replacing tornado with asyncio.
- Write a wiki page in `aiida_core` repository about how `aiida_core` involved with asynchronous programming. This will helpful for developers who will look into the relative part of the code base.

Mentors will help manage interactions with the `circus` maintainers.

### Community Bonding period

#### Early May-1 June (+12d):
- Familiarize myself with plumpy’s functionality.
- Familiarize myself with the circus's functionality.
- Knowing which modules depend on the asynchronous programming.

### Phase 1

#### 1-12 June (12d): plumpy migration

- Replace `tornado` dependency with `asyncio` for all unittest.
- Replace `tornado` dependency with `asyncio` for code.
- *EDIT*: The future return from kiwi is a thread future, which can no longer yield to get the result. A elegant way
is needed to unwrap the result from kiwi future.

#### 12-16 June (4d): plumpy tests & documentation

- Integration test to make sure the new version of plumpy works  well with `aiida-core==1.2` and upwards.
- Fix bugs appear in this test stage.
- *EDIT*: Need to make sure that the changes of plumpy will minimize the code change in aiida_core, which means
the API of plumpy should remain unchanged.
- Document existing code of `plumpy` not only for users but for developers.
- Turn on the missing-docstring option in pylint configuration file, and add the comprehensive docstring for useful class and methods.
- Summarize the outcomes of this phase and prepare for the next coding phase.

(Remember to submit Phase 1 evaluations, DDL June 29)

### Phase 2

#### 16 June - 28 June (12d): aiida-core migration & tests

- *EDIT*: First, since the kiwipy is updated to asyncio, should change the aiida_core to working with the new kiwi.
- Replace tornado dependencies in aiida-core with asyncio
- Start from `aiida/manager` and `aiida/engine` replaceing coroutines with `asyncio` and registing task into `asyncio` event loop.
- Make sure the change not break other part of `aiida-core`
- Using the event loop provided by asyncio, and registering  the process in this event loop.
- Steps in a process are synchronous and different processes are running asynchronously. Therefore, make sure not to break this behaviour.
- Making a comprehensive test.
- *EDIT*: Should test aiida-core changes with both old and new version of plumpy.


#### 28 June - 22 July (24d): migrate circus arbiter & client (stretch goal)
- Remove all py2 compatibility supported by six library.
- Remove backwards-compatible to Python version before 3.5.2. Setting the Tornado version to 6.0.4.
- Fixing and coding to pass all failed unittest after upgrading the `tornado`.
- Start from `circus/tests/arbiter.py` and `circus/tests/client.py` replace `tornado` event loop with `asyncio` event loop.
- Replace original test backend (nosetest) with pytest-asyncio for asynchronous unittest.

#### 22 July - 1 August (10d):  circus migration continued (stretch goal)
If the arbiter and client are successfully refactoring, the main body this phase is almost finished. Then refactor the remaining parts of the circus to working with `asyncio`.

#### 1-14 August (14d):
Further refine tests for `circus` and `aiida-core`. Make sure `aiida-core` can work flawlessly with the new `plumpy` and new `circus` and test coverage does not go down.

#### 14 August - 21 August (7d):
- Document existing code of `plumpy` and `kiwipy` not only for users but for developers.
- Summary the outcomes of the phase.

(Remember to submit Phase 2 evaluations, DDL July 31)

### Final Week

#### 21-31 August (10d):
A buffer of 10 days has been kept for any unpredictable delay.

## Development Experience
I am already familiar with the code of `aiida-core`, `plumpy` and `kiwipy`, and have already contributed pull requests that have been accepted[^pr-aiida][^pr-kiwipy][^pr-plumpy].
I have developed AiiDA plugins `aiida-ce`[^aiida-ce] and `aiida-deepmd`[^aiida-dp] for training the potential functions and I am currently leading the effort to translate the AiiDA documentation to Chinese[^aiida-zh].

I have experience in programming in Golang, and end with homework like project `stateflow`[^stateflow] which requires the use of concurrent programming techniques. I am the main developer of `sagar`[^sagar] which is a python library with some utilities to generate and inspect grid site structures in material science, and as the administrator of the compute cluster in our laboratory, I have ripe experience in building and maintaining the HPC.

[^aiida-zh]: https://aiida.readthedocs.io/projects/aiida-core/zh_CN/latest/
[^pr-aiida]: https://github.com/aiidateam/aiida-core/commits?author=unkcpz
[^pr-kiwipy]: https://github.com/aiidateam/kiwipy/commits?author=unkcpz
[^pr-plumpy]: https://github.com/aiidateam/plumpy/commits?author=unkcpz
[^aiida-ce]: aiida-ce, https://github.com/unkcpz/aiida-ce
[^aiida-dp]: aiida-deepmd, https://github.com/unkcpz/aiida-deepmd
[^stateflow]: stateflow, https://github.com/unkcpz/stateflow
[^sagar]: sagar, https://github.com/scut-ccmp/sagar/graphs/contributors

## Why this project?
I use AiiDA to manage my high-throughput calculations for a long time, and find it is really accelerating my academic research a lot (I now have a paper under review that uses AiiDA for its high-throughput calculations [^arxiv]). AiiDA is well maintained, actively developed, and open to the contributions. This makes it stand out among similar tools in related fields.

I notice from the community that AiiDA suffers a bit from the outdated circus which depends on `tornado<5` at the moment, since much of the python ecosystem used by AiiDA starts to require `tornado>=5` or `asyncio`. Thus, this project will benefit the AiiDA to get rid of dependency problems which occur frequently when installing other python tools with `aiida-core`.

As an active user I want to contribute my effort to the project so I can not only learn advanced and standard technologies in python programming but know how to collaborate with others on open source projects.

[^arxiv]: Academic paper, https://arxiv.org/abs/2003.01481

## Appendix
According to the suggestion of [GSoC](https://developers.google.com/open-source/gsoc/help/work-product) and [NumFOCUS](https://github.com/numfocus/gsoc/blob/master/CONTRIBUTING-students.md) I will post recaps and working logs on my personal [blog](http://morty.tech) [^blog] once a week.

I will frequently commit code changes and rebase commits clearly. In order to make an easy evaluation result for GSoC, I will make one PR for plumpy and aiida-core each. Separate PR to circus into two stages. For circus, I will make two PRs, one for removing compatibility `py2` and the other for replacing `tornado` with `asyncio`.

Coding style follows: [AiiDA Coding-style](https://github.com/aiidateam/aiida-core/wiki/Coding-style)[^code-style]
Turn on the pre-commit, checking and fixing the style of code.

[^blog]: Personal blog, http://morty.tech
[^code-style]: AiiDA coding-style, https://github.com/aiidateam/aiida-core/wiki/Coding-style
