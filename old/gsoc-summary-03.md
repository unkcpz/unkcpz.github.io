+++
title="GSoC weeks summary IV"
date=2020-08-07
Tags=["GSoC"]
Category=["Note"]
+++

## The state of each part of gsoc project

- [ ] `aiida_core` dopends on `circus`, `plumpy`.
  - [X] `plumpy` depends on `nest_asyncio`, `kiwipy`
    - [X] Changes of `nest_asyncio` is already merged and released in `v1.4.0`
    - [X] minor changes of `kiwipy` is merged and released in `v0.6.1`
    - [X] The github action bug which prevent py3.7 from installing correct version of `pyyaml`, RESOLVED.
    - [ ] The documentation for developers is planed to be woven on the new asyncio version plumpy. The docstring for all public function is now added in [this pr](https://github.com/unkcpz/plumpy/pull/3). I kept it in fork repository since it based on asyncio branch which not merged yet.

  - [ ] `circus` is used as the daemonizer in `aiida_core`.
    - [X] The update-tornado changes in `circus` library is already merged into `master` branch.
    - [ ] However, an [critical bug](https://github.com/circus-tent/circus/issues/1131) was found when test it. It is fixed in [#pr1132](https://github.com/circus-tent/circus/pull/1132) and plan to be merged and released as version `0.17` announced in the [milestone](https://github.com/circus-tent/circus/milestone/13)

  - [ ] `aiida_core` itself
    - [X] The refactoring is finished and ready to be reviewed.
    - [ ] Since the new `circus` and `plumpy` is not released yet, github action is explicitly doing test depend on package in fork repository. To simplify the future work, I only tag the CI workflow of `test` workflow, and left docker test workflow and install-test workflow unchanged, so they will failed in CI test.
    - [ ] Still not very sure I did the right modification of `runner.loop` in `aiida_core`. I will write a detail post about it and discuss with Martin.

Therefore, `plumpy` is clearly ready to be reviewed and merged. Meanwhile, maintainers from `circus` is progress in checking the code and I guess the new version will be released soon.

The upcoming deadlines is, from Aug-24 I can submit the code. In order to make you (mentors @Leopold and @Sebastiaan) easy to collect and submit the achievement of the project, I list all the links and content following. Besides, since I start helping Guoyu Yang and Qiang Sun to use AiiDA in their research filed, I plan to take the mission of aiidalab deployment etc. as bonus.

#### Codes, links, and contents of all achievement for easy evaluation

- All posts about the project is in my personal blog tag with gsoc: http://morty.tech/tags/gsoc/
- Drop the old version tornado and uses asyncio event loop tornado in `circus`. [1] https://github.com/circus-tent/circus/pull/1129 and [2] https://github.com/circus-tent/circus/pull/1132
- Replace `tornado` with `asyncio` in plumpy. https://github.com/aiidateam/plumpy/pull/150 also archive all discussion in community.
- [ ] plumpy documentation for developers.
- Replace `tornado` with `asyncio` in `aiida_core`. https://github.com/aiidateam/aiida-core/pull/4218
- Some sporadic contributions in and out of the AiiDA community during GSoC.
  - Mentioned minor fix for `nest_asyncio` https://github.com/erdewit/nest_asyncio/pull/28 .
  - PRs: https://github.com/aiidateam/aiida-core/pull/4271 , https://github.com/aiidateam/aiida-core/pull/4081
  - Code review attempts: https://github.com/aiidateam/aiida-core/pull/4117 , https://github.com/aiidateam/aiida-core/pull/3977
  - [ ] aiidalab: https://github.com/aiidalab/aiidalab-widgets-base/pull/110
