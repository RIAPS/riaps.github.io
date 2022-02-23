## Application Model and Deployment Files Tutorial

In RIAPS, applications consists of components (written as Python modules), additional libraries the application needs, and descriptions of the application architecture and how it should be deployed on a network or RIAPS nodes.
Applications are described using a model file (*.riaps*) that define component port types (and, implicitly, connections among those ports), the messages passed between or within components, and how the components are utilized to create actors. The deployment model (*.depl*) defines the expected configuration of the application on a specific network setup.  

### Page Contents

* [Application Model File (*.riaps*)](#riaps-def)
  * [Component Definitions](#comp-def)
    * [Timer Ports](#timer-def)
    * [Publish/Subscribe Ports](#pub-sub-def)
    * [Request/Reply Ports](#req-rep-def)
    * [Client/Server Ports](#clt-srv-def)
    * [Query/Answer Ports](#qry-ans-def)
    * [Inside Ports](#inside-def)
  * [Actor Definitions](#actor-def)
    * [Resource Management Specifications](#rm-model-spec)
* [Application Deployment File (*.depl*)](#depl-def)
  * [Network specification](#rm-model-spec)
* [Component Configuration Using Parameters](#comp-conf-def)
* [Advanced Modeling Options](#adv-model-opts)
  * [Message Scheduling](sched.md)
  * [Distributed Coordination (Groups)](groups.md)

### <a name="riaps-def">Application Model File (*.riaps*)</a>

The core of the application model file defines the application elements:
- **Messages** passed between Components
- **Component** definitions - available ports and the messages used by each defined port (interconnections)
- **Device component** definitions - special component definitions of available physical hardware devices  
- **Actors** definitions - components grouped together to make an actor

The key words used to create the core application elements are:
- Application name <br>
  'app yourAppName { ... }'
- Component definition - lists all port connections and message passing , explained further below <br>
  'component yourComponentName { ... }'
- Device component definition - same as component definition, but defines a physical hardware or software interface available to actors on a RIAPS node <br>
  'device yourDeviceName { ... }'
- Messages - list messages available in the application <br>
  'message yourMessageName;'
- Actor definition - list all actors in the application <br>
  'actor yourActorName{actorInstanceName : yourComponentName;}'
- *Optional:*  Library name - Python or C++ library available to the components code development, either third party packages or developer created.  The name is a string value representing the folder in which the library code is located.  <br>
  'library libraryName;'

Below is a skeleton of an application file to show how these elements relate in the model (*.riaps*) file.

```
app TestApp {
    message MsgRequest;
    message MsgReply;
    message Msg1Data;
    message Measured;

    library thirdPartyLibraryName;

    component ComponentName1 {     
        ...
    }

    component ComponentName2 {
        ...
    }

    device ADevice {
        ...
    }

    actor ActorName1 {
       {  
          componentInstanceName : ComponentName1;
          deviceInstanceName : ADevice
       }
    }
    actor ActorName2 {
       {  
          componentInstanceName : ComponentName2;
       }
    }
}
```

#### <a name="comp-def">Component Definitions</a>

The component definitions will indicate the available ports in each components.  The messages are the connections between the component ports.  Messages can be within a component or between components. The types of available ports are defined in the [architecture discussion](https://riaps.github.io/arch.html).  The following shows the interaction patterns available:

- Timers (**timer**)
- Publish (**pub**) --> Subscribe (**sub**)
- Request (**req**) <--> Reply (**rep**)
- Client (**clt**) <--> Server (**srv**)  
- Query (**qry**) / Answer (**ans**)
- Inside Ports (**inside**) - used only in device components to isolate hardware communication

##### <a name="timer-def">Timer Ports</a>

Timers are defined in the application model file with **name** and **period** values. Any number of timers can be defined within a component. Timers run in the background and send a message in every period. The period value is an integer and is specified in milliseconds by default. Time units of **sec**, **min**, or **msec** can be added after the period value to use a different unit of measured.

```
timer yourTimerName 5000; // 5s timer
```

or

```
timer yourTimerName 5 sec;
```

Timers can also be sporadic to allow delay times to be defined and controlled in the component code.  Sporadic timers do not have a period value in the specification, as seen below.

```
timer yourTimerName;
```

To detect deadline violations caused by the component's code that handles timer events, a deadline can be specified by adding **within deadlinePeriodValue** to the end of the port definition.  The same time units as Timer ports used can be added here.  If not specified, the time units are assumed to be in milliseconds. The component framework has a **handleDeadline()** function that can be utilized to perform actions when the deadline is not met.  An example of usage of this keyword can be see in the [DEstDeadline](https://github.com/RIAPS/riaps-pycom/tree/master/tests/DEstDeadline) example project.

```
timer yourTimerName 1 sec within 1 msec;
```

##### <a name="pub-sub-def">Publish/Subscribe Ports</a>

Publish-subscribe interactions occur when a publisher generates data samples, which are then asynchronously consumed by interested subscribers.  

Publisher and subscriber ports are defined as follows:

```
pub yourPublisherName : aMessage;
sub yourSubscriberName : aMessage;
```

Time stamping can be setup to determine when a message is published and when this message is received by the subscriber by adding a **timed** keyword to the end of the port definition.  From within the application component code, the send and receive times can be requested from the component framework.

```
pub yourPublisherName : aMessage timed;
```

Subscriber ports can specify a deadline for the completion of the processing of a message by adding **within deadlinePeriodValue** to the port description.    

##### <a name="req-rep-def">Request/Reply Ports</a>

For request/reply interactions, a request message is sent from a client's request port to a server's (corresponding) reply port.  Once the message is received on the server, a reply message is sent back to the client's request port. The ports are matched (i.e. connected) based on the pair of types of the messages they carry. This message exchanged is indicated in the port definition, as shown below.  These ports can be in the same component definition or in different components.

```
req yourRequestPortName : (aRequestMessage, aReplyMessage);
rep yourReplyPortName : (aRequestMessage, aReplyMessage);
```

Time stamping can be setup to determine when a client makes a request, when the message is received by the server, when the reply message is sent back to the client, and when the client receives the reply.  This enabled by adding the **timed** keyword to the port definition, either request or reply or both.

Once a request is sent to a server, the client can continue execution of other application triggered events until the expected reply is available.  Additional requests must not be made until the reply is received so that the request and reply messaging can stay in lockstep.  For applications expecting the completion of server operations within specific time limits **within deadlinePeriodValue** can be added to the server's reply port description.  The deadline checking can also be added to the request port.

##### <a name="clt-srv-def">Client/Server Ports</a>

The client/server pattern is similar to the request/reply pattern except the interaction is synchronous.  Once a request is sent from the client to the server, the client port must explicitly wait for a reply message (using a receive operation) from the server. This message exchanged is indicated in the port definition, as shown below.  Time stamping is available by adding the **timed** keyword to the end.

```
clt yourClientPortName : (aRequestMessage, aReplyMessage);
srv yourServerPortName : (aRequestMessage, aReplyMessage);
```

Since the client port waits for the reply from the server port, the  **within deadlinePeriodValue** option is only available on the server port.

##### <a name="qry-ans-def">Query/Answer Ports</a>

The query/answer pattern is similar to the request/reply pattern, except the lockstep rule is not enforced.  An arbitrary number of requests (or queries) can be sent without an intervening reply (answer) being received.  This message exchanged is indicated in the port definition, as shown below.  Time stamping is available by adding the **timed** keyword to the end.  As in request/reply, both query and answer ports can have a deadline specification for the associated operation using the **within deadlinePeriodValue** to the definition.

```
qry yourQueryPortName : (aQueryMessage, anAnswerMessage);
ans yourAnswerPortName : (aQueryMessage, anAnswerMessage);
```

##### <a name="inside-def">Inside Ports</a>

The **inside** port is used to forward messages coming from an internal thread to a component port.  This is typically utilized in device components to isolate hardware communication that happens in a separate thread, called the I/O thread. Specify this port by

```
inside anInsidePortName;
```

Such ports are input ports for the component, i.e. if the (inner) I/O thread sends a message to this port, it will trigger a corresponding component operation. The component's operations can send messages through this port to the inner I/O thread. If the keyword **default** is added after the port name, a 1 sec periodic timer will be attached to this port, which will then trigger corresponding component operation with a 1 Hz frequency.

#### <a name="actor-def">Actor Definitions</a>

This is where the developer indicates how the components are combined to create the application.  The application is made of one or more actors.  An actor definition includes a list of component instances that are utilized, an indication of the message exposure scope, and the resource management specification.  Below is an actor definition where all the components communicate on a global level, all actors can see the message traffic for these components.

```
actor ActorName1 {
   {  
      component1InstanceName : ComponentName1;
      component2InstanceName : ComponentName2;
   }
}
```

If a message pattern (such as request/reply) exists in one or more of the component definitions and the messages need to stay local to the RIAPS node where the actor resides, then the **local** keyword is added to indicate specific messages stay within the node communication, shown below.

```
actor ActorName1 {
   local aRequestMesssage, aReplyMessage;  
   {  
      componentInstanceName1 : RequestComponent;
      componentInstanceName2 : ReplyComponent;
   }
}
```

If two or more actors on the same RIAPS node exchange a local message, it is specified as **local** in both actors.  Then both actors will see each of the local messages.  Device components can only exchange messages with local actors, therefore an application specification may be as follows:

```
actor ActorName1 {
   local aRequestMesssage, aReplyMessage;  
   {  
      componentInstanceName1 : RequestComponent;
   }
}

actor ActorName2 {
   local aRequestMesssage, aReplyMessage;  
   {  
      deviceInstanceName : ADevice;  // Reply to aRequestMessage
   }
}
```

To keep messages visible only to the other components within an actor, the **internal** keyword can be used.  This can be used in combination with the **internal** keyword as shown below where the request/reply messaging will be seen by both actors on the RIAPS node and the query/answer messages will only be visible to **ActorName1**.

```
actor ActorName1 {
   local aRequestMesssage, aReplyMessage;  
   internal aQueryMessage, anAnswerMessage;
   {  
      componentInstanceName1 : RequestComponent;
      componentInstanceName2 : QueryComponent;
      componentInstanceName3 : AnsComponent;
   }
}

actor ActorName2 {
   local aRequestMesssage, aReplyMessage;  
   {  
      deviceInstanceName : ADevice;  // Reply to aRequestMessage
   }
}
```

##### <a name="rm-model-spec">Resource Management Specifications</a>

The developer can specify a limit on the following system resource usage at the actor level:  CPU, memory, file space, and network bandwidth.  Prior to indicating the component instance definitions for the actor, a **uses {...}** clause can be added to outline the desired limits, an example is  shown below.

```
actor ActorName1 {
   local aRequestMesssage, aReplyMessage;  // local message types
   uses {
      cpu max 10 % over 1;  // CPU limit
      mem 200 mb;           // Memory limit
      space 10 mb;          // File space limit
      net rate 10 kbps ceil 12 kbps burst 1.2 k; // Network bandwidth limits
   }
   {  
      componentInstanceName1 : RequestComponent;
      componentInstanceName2 : ReplyComponent;
   }
}
```

A CPU usage limit (**cpu**) can either be a hard (using **max** keyword) or soft limit (default, no keyword).  The usage value is an integer value representing the percentage of the CPU this actor is allowed to utilize followed by a "%".  This calculation can be over a specific timeframe, indicated by **over** keyword followed by an integer value.  Time units can be specified in **sec**, **min** or **msec** (default).

```
cpu max 10 % over 1;
```

A memory usage limit (**mem**) is specified as an integer value representing the amount of memory available to the actor followed by unit of measure.  Size units available are megabytes (**mb**), kilobytes(**kb**), and gigabytes(**gb**).

```
mem 200 mb;
```

A file space limit (**space**) is specified as an integer value representing the amount of disk space available to the actor followed by unit of measure.  Size units available are megabytes(**mb**) or gigabytes(**gb**).

```
space 10 mb;
```

A network bandwidth limit (**net**) is specified by defining the target limit **rate** value an actor can use, in units of either kilobytes per second (**kbps**) or megabytes per second (**mbps**).  Optionally, a ceiling value (**ceil**) can be setup to indicate the maximum bandwidth the actor can use over the allocated bandwidth (or rate) by borrowing from any extra bandwidth available.  Additionally, a maximum allowed burst rate (**burst**) can be specified to control the amount of data that can be sent at the maximum speed before allowing another actor to use the network.  The value is provided in kilobits (**k** or **kb**).

```
net rate 10 kbps ceil 12 kbps burst 1.2 k;
```

### <a name="depl-def">Application Deployment File (*.depl*)</a>

This file sets up the hardware configuration to reflect where the actors are located relative to the physical hardware devices.  The application name here must match the application name used in the corresponding model file (*.riaps*).  The deployment specification begins with the keyword **on** and then specifies the device location and the actor that is placed on this device.  The device location can either be a specific device or indicate to place the actor on **all** devices discovered by the RIAPS controller.  The specific device can be indicated by its host name (riaps-1234) with a **.local** afterwards or by its IP address.  The hostname is indicated when you remote login into the specific hardware device, it is the command prompt available.  Below is an example of a basic deployment file.  

```
app TestApp {
    on (192.168.1.101) ActorName1;
    on (riaps-1234.local) ActorName2;
}
```

If one or more actors are being deployed to all available RIAPS nodes, then the **all** keyword can be used instead of specifying each node.  It is also possible to indicate that multiple actors are going to the same location by putting them on the same specification line.

```
app TestApp {
    on all ActorName1, ActorName2;
```

To control access to network resources, a **host** definition can be added to indicate the network connections allowed for **all** hosts or specific hosts (indicated by either IP address or hostname).  Below is an example of this definition added with several different levels of access specified.  The keyword of **any** can be used after the **network** keyword to allow full access to any Internet node, while **dns** will restrict connects to the existing subnet. Restrictions can also be set down to a specific host node by specifying the desired IP address or hostname.

```
app DistributedEstimator {
  host 192.168.57.1 {
    network any;          // Actors on this host may connect to any Internet nodes
  }
  host all {
    network dns;          // All hosts may connect to the domain name service
  }
  host riaps-452e.local {
    network 198.168.1.1;  // This host may connect to the specified IP address
  }
  on all Estimator;       // Estimator actor deployed to all nodes
  on (192.168.57.1) Aggregator(posArg=123);  // Aggregator on 192.168.57.1 only
}
```

### <a name="comp-conf-def">Component Configuration Using Parameters</a>

Components can be customized by using parameter passing to setup key features of the component.  A good example of the utilization of parameter passing can be found in the [DistributedEstimatorGpio test example](https://github.com/RIAPS/riaps-apps/tree/master/apps-vu/DistributedEstimatorGPIO).  Here you have two customized components:

* Sensor where the reported value is set by the passed value (if provided).  The component has a default behavior that can be overridden by this passed value.
*  LocalEstimator allows configure of the estimation frequency rate.

The component definitions in the model file (*.riaps*) provide the default value for the parameters.  

```
component Sensor (value=1.0) {
    timer clock 500;
    pub ready : SensorReady;
    req request : (SensorQuery, SensorValue);
}
```

The parameters can be configured in the actor definition (*.riaps*) or the deployment model (*.depl*).  Values defined in the deployment model will be passed to the actor definition and overrides the value indicated by the actor definition.  Values defined at the actor level will override the default values set at the component definition.  It is anticipated that application developers will create reusable components that provide configurability, hence the parameter setup with possible defaults.  Then other application developers can utilize the components for their specific algorithms and expected component configuration, which can lead to more specific component configurations to meet the application needs but still not be specific to any particular hardware configuration.  Finally, the deployment managers know the complete system configuration and can deploy the application actors on specific hardware sets which can alter the desired parameter setup further.

The actor definition for the 'DistributedEstimatorGpio' example shows that passing the frequency value for the 'LocalEstimator' component will be required from the deployment model, while the 'Sensor' component value information is optional since there is a default value indicated in the actor definition.

```
actor Estimator (freqArg,value=0.0) {
    local SensorReady, SensorQuery, SensorValue, Blink;
    {
        sensor : Sensor(value=value);
        filter : LocalEstimator(frqArg=freqArg);
        gpio : GPIODevice();
    }
}
```

Here is an example where the deployment model defines the final setup of the application.

```
app DistributedEstimatorGpio {
    on (riaps-89a5.local) Estimator(freqArg=0.5,value=1.0);  // 0.5 Hz update rate
    on (riaps-a06e.local) Estimator(freqArg=1.0,value=2.0);  // 1 Hz update rate
    on (riaps-398d.local) Estimator(freqArg=2.0,value=3.0);  // 2 Hz update rate
    on (192.168.1.101) Aggregator();  
}
```

### <a name="adv-model-opts">Advanced Modeling Options</a>

In addition to the basic modeling language elements described above, there are additional features available in RIAPS that can be configured in the model file.

* <a name="adv-sched">[Message Scheduling](sched.md)</a>
* <a name="adv-groups">[Distributed Coordination (Groups)](groups.md)</a>
