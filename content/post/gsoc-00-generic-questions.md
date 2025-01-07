+++
title="GSoC Logs: Generic questions"
date=2020-05-11
Tags=["GSoC", "AiiDA", "QNA"]
Category=["Note"]
lastmod=2020-05-11
+++

## Description

Here the post record the questions and difficulties encountered during the project which
are not able to categorized to the sub-project.

## May-11

#### Specify event loop in function is removed

In high version tornado (>4.0) and new version asyncio, the argument `loop`
is all removed. Then all functions with this argument specified should all
be updated. Check the question [here](https://stackoverflow.com/questions/60312374/what-are-all-these-deprecated-loop-parameters-in-asyncio). This to-do change appears in all the sub-project when removing
the older version of tornado.

Well, how to update? By just remove the argument in function?
