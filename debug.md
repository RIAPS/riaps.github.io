# Using the Python Debugger for RIAPS Components

This section describes how to debug RIAPS components without modifying them.

We assume that Eclipse is used for developing RIAPS application components. The Python
development environment has a debugger that can be used for debugging the components
running on the RIAPS target nodes while the graphical front-end of the debugger is
running on the development machine. The debugger uses the Python source code
of the components located on development host.

Note: The same debugger can also be used for debugging the RIAPS framework itself.

When an actor (or a device component that runs in its own device actor) on the target node
is started it must connect to the debugger front end that runs on the development host.
This is controlled by a configuration attribute in the **/usr/local/riaps/etc/riaps.conf**
file on the target node. See the [RIAPS Configuration Files](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/etc/README.md) information for how to configure the target nodes for debugging.

Having configured the target node a debugging session can be started as follows:

1. Launch Eclipse, open the project that contains the code for the app components.
Open a component and set a breakpoint somewhere (e.g. in the constructor, or at any operation).

2. Launch the riaps_ctrl application on the development host. Launch the deployment manager on
the target host (unless it is already running).

3. In Eclipse, select the Debug perspective and launch the debug server (Pydev/Start Debug Server).

4. Use riaps_ctrl to launch the application from the development host on the target hosts.

5. When the riaps_actor (or riaps_device) is started, it will link up to the debugger running
on the development host. This event will activate the debugger, that will stop the code running
on the target machine at a specific place within the RIAPS code. The **Resume(F8)** button should
be used at this point to let the program proceed. <br/>

> Note: This will happen for each actor/device started on the target node. Watch out for the processes running under the Debug Server with the name **unknown** -- these are the remote actors/devices running on the target. Each one of these should be **resumed** after the initial connection.

6. After the initial connection to the debugger has been made, the debugger will stop the target
code at a breakpoint, and the usual debugging activities can commence.

7. The app should be managed / terminated using the riaps_ctrl, as usual. Note that whenever
the app is restarted, the connection to the front-end debugger will take place.

8. Once debugging is finished, the riaps.conf file should be restored to its original state
(i.e. no values for the actor_debug_server / device_debug_server parameters). This will turn off
debugging on the target node.
