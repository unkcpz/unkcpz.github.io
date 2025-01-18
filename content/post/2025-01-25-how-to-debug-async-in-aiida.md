---
title: How to debug async code in aiida-core engine
date: '2025-01-25'
categories:
    - post
tags:
    - aiida 
    - aysnc
---

<!-- debug memory leak, another post? -->

- Introduce why we need asyncio programming in aiida.
- Why async debugging is hard for aiida.
- Introduce the normal ways of debugging async python code.
- Which way we can use and how to use it to debug with one specific case where the event loop stuck at running some coroutine.
- How we can keep on improve for aysnc programming in aiida.

## Async programming in aiida.

## Why debug it is hard?

The async programming in general is not easy to debug, because things happened in the same place.
Debugging is mostly rely on the logging, because setting breakpoint is not stop running things it yield the control back so other thing may still running and make the behavior at the break point not reliable. 
The runtime is running in a thread that can not be accessed. 
Python didn't provide ways to put the breakpoint to stop as synchronous code. 
In aiida-core the things get a bit non-standard because we use nest-asyncio to make the event loop reentry.
In aiida-core the even loop can run on different inteperet through a dedicate runner.

## Debug async code in aiida-core
- What is the regular way to use for asyncio programming debugging?
- What fit for aiida?
- `loop.set_debug(True)`
- tracing? more log info?
- Where is my debug message goes?
- Find bug from plumpy that stuck the aiida RPC communication.

## `aiomonitor` is my good friennd
- Find where it stuck, a typical case study.

## What we can improve?
