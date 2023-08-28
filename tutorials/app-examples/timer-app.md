# Timer programming
Two applications, `Periodic` and `Sporadic`, are located in the LearnTimer directory of [riaps-tutorials](https://github.com/RIAPS/riaps-tutorials). They demonstrate the below timer programming models.
## Periodic Timers
Periodic software timers enable programmers to call functions at regular intervals. RIAPS supports this by allowing components to declare a `timer` port, as in the below model file snippet: 
```
...
component Foo {
    timer clock 1000 msec;
    ...
}
...
```
This port, named `clock`, will send the `Foo` component a message every 1000 milliseconds. Because an interval is specified, this will be a **periodic timer**. When the `Foo` thread receives that message, it will look for a function named `on_clock` in the `Foo.py` component code.
```python
from riaps.run.comp import Component

class Foo(Component):
    def __init__(self):
        super(Foo, self).__init__()

    def on_clock(self):
        msg = self.clock.recv_pyobj() # Required to remove the timer message from the message queue
        self.logger.info(f"Hello world!")
```
Running this component will cause `Hello world!` to be logged once every second. `msg` in this scope is a timestamp of when the `clock` timer fired; its value is a `float` instance, containing the number of seconds since the [Unix epoch](https://en.wikipedia.org/wiki/Unix_time).

### Model file syntax:
```
timer <PORT NAME> <INTERVAL> <UNITS>;
```
UNITS can be one of `min`, `sec`, `msec`. 
### Timer Port API
[View on GitHub](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/run/timPort.py#L239)

## Sporadic Timers
Sporadic software timers allow code to declare that some action will happen once, after a specified delay. RIAPS supports this by allowing components to declare a `timer` port without a given interval, as in the below model file snippet:
```
...
component Bar {
    timer clock;
    ...
}
...
```
To use this timer port, give it a delay interval using it's `setDelay(<some interval, float>)` method, then launch the timer thread with its `launch()` method. The below implementation of `Bar` calls these functions on the timer port `clock` in the components `handleActivate` method.
```python
from riaps.run.comp import Component

class Bar(Component):
    def __init__(self):
        super(Bar, self).__init__()

    def handleActivate(self):
        self.clock.setDelay(3.0) # Delay for 3s
        self.clock.launch() # Start the timer
        self.logger.info("I will say Hello world! in 3 seconds...")

    def on_clock(self):
        msg = self.clock.recv_pyobj() # Required to remove the timer message from the message queue
        self.logger.info(f"Hello world!")
```
Running an application that uses this component will result in this sequence of events:  
1. The actor containing `Bar` initializes the component, then its ports.
2. The actor calls `handleActivate()` for each of its components, during which `Bar` sets its sporadic timer delay (`setDelay(3.0)`) and launches it (`launch()`).
3. The actor starts the component polling.
4. After 3 seconds "Hello World!" is logged by the component.

Sporadic timers support other methods for modifying the interval and cancelling an active timer. These are documented in the port's [API](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/run/timPort.py#L239).