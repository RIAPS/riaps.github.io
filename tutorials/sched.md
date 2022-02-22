## RIAPS Scheduling

RIAPS application actors and components are scheduled using the scheduling features available
in Linux. An actor is a Linux process, and each component runs in its own thread. Python components
run in a Python thread. Each actor has a main thread (that is always the main thread of the Python
interpreter), as well as ZeroMQ communication threads (that are pthreads).
When a Python component is activated it gets its own thread.

Scheduling of actors (processes) and component (threads) is under the control of the Linux scheduler.
Each component inherits the scheduling properties of the actor the component is running in, and all
component threads have the same scheduling property.

On the component level, the designer can select how to schedule the processing of incoming messages,
but all this processing happens in the same thread (that has the same scheduling properties as the
parent actor, and all other component threads in the same actor).

### Default Actor Scheduling

The default actor scheduler is the default Linux scheduler - 'Completely Fault Scheduler' (CFS).
In this case, an actor is subject to the same scheduling policy as any conventional Linux program,
including priority adjustments, etc. The CFS "aims to maximize overall CPU utilization while also
maximizing interactive performance" [https://en.wikipedia.org/wiki/Completely_Fair_Scheduler].

### Real-time Actor Scheduling

If the actor specification is prefixed with the keyword 'real-time' then the actor will follow
Linux real-time scheduling policies. In this case, the specification may include a clause that
controls the scheduling properties for the entire actor (and all components in it).

#### Priority-based Scheduling

Here the designer should specify the desired priority level (from the range of 1..99 of Linux) at
which the actor runs. The example below shows how to do it.

```
real-time actor Estimator {
  local SensorReady, SensorQuery, SensorValue ; // Local message types
  scheduler priority 90;                        // Runs @ real-time (OS) priority 90
  { // Sensor component
    sensor : Sensor;
    // Local estimator, publishes global message 'Estimate'
    filter : LocalEstimator(iArg=789,fArg=0.12,sArg="text",bArg=false);
  }
}
```

In this case the 'SCHED_FIFO' policy of Linux will be used for the actor, with the given priority.
Under this policy a higher priority actor always preempts a lower priority actor (and an actor scheduled
under CFS).

#### Round-Robin Scheduling

Here the designer just selects the scheduling discipline and Linux's 'SCHED_RR' policy will apply.
The example is shown below.

```
real-time actor Aggregator (posArg,optArg="optString") {
  scheduler rr;            // Runs under round-robin (OS) scheduler
  { // Global estimator, subscribes to 'Estimate' messages
    aggr : GlobalEstimator(iArg=posArg,sArg=optArg,bArg=true);
  }
}
```

Under this policy the actor will be scheduled in a round-robin manner, with a certain time slice (as set by the OS).

### Default Component Message Scheduling

The default component message scheduler is based on the 'poll' operation available in Linux (and ZeroMQ).
After activation, the component is waiting for an incoming message on all its input ports, using 'poll'.
Component input ports include the following port types: sub, srv, rep, ans, timer, and ports that are created
for the dynamic groups the component participates in. When one or more ports receive a message, the component
randomly picks one port with a message and executes the handler associated with it. After execution, the component
keeps picking from the set of ports with messages and calls their handlers. This process runs until all ports are
handled in the batch returned from the poll, and then the component blocks on another poll operation that waits
for any of the ports. The choice of ports is non-deterministic and cannot be controlled.

#### Priority-based Component Message Scheduling

In this scheme incoming messages are processed in a priority order determined by the lexical order of the ports
in the specification of the component. Messages arriving at a port that is the first in the list of (input) ports
will be processed before messages for ports later in the list. Consider the example below.

```
component Sensor
  scheduler priority;              // Schedule message processing per priority (order in the port list)
  {
    timer clock 1000;              // Periodic timer trigger to trigger sensor every 1 sec
    pub ready : SensorReady ;      // Publish port for SensorReady messages
    rep request : (SensorQuery,SensorValue); // Reply port to query the sensor and retrieve its value
  }
```

In the above example 'clock' messages will be processed first, and then the 'request' messages. This rule applies
only if there are multiple messages waiting on multiple ports -- then the processing is done in the lexical order
of the ports. If there is only one message at any port, that will be processed as soon as the component finished
processing the previous message.

#### Round-robin Component Message Scheduling

In this scheme, incoming messages processed in a round-robin fashion, where the order of the 'round' is determined
by the lexical order. See example below.

```
// Local estimator component
component LocalEstimator (iArg,fArg,sArg,bArg)
  scheduler rr;                             // Schedule messages processing per round-robin algorithm
  {
    timer tick 1000;                        // One more ticker
    sub ready : SensorReady ;               // Subscriber port to trigger component with SensorReady messages
    req query : (SensorQuery,SensorValue) ; // Request port to query the sensor and retrieve its value
    pub estimate : Estimate ;               // Publish port to publish estimated value messages
  }
```
In the above example 'tick' messages are processed, then 'ready', then 'query, and then 'tick' again, etc.
This rules applies only of there multiple messages waiting on multiple ports -- then the processing is done
in the round robin order. For example, if we have two 'ticks', three 'ready'-s, and one 'query', the order
will be: tick, ready, query, tick, ready, ready. If there is only one message at any port, that will be processed
as soon as the component finished processing the previous message.
