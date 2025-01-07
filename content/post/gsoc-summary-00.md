+++
title="GSoC week summary I"
date=2020-06-12
Tags=["GSoC"]
Category=["Note"]
+++

## The overview of progress

The following milestones are planed to be reached before submitting the evaluation
of phase-2 (31th-July) of project:

- [X] replace `tornado` with `asyncio` in `plumpy`
- [X] replace `tornado` with `asyncio` in `aiida_core`
- [X] Make a through test in new environment (`aiida_core(asyncio)`+`plumpy(asyncio)`+`kiwipy(tornado)`+`circus(old)`) to guarantee that the changes do not break any functionalities of `aiida_core`.
- [ ] Document code of `plumpy` not only for users but for developers.
- [ ] update `tornado` to `>4.5.2` which use asyncio event loop in `circus`
  - [X] Remove all py2 compatibility supported by six library of `circus`. (Already done by their maintainer)
  - [X] Setting the `tornado>5.0.2`, all unit tests are passed after refactoring circus.
  - [ ] Running all examples of circus, to make sure all the required modifications have been covered.
- [ ] Further refine tests for circus and aiida-core so aiida-core can work flawlessly with the new plumpy and new circus and test coverage does not go down.

## Details of each checkbox: Achievements and Difficulties

### Replace `tornado` with `asyncio` in `plumpy`

It is already done and under review in [PR#150](https://github.com/aiidateam/plumpy/pull/150) .
Two things prevent this pr from being merged.
Firstly, The processes are able to nested and launch in other process rely on the reentrant of event loop which is not supported by asyncio event loop.
I have to use `nest_asyncio` patch to make this possible, but there is a event loop setting bug in that library, need to wait until it is addressed and release to pypi.
Secondly, as mentioned by Sebastiaain there are some crucial improvements which not yet finished need to be merge to the develop branch before this.

Therefore, there is nothing for me to do right now, I mark it as finished.

###  replace `tornado` with `asyncio` in `aiida_core` (and tests in beneath checkbox)

I am going to submit this [PR](https://github.com/unkcpz/aiida_core/pull/1) after
[#4154](https://github.com/aiidateam/aiida-core/pull/4154)
and [#4156](https://github.com/aiidateam/aiida-core/pull/4154) are merged,
since there will be some hard to addressed conflicts after Sebastiaain add the broadcast
subscriber to deal with the awaitables in workflows.
And since the thread communicator of runner is going to be replaced with the `LoopCommunicator`,
I think it will cause more subtle conflicts with my refactoring now, so I plan to submit
this PR later after more checks locally to relieve your burden.

Meanwhile, another problem prevent me from surely mark this part as finished.
That is the refactoring one  (`aiida_core(asyncio)`+`plumpy(asyncio)`) did not work flawless with `kiwipy-0.6.0(asyncio)`.
I have elaborated on this question in the slack discussion. When working with communicators from `kiwipy-0.6.0` the
channels can not be successfully closed and connections hangs on disconnection when the rmq communication finished.

Even that I marked this as finished, there are still something I need to take care of and the remaining problems need your help to solve together.

And what follows is the through test of `aiida_core` in new environment.
Since the test coverage of `aiida_core` is not high enough to be reassuring, I make the a blackbox test of `aiida_core`
with running examples of tutorials and workflows I used in my research project before.
It is proved that this kind of test is necessary. I spot a small process status wrongly set bug which caused by my careless when refactoring the coroutine in `CalcJob`.
Beside from that, no more issues were found temporarily. So I also marked this checkbox as finished.
I looking forward that `kiwipy-0.6.0` can correctly work in this environment, so that all tornado dependencies are coming from circus and I can make a test with `circus>5.0.2`. At that time, the final goal is regard as reached.

### Document code of `plumpy` not only for users but for developers

This part is prepare to start right after the refactoring of plumpy is done, and in facto I have
made some attempt in doing this https://github.com/unkcpz/plumpy/tree/gsoc-doc
However, I then plan to postpone this after scipy meeting and focus on completing the deprecate of tornado before that time point.
Previously, I arrange the documentation work to be done within a week, but I realized I slightly overestimate my
ability in doing this (mostly from my non-native English writing).

I marked this item as unfinished, and plan to start this in 20th-July. But not quite, I may start this if
there is nothing for me to do before the middle of July.  

### update `tornado` to `>4.5.2` which use asyncio event loop in `circus`

This part of project was thought to be the most difficult part of the gsoc project, however, it is not that difficult after some attempts.
It was plan to start with dropping all `py2` support.
This have been solved and merged by their maintainer https://github.com/circus-tent/circus/pull/1126
So, I directly start this from dropping dependency of `tornado<5` which use tornado's own event loop.
The `arbiter` and `client` are two core component of `circus`, I start refactoring from their unit test case and
replace the deprecated old tornado syntax step by step end with my goal reach without too much barriers.

As the test coverage of `circus` is less than 70% I am not confident my changes cover all tornado dependent code.
My subsequent plan is running the examples provided in the package as some sort of blackbox test in order to guarantee
all relevant code is covered.

I personally not propose to move to asyncio in hurry. Only if the refactoring circus works flawless with new tornado the subsequent migration to asyncio would all comes naturally.

## Next todo

I do not fully clear about the priorities of what to do next. My plan is to make the
through blackbox test of circus with its examples if nothing else is a higher priority. 
