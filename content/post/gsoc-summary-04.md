+++
title="GSoC weeks summary V (event loop policy)"
date=2020-08-13
Tags=["GSoC"]
Category=["Note"]
+++

## New event loop policy of aiida_core after migrating to asyncio

At the moment, the old tornado<5 event loop is totally replaced with asyncio event loop. I slightly change the Runner’s event loop policy in aiida_core. The new design is represented by following two user cases.

Scenario 1:

Without starting daemon, user directly run the following workfunction:

```python
from aiida.engine import calcfunction, workfunction
from aiida.orm import Int

@calcfunction
def add(x, y):
    return Int(x + y)

@calcfunction
def multiply(x, y):
    return Int(x * y)

@workfunction
def add_and_multiply(x, y, z):
    sum = add(x, y)
    product = multiply(sum, z)
    return product

result, node = run.get_node(add_and_multiply, Int(1), Int(2), Int(3))
print(result)
print(node)
```  

Decorating a regular funtion with `@workfunction` or `@calcfunction` make the function become the `process_function` which defined in `aiida/engine/processes/function.py`. When user trigger the process_function, it firstly calling `manager.get_runner(with_persistence=False)` to find the runner, if the runner not exist create the runner and use the current event loop (asyncio.get_event_loop() which get the current process-wide loop in default asyncio loop policy). Therefore if the nested process_function is triggered, it uses the runner of parent process_function. The runner providing the event loop for process to be running. The process is executed by `process.execute()`, this method is defined in `plumpy.processes.Process`. Since we have re-entrancy by using `nest_asyncio` all process running in the same event loop. This is the biggest difference from the previous tornaod<5 event loop policy where every `child` process create a new runner and correspoinding to new event loop.
In this case, the runner's loop not being stopped explicitly by calling `loop.stop()`, since we never explicitly start the loop and keep it running, rather the process is run by run_until_complete which stop running until all the task in the loop is finished.

Above logic can be migrated to 'run'(just to distingush from submit) the calcjob. The only different is the runner is get(if exist) or create by function `run` or `run_get_node` defined in `aiida/engine/launch.py`. The event loop is shared by all the process launched at that moment.

Scenario 2:

Submiting a `NestedWorkChain`.

The `NestedWorkChain` is printed following:

```python
class NestedWorkChain(WorkChain):
    """
    Nested workchain which creates a workflow where the nesting level is equal to its input.
    """

    @classmethod
    def define(cls, spec):
        super().define(spec)
        spec.input('inp', valid_type=Int)
        spec.outline(cls.do_submit, cls.finalize)
        spec.output('output', valid_type=Int, required=True)

    def do_submit(self):
        if self.should_submit():
            self.report('Submitting nested workchain.')
            return ToContext(workchain=append_(self.submit(NestedWorkChain, inp=self.inputs.inp - 1)))

    def should_submit(self):
        return int(self.inputs.inp) > 0

    def finalize(self):
        """Attach the outputs."""
        if self.should_submit():
            self.report('Getting sub-workchain output.')
            sub_workchain = self.ctx.workchain[0]
            self.out('output', Int(sub_workchain.outputs.output + 1).store())
        else:
            self.report('Bottom-level workchain reached.')
            self.out('output', Int(0).store())

```

It will nest workchains layers by layers depend on the input value. Here, we use `submit` function imported from `aiida.engine.launch` to submit the workchain, in submit we simply create a runner to let the persister and controller conrrespondent to each other, otherwise the saving checkpoint can not be coorectly read by the right controller.
After submit the workchain, user is waiting for the process to be run. However, at this moment, there is no runner actually running. In order to run the process, the daemon runner is created and running in the background. It's now start listening for the rmq message and when message(continue_process) recieved, the process is recreated and running in the daemon runner event loop. From this time, the nested workchain(nested one rather than the one we submit just now) will running, when run to `self.submit` this submit is the submit method of `Runner` it will schedule the process in the currently activate daemon runner's loop. And this is the primary cause why use `aiida.engine.submit` to launch the outmost workchain and use `self.submit` to launch the nested workchain.

### Important note for daemon

Every time while a daemon is start, it is started by `verdi devel run_daemon` through circus, meanwhile a runner is created concomitantly. The event loop of this runner is thread independent and every 'process'(the concept of circus, correspoing to a running command) has its own event loop.
