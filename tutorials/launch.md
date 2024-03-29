## RIAPS Application Management Tutorial

Each RIAPS system includes a dedicated control node that will launch and control applications using the RIAPS control app (***riaps_ctrl***) to interact with available RIAPS hosts.  Each available RIAPS host runs a deployment control application (***riaps_deplo***), which is typically running as a background process on the remote hardware.  This tutorial will explain tips and tricks for the application management process and verifying the system is working as desired.

### Page Contents

* [RIAPS Control App](#app-start)
  * [Application Startup](#app-start)
  * [RIAPS Node Registration](#node-reg)
  * [Application Selection](#app-select)
  * [Application Deployment](#app-deploy)
* [RIAPS Node Debugging Tips](#node-deploy-tips)
  * [Remote Node Status Check](#sys-check)
  * [Remote Node Reset](#remote-reset)
  * [Remote Node Configuration](#remote-config)
* [Application Deployment Debugging Tips](#app-debug-tips)
  * [Missing RIAPS Node](#missing-node)
  * [SSH Key Issue](#key-issue)

### <a name="app-start">RIAPS Control App</a>

#### <a name="app-start">Application Startup</a>

To find the available RIAPS hosts, the control node must be configured to see the network interface where the host nodes are connected.  See **[Configuring Environment for Local Network Setup](https://github.com/RIAPS/riaps-integration/tree/master/riaps-x86runtime#configuring-environment-for-local-network-setup)** for instructions on setting this configuration option to the desired network interface.  If the network configuration option is updated, then the ***riaps-rpyc-registry*** systemd service used in registering RIAPS nodes needs to be restarted (command shown below) in order to monitor the correct network interface.

```
sudo systemctl restart riaps-rpyc-registry.service
```

Starting the RIAPS control app (***riaps_ctrl***) will launch a controller graphical application (shown below).  This interface will be used to select the application location and the model files used to define the application and its deployment.   A visual of the various elements used in the ***riaps_ctrl*** tool is found in the [controller implementation discussion](https://github.com/RIAPS/riaps.github.io/blob/master/impl.md#riaps_ctrl).

![Controller Graphical Application](../img/riaps-ctrl.png)

The control app also starts ***redis-server*** in the background.  If the control application started up successfully, the logging information (in the terminal where the ***riaps_ctrl*** was started) will state that it is "Ready to accept connections".  If the control app was not shutdown properly and left the ***redis-server*** process running, restarting the control app will indicate that the socket at port 6379 has the "Address already in use".  The control app should be shutdown and the running ***redis-server*** process should then be removed (using ```sudo pkill -9 redis-server```).  The ***riaps_ctrl*** tool can then be restarted to find the correct port connection.

* Successful launch of ***riaps_ctrl***

  ```
  riaps@riaps-devbox:~/$ riaps_ctrl
  29220:C 18 Feb 14:13:44.670 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  29220:C 18 Feb 14:13:44.670 # Redis version=4.0.11, bits=64, commit=fd6b747b, modified=0, pid=29220, just started
  29220:C 18 Feb 14:13:44.670 # Configuration loaded
  29220:M 18 Feb 14:13:44.670 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                  _._                                                  
             _.-``__ ''-._                                             
        _.-``    `.  `_.  ''-._           Redis 4.0.11 (fd6b747b/0) 64 bit
    .-`` .-```.  ```\/    _.,_ ''-._                                   
   (    '      ,       .-`  | `,    )     Running in standalone mode
   |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
   |    `-._   `._    /     _.-'    |     PID: 29220
    `-._    `-._  `-./  _.-'    _.-'                                   
   |`-._`-._    `-.__.-'    _.-'_.-'|                                  
   |    `-._`-._        _.-'_.-'    |           http://redis.io        
    `-._    `-._`-.__.-'_.-'    _.-'                                   
   |`-._`-._    `-.__.-'    _.-'_.-'|                                  
   |    `-._`-._        _.-'_.-'    |                                  
    `-._    `-._`-.__.-'_.-'    _.-'                                   
        `-._    `-.__.-'    _.-'                                       
            `-._        _.-'                                           
                `-.__.-'                                               

  29220:M 18 Feb 14:13:44.671 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
  29220:M 18 Feb 14:13:44.671 # Server initialized
  29220:M 18 Feb 14:13:44.671 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
  29220:M 18 Feb 14:13:44.671 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
  29220:M 18 Feb 14:13:44.671 * Ready to accept connections
  ```

* Unsuccessful launch of ***riaps_ctrl***

  ```
  riaps@riaps-devbox:~$ riaps_ctrl
  29542:C 18 Feb 14:30:49.074 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  29542:C 18 Feb 14:30:49.074 # Redis version=4.0.11, bits=64, commit=fd6b747b, modified=0, pid=29542, just started
  29542:C 18 Feb 14:30:49.074 # Configuration loaded
  29542:M 18 Feb 14:30:49.075 * Increased maximum number of open files to 10032 (it was originally set to 1024).
  29542:M 18 Feb 14:30:49.075 # Creating Server TCP listening socket *:6379: bind: Address already in use
  ```

#### <a name="node-reg">RIAPS Node Registration</a>

Once the control application is running, available RIAPS nodes will begin to register and be revealed as a column in the App/Node section.  The bottom section of the controller will also log the registration of a node by indicate plus sign and the IP address of the node added: **+192.168.1.103**. When nodes are removed, the node's column will be removed and the logging section will show a minus sign and the IP address of the node being removed: **-192.168.1.103**.

If security is enabled on the controller node, then only the RIAPS nodes with security enabled and with the correct key configuration will be found by the controller.  If security is turned off, then only the RIAPS nodes with security disabled will be located by the RIAPS Control, secure RIAPS nodes will not be available.  Once all the desired nodes are available, the application can then be selected and deployed to these nodes (per the deployment model file).

#### <a name="app-select">Application Selection</a>

The file selection (the file folder under the menus) is the location of the component code that will be downloaded to the available RIAPS nodes when the application is launched.  When working with Python components only, selecting the appropriate folder is all that is necessary.  If you are working with C++ elements of a components (i.e. using cython), then make sure to compile the component code to create library files (*.so*) to the appropriate hardware architectures (amd64 or armhf or arm64) and have them available in the selected folder.  During development, if changes are made to the component code be sure to recompile the library file.

When selecting the **Model** and **Depl** (or deployment) files, the selected files are compiled as they are loaded.  If the file is valid and compiles correctly, as message will appear in the lower portion of the application to indicate that the file has been compiled and no errors are presented.  If there is an error in these files as they get loaded, an error will be presented in addition to the compiling message.  This error message should provide information on the first error found while compiling.  Another method to check the validity of these model files is to scripts to individually compile these files.  ```riaps_lang <model filename>``` can be used to check the model file (*.riaps*) and ```riaps_depll <depl filename>``` can be used to check the deployment file (*.depl*).  Using these check tools will also provide more information on the location of the first error found.

Once valid model and deployment files are loaded, the user can view how this application will be deployed given the specified RIAPS nodes by using the **View** button (right side of the controller application - see image below).  Looking at this view of the deployable application will help the developer and deployment manager to understand how the components of the application will be distributed across the available RIAPS nodes.  This can be a useful system level debugging tool.

![Application Viewer](../img/riaps-ctrl-view.png)

#### <a name="app-deploy">Application Deployment</a>

After the application is ready, the application is deployed to the available nodes by selecting the **Deploy** button.  To launch an application on the nodes, click on the **AppName** button that appeared when the application was deployed (in the image below, the AppName=DistributedEstimator).  The **Launch** button is then available.  This is also where the application management options are found: **Stop** and **Remove**.  Stop just halts the running application, leaving the application code on the RIAPS nodes.  Remove is used to delete the application code and resulting logging files from the RIAPS node.

![Deploying Application](../img/riaps-ctrl-deploy.png)

When an application successfully deploys, the list of actors loaded on each RIAPS node will be indicated below the appropriate node's column and will turn green when running.  When the application is stopped (but not removed), these actor indicates will be in a gray box.  Also, the logging information in the controller shows that the application has been installed by indicating **I** and then the node and application installed.  Once successfully installed remotely, the application in launched, which is indicated by **L** and then the node, application and actors started (show in image below).

![Deployed Application](../img/riaps-ctrl-deployed.png)

When stopping an application, the logging will be indicated with a **H** and then the node, application and actors removed.  When all application is removed from the RIAPS nodes, the logging will be indicated with a **R** and the application name.

>NOTE: If file logging is utilized (see [Component Logging Tutorial](logging.md#logging-custom)), then the user MUST retrieve the log files before removing the application from the nodes or the data will be lost.

The easiest way to grab log files from RIAPS node is use [***riaps_fab***](https://github.com/RIAPS/riaps-pycom/tree/master/src/riaps/fabfile) command **riaps.getAppLogs**.  Since using ***riaps_fab*** can pull from multiple RIAPS nodes and all the log files for the application on each node are named the same and the logs will be placed in folders with the node hostnames under a `logs` folder.  To transfer application log files from specific hosts, use `-H` option to indicate the desired hostname.  This command will get the files from the application deployment folder (riaps_apps/appname).  The log file location on the remote nodes is configurable in the *riaps-log.conf* file included with the application (see [Customizable Component-Level Logging](logging.md#logging-custom)).  The following example command shows using the command to grab the DistributedEstimator application logs from a single node (riaps-a123.local).  

```
riaps_fab riaps.getAppLogs:DistributedEstimator -H riaps-a123.local
```

### <a name="node-deploy-tips">RIAPS Node Debugging Tips</a>

When an application is deployed in the field, the available RIAPS nodes are typically remote computing nodes (such as the Beaglebone Black).  But developers may choose to debug component code locally on the controller node (or VM), which is handled slightly differently (indicated below).  The remote computing nodes are configured to automatically run the **riaps-deplo.service** when the system is booted up.  This allows easy remote startup of applications.

When debugging applications on the controller node, the RIAPS deployment application needs to be started manually using ```sudo -E riaps_deplo```.  To communicate with this deployment instance using riaps_fab, utilize the local reference for the node:  i.e. ```riaps_fab sys.check -H localhost```.  

#### <a name="sys-check">Remote Node Status Check</a>

To verify if the desired remote nodes are available for communication, utilize the ```riaps_fab sys.check``` command, see [RIAPS Fabric File Information](https://github.com/RIAPS/riaps-pycom/tree/master/src/riaps/fabfile) for setting up the *riaps-host.conf* file.  Successful communication will be when all remote nodes indicate their hostnames and kernel information.  It is possible to reboot or shutdown a specific host using the following commands respectively:  ```sys.reboot -H <hostname>``` and ```sys.shutdown -H <hostname>```.

To see the application logging activity on the remote nodes, ssh into the desired node and look at the system logs (see [Default Logging tutorial](tutorial/logging.md#logging-default)).

#### <a name="remote-reset">Remote Node Reset</a>

During application development, there may be several ways a system may appear to be non-responsive.  A component may not be handling request/reply communication appropriately and the application hangs up (i.e. request made and then another request made before a reply is received).  Or when developing a device component that utilizes system ports (such as UART) that are not functioning as desired and the port is now stuck open.  These are just a couple of instances to consider.  To reset the system, there are two ways to reset the RIAPS framework on the RIAPS nodes:  using the RIAPS Controller application or ***riaps_fab*** command.  

##### Reset using RIAPS Control Application

On the RIAPS Control application, the **Select** menu item contains a **Reset** option that will send a command to all registered RIAPS nodes to kill the riaps processes (such as running actors and the deployment service) and reset the node.  There is also an option to **Halt** the deployment function on all nodes.  On the remote RIAPS nodes, the deployment service is setup to restart if it stops.  Therefore after the RIAPS processes are either killed or halted, the system will be reset and the RIAPS node will be removed from the available node list and then added again once the deployment service starts again.  The manually started RIAPS deployment services will not be restarted automatically, such as on the development VM.

![Reset Menu Item](../img/riaps-ctrl-menu1.png)

##### Reset using Fabric Command

 Another way to reset the RIAPS node is to run ```riaps_fab riaps.reset```.  This will reset the RIAPS processes on the remote nodes identified in the *riaps-hosts.conf* file.  Once again, the deployment service will restart after this process is complete.

#### <a name="remote-config">Remote Node Configuration</a>

At times, the configuration setup of the remote nodes needs to be modified.  This can be accomplished by editing a local copy of the [*riaps.conf* file](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/etc/riaps.conf) and then transferring this file to the remote nodes using the ***riaps_fab*** command below to put the file on the remote node.  The same can be done with the */riaps/etc/riaps-log.conf* file.

```
riaps_fab riaps.updateConfig
```

The deployment service reads in the configure files at the beginning of execution of this service.  Therefore changes made to the configurations will not take effect until the deployment service is restarted.  Methods described in the [previous reset discussion](launch.md#remote-reset) can be used to restart the deployment service.  Or the following sequence of ***riaps_fab*** commands can be used:

```
riaps_fab deplo.stop
riaps_fab deplo.start
```

### <a name="app-debug-tips">Application Deployment Debugging Tips</a>

#### <a name="missing-node">Missing RIAPS Node</a>

If an application model includes a RIAPS node specification for a node that is not currently available, the application can be deployed on the available nodes.  The missing node will be indicated in the controller log section by a **?** and the node name specified by the deployment model (*.depl*).  Either determine why this node is offline or pick an available node.

![Missing RIAPS Node](../img/riaps-ctrl-deploy-missing.png)

#### <a name="key-issue">SSH Key Issue</a>

When deploying an application, the available RIAPS node's ssh keys (public and private) should match the ssh keys of the controller node.  These keys are located in */usr/local/riaps/keys* folder (*id_rsa.pub* and *id_rsa.key*).  If they do not match (even when security is turned off on both the controller and the RIAPS node), then an **App download fault** will occur with and indication that there was an **Error while verifying app 'appName'** (see image below).  It will also indicate the RIAPS node where the deploy issue occurred.

![SSH Key Issue with RIAPS Node](../img/riaps-ctrl-deploy-appfault.png)
