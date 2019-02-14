## RIAPS Tutorials

RIAPS applications are built from distributed computing components that work together.  The configuration and interactions of the components, and how they are combined to create processes (called actors) are defined within a RIAPS model file (.riaps).  The defined actors are then allocated to the specific system nodes in the network using a deployment model file (.deplo).  An explanation of the language used to create these two key files can be found in the [Application Model and Deployment Files Tutorial](tutorials/models.md).

For Eclipse, a RIAPS plug-in is available to assist in the creation of application model and deployment files.  This tool will guide the user to the available keywords for the specific definition elements outlined in the model tutorial.  Information on utilizing this tool can be found on the [RIAPS Domain-Specific Modeling Language (DSML) Tool Tutorial](tutorials/dsml.md).  This plug-in utilizes a code generation tool that can also be used from the command line.  Information on the **riaps_gen** code generation tool can be found in the [RIAPS Code Generation Tutorial](https://github.com/RIAPS/riaps-pycom/tree/master/src/riaps/gen/README.md).

Component implementation can be written in either Python or C++.  Python allow quick prototyping of algorithms using a simple language.  Those not requiring fast update rates or actions are well suited for a Python implementation.  Tutorial information on moving from the model file to creating a Python component can be found on the [Python Component Development Tutorial](tutorials/pyapps.md).  For components requiring fast timing or developers wanting to use C++, refer to the [C++ Component Development Tutorial](https://github.com/RIAPS/riaps-core/wiki).

A RIAPS application is launched and controlled using the **riaps_ctrl** tool.  This tool interacts with the available RIAPS network nodes on which the application can be launched.  The [RIAPS Application Management Tutorial](tutorials/launch.md) provides information on running the controller on a host machine and launching the applications on the nodes using the **riaps_deplo** tool (either automatically or manually).  

Logging files are useful in the debugging and monitoring of applications.  The [Component Logging Tutorial](tutorials/logging.md) describes ways to view logs on the RIAPS nodes or monitor it remotely. There is also a [Debugging RIAPS Apps Tutorial](tutorials/debug.md) that can provide information on different ways to debug an application during development or when something goes wrong.

### Application Tutorials

This is a set of specific application types to help illustrate some options available for developers to consider when starting design.  When possible both a Python and C++ implementation will be available.  

>NOTE:  YET TO BE DEVELOPED:  expect to have a short videos for these and accompanying code snippets

#### Hello World App
This application will show a **Timer** triggering a component.  The component prints a message.

#### Simple Publish/Subscribe App
1 Pub / 1 Sub: same as above, deployed on 1, then 2 nodes

#### Simple Request/Reply App
1 Req / 1 Rep: same as above, deployed on 2 nodes

#### Complex example: Distributed Estimator App

#### Utilizing Parameters in Application Definitions

#### Additional Port types
Show examples of client/server and query/answer

#### Timer programming
Show periodic and sporadic timer usage, along with purposeful disabling of the timer port.

#### Simple Device Component Example
Utilize the EchoIO code to show a simple device component

#### Advanced Device Component Example
Show UART device component details (including use of multithreading - i.e. inside ports).  This example uses two different RIAPS nodes communicating via UARTs

#### Device Component Library
C37, Modbus, etc.

#### Time Synchronization Example
Setting up time sync configuration for each RIAPS node (and any other real-time system setups)

#### How to Utilize the ChronoCape for GPS
This is an add-on board developed at Vanderbilt University for use in working with time synchronization applications
