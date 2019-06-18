## RIAPS Tutorials

RIAPS applications are built from distributed computing components that work together.  The configuration and interactions of the components, and how they are combined to create processes (called actors) are defined within a RIAPS model file (.riaps).  The defined actors are then allocated to the specific system nodes in the network using a deployment model file (.deplo).  Here is a short video showing [how to create, deploy and run a simple RIAPS application]
(tutorials/app-examples/hello-world.md or youtube video) utilizing Eclipse development tool and a RIAPS plug-in to assist in the file creation.

Component implementation can be written in either Python or C++.  Python allow quick prototyping of algorithms using a simple language.  Those not requiring fast update rates or actions are well suited for a Python implementation.  For components requiring fast timing or developers wanting to use C++, refer to the [C++ Component Development Tutorial](https://github.com/RIAPS/riaps-core/wiki).

An explanation of the language used to create these RIAPS model and deployment files can be found in the [Application Model and Deployment Files Tutorial](tutorials/models.md).  The RIAPS plug-in utilizes a code generation tool that can also be used from the command line.  Information on the **riaps_gen** code generation tool can be found in the [RIAPS Code Generation Tutorial](https://github.com/RIAPS/riaps-pycom/tree/master/src/riaps/gen/README.md).

A RIAPS application is launched and controlled using the **riaps_ctrl** tool.  This tool interacts with the available RIAPS network nodes on which the application can be launched.  The [RIAPS Application Management Tutorial](tutorials/launch.md) provides command line information on running the controller on a host machine and launching the applications on the nodes using the **riaps_deplo** tool (either automatically or manually), along with hints of situations that may occur when the developing a new application.

Logging files are useful in the debugging and monitoring of applications.  The [Component Logging Tutorial](tutorials/logging.md) describes ways to view logs on the RIAPS nodes or monitor it remotely. There is also a [Debugging RIAPS Apps Tutorial](tutorials/debug.md) that can provide information on different ways to debug an application during development or when something goes wrong.

### Application Tutorials

Here is a set of videos describing different features of the RIAPS platform and how they can be utilized in application development.  When possible both a Python and C++ implementation will be available.  

>NOTE:  YET TO BE DEVELOPED:  expect to have a short videos for these and accompanying code snippets

* [Simple Publish/Subscribe App](tutorials/app-examples/pub-sub.md)
* [Simple Request/Reply App](tutorials/app-examples/req-rep.md)
* [Complex example: Distributed Estimator App](tutorials/app-examples/complex-app.md)
* [Utilizing Parameters in Application Definitions](tutorials/app-examples/parameters-app.md)
* [Additional Port types](tutorials/app-examples/other-ports.md)
* [Timer programming](tutorials/app-examples/timer-app.md)
* [Simple Device Component Example](tutorials/app-examples/simple-device.md)
* [Advanced Device Component Example](tutorials/app-examples/multithread-device.md)
* [Device Component Library](tutorials/app-examples/device-library.md)
* [Time Synchronization Example](tutorials/app-examples/time-sync.md)
* [How to Utilize the ChronoCape for GPS](tutorials/app-examples/chronocape.md)
