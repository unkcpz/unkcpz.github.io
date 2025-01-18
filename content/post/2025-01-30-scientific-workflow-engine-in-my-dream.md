---
title: The perfect scientific workflow engine in my dream
date: '2025-01-30'
categories:
    - post
tags:
    - aiida 
    - workflow
draft: true
---

## The almost python-like API 

I have some example code from egobox and want to run some heavy evaluations for `y`
```python
def opt_origin():
    xlimits = egx.to_specs([[0.0, 25.0]])
    egor = egx.Egor(xlimits, seed=42) 

    # initial doe
    x_doe = egx.lhs(xlimits, 3, seed=42)
    y_doe = []
    for x in x_doe:
        y = xsinx(x)
        y_doe.append(y)

    for _ in range(10): # run for 10 iterations
        x = egor.suggest(x_doe, y_doe)  # ask for best location
        x_doe = np.concatenate((x_doe, x))

        y = xsinx(x)

        y_doe = np.concatenate((y_doe, y)) 

    res = egor.get_result(x_doe, y_doe)
    print(f"Optimization f={res.y_opt} at {res.x_opt}")
```

The code can be translate to following template of a workflow
```python
@workflow # where all the magic conversions happened
def opt_workflow():
    xlimits = egx.to_specs([[0.0, 25.0]])
    egor = egx.Egor(xlimits, seed=42) 

    # initial doe
    x_doe = egx.lhs(xlimits, 3, seed=42)
    y_doe = []
    for x in x_doe:
        # Here is the change, where the evaluation can happened in remote resource
        y = submit(xsinx, x=x)

        y_doe.append(y)

    for _ in range(10): # run for 10 iterations
        x = egor.suggest(x_doe, y_doe)  # ask for best location
        x_doe = np.concatenate((x_doe, x))

        # Here is the change, where the evaluation can happened in remote resource
        y = submit(xsinx, x=x)

        y_doe = np.concatenate((y_doe, y)) 

    res = egor.get_result(x_doe, y_doe)
    print(f"Optimization f={res.y_opt} at {res.x_opt}")

# the function will first "compiled" to a Process which is able to be executed by engine
node = engine.submit(opt_workflow)
```

It requires to store the local state of a for loop.
