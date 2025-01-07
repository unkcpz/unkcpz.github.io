+++
title="Final report GSoC 2020: full support of asyncio in AiiDA"
Tags=["GSoC"]
Category=["Note"]
date=2020-08-20
lastmod=2020-08-28
+++

## Codes and links of all achievement for easy evaluation

- [X] Drop the old version tornado and uses asyncio event loop tornado in `circus`. (status: work finished)
  - [circus#1129](https://github.com/circus-tent/circus/pull/1129) circus with new tornado>5.0.2 (Status: PR merged, unreleased)
  - [circus#1132](https://github.com/circus-tent/circus/pull/1132) Fix the `quit --waiting` command hang up issue (Status: PR open)


- [X] Replace `tornado` with `asyncio` in plumpy. (Status: Work finishied, released in package version [v0.16.0](https://github.com/aiidateam/plumpy/releases/tag/v0.16.0))
  - [plumpy#160](https://github.com/aiidateam/plumpy/pull/160) (for discussions, see [plumpy#150]((https://github.com/aiidateam/plumpy/pull/150))
  - plumpy documentation revamp. (Status: PR draft) https://github.com/aiidateam/plumpy/pull/167


- [X] Replace `tornado` with `asyncio` in `aiida_core`. (status: work finished, PR open)
  - (under review)[aiida_core#4317](https://github.com/aiidateam/aiida-core/pull/4317) drop tornado<5 and implement new event loop policy with asyncio.

## Sporadic contributions in and out of the AiiDA community during GSoC.
  - Mentioned a minor fix for `nest_asyncio`. https://github.com/erdewit/nest_asyncio/pull/28
    - This PR fix the behavior when exit from the inner nested loop, set the running loop to the original one.


  - PR for aiida_core:
    - Add an option in `verdi computer` to switch off login shell. https://github.com/aiidateam/aiida-core/pull/4271
    - Enhance provenance graph visualization to make it possible to highlight the graph node by designate its class name. https://github.com/aiidateam/aiida-core/pull/4081


  - Code review for aiida_core:
    - Add support for process functions in `verdi plugin list`. https://github.com/aiidateam/aiida-core/pull/4117
    - Enable loading config.yml files from URL. https://github.com/aiidateam/aiida-core/pull/3977


  - aiidalab (Status: PR open):
    - Add feature: configure computer ssh transport by providing private key. https://github.com/aiidalab/aiidalab-widgets-base/pull/110

## Logs and week summary of the project
All posts about the project is in my personal blog tag with gsoc: https://morty.tech/tags/gsoc/
