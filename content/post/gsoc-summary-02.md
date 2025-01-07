+++
title="GSoC week summary III"
date=2020-07-13
Tags=["GSoC"]
Category=["Note"]
+++

## The overview of progress

The following milestones are planed to be reached before submitting the evaluation
of phase-2 (31th-July) of project:

- [X] replace `tornado` with `asyncio` in `plumpy`
- [X] replace `tornado` with `asyncio` in `aiida_core`
- [X] *EDIT:* Make a through test in new environment (`aiida_core(asyncio)`+`plumpy(asyncio)`+`kiwipy(asyncio)`+`circus(new)`) to guarantee that the changes do not break any functionalities of `aiida_core`.
- [ ] Document code of `plumpy` not only for users but for developers.
- [X] update `tornado` to `>4.5.2` which use asyncio event loop in `circus`
  - [X] Remove all py2 compatibility supported by six library of `circus`. (Already done by their maintainer)
  - [X] Setting the `tornado>5.0.2`, all unit tests are passed after refactoring circus.
  - [X] Running all examples of circus, to make sure all the required modifications have been covered.
- [X] Further refine tests for circus and aiida-core so aiida-core can work flawlessly with the new plumpy and new circus and test coverage does not go down.
- [X] *ADD:* update localization documentation to revap version. (plan to translate at least 100 strings everyday so can finish before September)

## Details of some achievements and difficulties

### documentation

Thanks to the engine paper, I have got a more clear bird's-eye view of the plumpy,
it will be a nice signal for me to start writing the documentation of plumpy.

Another progress is that I have reorganized the [aiida-core documentation localization](https://github.com/unkcpz/aiida-l10n-zh_CN),
due to the new docs structure, it is more easier to manage the localization repository.
You can check the translated documentation [here](https://aiida.readthedocs.io/projects/aiida-core/zh_CN/latest/)
(or directly access from the language button from official documentation) everything is perfect!

### `nest_asyncio` not merged

It seems the maintainer of `nest_asyncio` still not address the [nest_asyncio #28](https://github.com/erdewit/nest_asyncio/pull/28)

### return value of `kill_process` method

State: Resolved

Since `LoopCommunicator` is the communicator for all the rpc and
broadcast communication, process controlling operation via command line
such as `verdi process pause` which calling the
`controller.pause_process` will wrap the result in one more
`kiwipy.Future`. Therefore, the future get from controller need to be
unwrapped by `plumpy.unwrap_kiwi_future` first.

### unwrap future returned from process controller

State: Resolved

The future result return from `controller.kill_process` is the result of
rpc calling of `aiida.engine.processes.process::Process::kill` method,
which will return a future represent a boolean value to
indicate that the process and its child process is successfully killed.
So the list of futures is chained to a future and returned.

### When exception raised and captured by `kiwipy` rmq can not be disconnected

State: Resolved by [kiwipy #71](https://github.com/aiidateam/kiwipy/pull/71)

When exception raised and going to be handled by
`kiwipy.capture_exception` (for example in `plumpy.futures.create_task)` in
event loop other than the communicator one, calling `aio_future.set_exception(exc)` directly is
non-threadsafe.

This will lead to a bad consequence that connection is not be able to be closed:

```
raise RuntimeError('CRASH')                                                                                         
RuntimeError: CRASH                                                                                                     

During handling of the above exception, another exception occurred:                                                     

Traceback (most recent call last):                                                                                      
File "/data/CONDA_ENV/gsoc-aiida/lib/python3.8/concurrent/futures/_base.py", line 328, in _invoke_callbacks           
callback(self)                                                                                                      
File "/home/unkcpz/pyProject/kiwipy/kiwipy/rmq/threadcomms.py", line 256, in done                                     
aio_future.set_exception(exc)                           
File "/data/CONDA_ENV/gsoc-aiida/lib/python3.8/asyncio/futures.py", line 254, in set_exception                        
self.__schedule_callbacks()                                                                                         
File "/data/CONDA_ENV/gsoc-aiida/lib/python3.8/asyncio/futures.py", line 149, in __schedule_callbacks                 
self._loop.call_soon(callback, self, context=ctx)                                                                   
File "/data/CONDA_ENV/gsoc-aiida/lib/python3.8/asyncio/base_events.py", line 721, in call_soon                        
self._check_thread()                                                                                                
File "/data/CONDA_ENV/gsoc-aiida/lib/python3.8/asyncio/base_events.py", line 758, in _check_thread                    
raise RuntimeError(                                                                                                 
RuntimeError: Non-thread-safe operation invoked on an event loop other than the current one
```

## A premature user requirement of AiiDA
