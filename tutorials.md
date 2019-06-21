## RIAPS Tutorials

Informations is provided in several formats to assist users in understand the system concepts and options better.  Videos will be utilized to explain system concepts and examples of code that leverage the feature discussed.  Any code used in the videos will be provided to all users to try out the concepts on their systems.  Written tutorials will outline available options and provide supporting information on tips and tricks available.

### Tutorial Legend

* Key concepts that users might be looking to find will be shown in bold.  **log levels**
* Key words in RIAPS configuration files will also be shown with single quotes:  'security'
* Key file or folder locations will be shown in italics:  */usr/local/riaps/etc/riaps.conf*
* Links to other information will be in blue: [this page link](https://riaps.github.io/tutorials.html)
* When discussing RIAPS tools, third party tools or tool menu items, they are shown in bold and italics: ***riaps_ctrl***
* Code Snippets (either inline or segment):  ```sudo -E riaps_deplo```
* When code snippet contains portions that relate specifically to the system deployment, the information that needs to be tailored to the specific setup will be indicated by ```<information to tailor>```.  The code can be copied, but should be adjusted to fit the current needs.  The brackets (\<>) are also removed.  An example would be when the application name may be needed and will be indicated as \<app name>.  

### General Tutorials

#### Starting Point

RIAPS applications are built from distributed computing components that work together.  The configuration and interactions of the components, and how they are combined to create processes (called actors) are defined within a RIAPS model file (.riaps).  The defined actors are then allocated to the specific system nodes in the network using a deployment model file (.deplo).  Here is a short video showing [how to create, deploy and run a simple RIAPS application](tutorials/app-examples/hello-world.md) utilizing Eclipse development tool and a RIAPS plug-in to assist in the file creation.

#### Application Development

Component implementation can be written in either Python or C++.  Python allow quick prototyping of algorithms using a simple language.  Those not requiring fast update rates or actions are well suited for a Python implementation.  For components requiring fast timing or developers wanting to use C++, refer to the [C++ Component Development Tutorial](https://github.com/RIAPS/riaps-core/wiki).

An explanation of the language used to create these RIAPS model and deployment files can be found in the [Application Model and Deployment Files Tutorial](tutorials/models.md).  

The RIAPS Eclipse plug-in utilizes a code generation tool that can also be used from the command line.  Information on the ***riaps_gen*** code generation tool can be found in the [RIAPS Code Generation Tutorial](https://github.com/RIAPS/riaps-pycom/tree/master/src/riaps/gen/README.md).

#### Application Management

A RIAPS application is launched and controlled using the ***riaps_ctrl*** tool.  This tool interacts with the available RIAPS network nodes on which the application can be launched.  The [RIAPS Application Management Tutorial](tutorials/launch.md) provides command line information on running the controller on a host machine and launching the applications on the nodes using the ***riaps_deplo*** tool (either automatically or manually), along with hints of situations that may occur when the developing a new application.

#### Debugging Tutorials

Logging files are useful in the debugging and monitoring of applications.  The [Component Logging Tutorial](tutorials/logging.md) describes ways to view logs on the RIAPS nodes or monitor it remotely.

There is also a [Debugging RIAPS Apps Tutorial](tutorials/debug.md) that can provide information on different ways to debug an application during development or when something goes wrong.

### RIAPS Feature Tutorials

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
