# RIAPS Application Management Tutorial

Each RIAPS system includes a dedicated control node that will launch and control applications using the RIAPS control app (**riaps_ctrl**) to interact with available RIAPS hosts.  This tutorial will explain how to deploy a RIAPS application using the control app and how to verify the system is working as desired.
>MM TODO:  update summary after developing more of below

## RIAPS Deployment Service

RIAPS applications are deployed to available RIAPS nodes utilizing a central control application

>MM TODO:  Below are notes on what to include, not complete yet just thought capturing
- Make sure development system nic name is correct
  -  sudo systemctl restart riaps-rpyc-registry.service
- Starting RIAPS tools from Eclipse (ctrl and deplo)
  - reference img/eclipse-tool-start.png
  - how to import tools into a new workspace
- riaps_deplo in VM vs. BBB
- loading up an applications
  - model files
  - viewing the model and deployment (view button)
- Interpreting feedback in control app
- Using kill in control app or fab scripts
- indicate how to tell when system needs resetting (during debugging session)
  - riaps_ctrl:  redis-server
  - riaps_deplo:  discovery already running, application actors still running, req/rep port communication hang
- Fab file usage
  - Useful Commands
    - Fab help
    - sys.check, sys.reboot, sys.shutdown,
    - Riaps.update
    - riaps.kill, deplo.stop/deplo.start
    - Command nodes individually:  “-H localhost” for example
    - Configuration file updates


## RIAPS Control App (riaps_ctrl)

>MM TODO:  summary information

To find the available RIAPS hosts, the control node must be configured to see the network interface where the host nodes are connected.  See [Configuring Environment for Local Network Setup](https://github.com/RIAPS/riaps-integration/tree/master/riaps-x86runtime#configuring-environment-for-local-network-setup) for instructions on setting this configuration option to the desired network interface.  If the network configuration option is updated, then the **riaps-rpyc-registry** systemd service used in registering RIAPS nodes needs to be restarted (command shown below) in order to monitor the correct network interface.  A visual of the various elements used in the riaps_ctrl tool is found in the [implementation discussion](https://github.com/RIAPS/riaps.github.io/blob/master/impl.md#riaps_ctrl).

```
sudo systemctl restart riaps-rpyc-registry.service
```

Starting the RIAPS control app (riaps_ctrl) will launch a controller graphical application (shown below).  This interface will be used to select the application location and the model files used to define the application and its deployment.  

![Controller Graphical Application](../img/riaps-ctrl.png)

The control app also starts redis-server in the background.  If the control application started up successfully, the redis-server will state that it is "Ready to accept connections".  If the control app was not shutdown properly and left the redis-server process running, restarting the control app will indicate that the socket at port 6379 has the "Address already in use".  The control app should be shutdown and the running redis-server process should then be removed.  The riaps_ctrl tool can then be restarted to find the correct port connection.

* Successful launch of riaps_ctrl

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

* Unsuccessful launch of riaps_ctrl

  ```
  riaps@riaps-devbox:~$ riaps_ctrl
  29542:C 18 Feb 14:30:49.074 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  29542:C 18 Feb 14:30:49.074 # Redis version=4.0.11, bits=64, commit=fd6b747b, modified=0, pid=29542, just started
  29542:C 18 Feb 14:30:49.074 # Configuration loaded
  29542:M 18 Feb 14:30:49.075 * Increased maximum number of open files to 10032 (it was originally set to 1024).
  29542:M 18 Feb 14:30:49.075 # Creating Server TCP listening socket *:6379: bind: Address already in use
  ```










>MM TODO: Platform updates: this may not belong here, but staging it here until figure out where belongs (maybe on setup.md, if not already there)
Update procedures for the VM, framework, BBB
Downloads:  https://riaps.isis.vanderbilt.edu/downloads/
For VM: ./riaps_install_amd64.sh
For BBBs:  fab riaps.update
