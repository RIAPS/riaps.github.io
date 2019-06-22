## Component Logging Tutorial

The RIAPS framework utilizes **spdlog** (a fast C++ logging library) to provide a logging mechanism for both the RIAPS framework and the application components.  There are 3 ways to log information within a component:  default, actor-level, and customizable component-level logging.  

### Page Contents
* [Default Logging](#logging-default)
* [Actor-Level Logging](#logging-actor)
* [Customizable Component-Level Logging](#logging-custom)
* [RIAPS Framework Logging](#logging-framework)

### <a name="logging-default">Default Logging</a>

Use of logging within a component is simple.  Utilize the RIAPS component ```self.logger.<log level>``` function to provide user information.  Here is an example:

```python
self.logger.info("on_tempupdate(): Temperature:%s, PID %s, Timestamp:%s" % (temperatureValue, str(now), temperatureTime))
```

The available **log levels** are listed below in order of priority (from low to high).  The RIAPS platform sets a log level of **info** by default.  At this level, you will receive all messages at the info level and others of higher priority (such as warn, error and critical).
- trace
- debug
- info
- warn
- error
- critical

For debugging, you can adjust the logging level using the following in the application component code.

```python
import spdlog as spd

logger.set_level(spd.LogLevel.INFO)
```

The log messages are sent to the console output on the node.  If working on the development VM and starting the ***riaps_deplo*** using either Eclipse or ```sudo -E riaps_deplo``` in a terminal window, the console output will be available.  The information provided will be both the RIAPS platform and the application component logging information.

The BBB nodes run ***riaps_deplo*** as a systemd service, so the console log information is placed in the system logs.  To view the information as the application is running, ssh into a BBB node and run the ***journalctl*** command or you can view an individual BBB journal file using a fabric command.

* SSH to BBB:
  ```bash
  sudo journalctl -u riaps-deplo.service -f
  ```

* Fabric Command from VM:
  ```
  fab deplo.journal:n=200,grep=WeatherMonitor -H bbb-a2c4.local > debug-log.log

  where
    - n is the number of lines of the journal desired, defaults to 10
    - grep='' allows one word search narrowing,
      it is case sensitive and only search number of lines pulled (n=)
    - '-H' allows specification of a single BBB hostname
      (either IP address or bbb-xxxx.local format),
      defaults to all RIAPS hosted specified in fabric riaps_host.py file
    - use '>' to direct output to a file, if desired
  ```


### <a name="logging-actor">Actor-Level Logging</a>

A global way to log **Actor** information to a file in the deployed application directory is to turn on the RIAPS Framework application log option in the [RIAPS Framework Configuration file](https://github.com/RIAPS/riaps-pycom/tree/develop/src/riaps/etc/README.md).  This is done by adding the 'log' designations to the 'app_logs =' configuration line, as shown below.  This file is located on each RIAPS node in */usr/local/riaps/etc/riaps.conf*.  Each actor defined in the application model file will have a log file in */home/riaps/riaps_apps/\<app name>/*.  

```python
# application logs
# ''        -- stdout
# log       -- app/actor.log file
app_logs = log
```

For the remote RIAPS nodes, the *riaps.conf* can be modified on the development VM and then moved to the remote nodes using the [fabfile utility](https://github.com/RIAPS/riaps-pycom/tree/master/src/riaps/fabfile). Start with [original *riaps.conf* file](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/etc/riaps.conf) and configure appropriately for your system configuration (including the NIC name). Make sure the RIAPS nodes are accessible from the fab command (see ```fab sys.check```). Then use the ```fab riaps.updateConfig``` command to transfer the locally edited *riaps.conf* file to the remote nodes.

Since applications are deployed under a generated username, the developer must have root access to view the log messages in the */home/riaps/riaps_apps/\<app name>* directory.  When writing to a log file, output to this file happens in batches so it can take some time for the data to appear in the desired file (on the order of a minute or more).

The */home/riaps/riaps_apps/\<app name>* directory is only available when the application is deployed.  Removing an application using the RIAPS controller will delete the log information.  It is anticipated that when an application is deployed in a real system, it could be stopped and restarted or just left continuously running throughout the life of the application on the system.  Therefore, logged information will remain available to system operators.  During application development, the developer will be deploying and removing the application during debugging efforts.  So it is recommended to copy the logs to another location if access is needed at a later time (for debugging).  The log files can be moved from a remote node to the development machine using the following fabric utility command from the development machine (passing where to locate the file and then where to place the file locally).  The 'true' is needed since the logs are located in a privileged location that requires 'sudo' in the get command.

```
sys.get:riaps_apps/DistributedEstimator/log/*,.,true -H bbb-8014.local
```

> Note: It is not recommended to leave actor-level logging on in a real system deployed application unless the logging is for errors and warnings only, in order to minimize disk usage.  Consider using customizable component-level logging instead.

### <a name="logging-custom">Customizable Component-Level Logging</a>

The application developer can customize the logging setup for each application component instance by defining a *riaps-log.conf* file and placing it in the application directory with the components.  This file defines the logging configuration and output pattern for each of the application components instances, thus providing flexibility for the developer to direct and format the output of each component instance individually.  

The logging configuration ('[[logger]]') indicates the component instance for the specific configuration definition, how the output will be handles, and the output format used.  When the actors were defined in the application model file (*.riaps*), a name was given for each instance of the application component used in the actor.  Using the [WeatherMonitor example application](https://github.com/RIAPS/riaps-apps/blob/master/apps-vu/WeatherMonitor/Python/wmonitor.riaps), an actor definition is shown below.  The 'name' for this component instance will be  'WeatherIndicator.sensor', where 'WeatherIndicator' is the actor's name and 'sensor' is the component instance name.

```
    actor WeatherIndicator {
       {  // TempSensor publishes 'TempData' messages
          sensor : TempSensor;
       }
    }
```

In spdlog terms, a sink ('[[sink]]') is an interface between a logger instance and its desired target. Each logger is associated with one or more sink objects that actually write the log contents to a specific target such as a file, console or database. Each logger instance also contains its own private instance of a formatter object, which is responsible for the visual representation of the logged data. It can be customized with user-defined patterns ('[[pattern]]') that specify the log format.

Console and file sink types available are:

- ConsoleLogger
  - Standard output: 'stdout_sink (_mt or _st)'
  - Colored standard output: 'stdout_color_sink (_mt or _st)'
  - Standard error output: 'stderr_sink (_mt or _st)'
  - Colored standard error output: 'stderr_color_sink (_mt or _st)'
- FileLogger
  - Basic file sink that writes to a given log file: 'simple_file_sink (_mt or _st)'
  - Rotating log files: 'rotating_file_sink (_mt or _st)'
  - Create new file every day at a specified time: 'daily_file_sink_st (_mt or _st)'

> Note: Utilize either single threaded ('_st') or thread safe multi-threaded ('_mt') loggers.  While single threaded sinks cannot be used from multiple threads simultaneously, they are faster since no locking is employed.  For a full list of loggers, see [Supported Sinks](https://github.com/guangie88/spdlog_setup#supported-sinks)

For the rotating log file, when the maximum size is reached, a new file is created and the logger switches to a new file. The maximum size of each file and the maximum number of files can be configured.

The **pattern** section defines the expected format of the logging message, any identifying text, and the message logged by the component in the code.  The pattern flag options are listed in a [spdlog custom formatting tutorial](https://github.com/gabime/spdlog/wiki/3.-Custom-formatting).

Using the example [WeatherMonitor application](https://github.com/RIAPS/riaps-apps/tree/master/apps-vu/WeatherMonitor/Python), an example  *riaps-log.conf* is as follows, showing custom patterns for each component being sent to the console for display:

```
#
# Log configuration example
#

[[sink]]
name = "console_mt"
type = "stdout_sink_mt"

# Override pattern for WeatherIndicator.sensor
[[pattern]]
name = "sensor_console"
value = "[%l]:%H:%M:%S,%e:[%P]:SENSOR:%v"

[[logger]]
name = "WeatherIndicator.sensor"
sinks = ["console_mt"]
pattern = "sensor_console"

# Override pattern for WeatherReceiver.monitor
[[pattern]]
name = "monitor_console"
value = "[%l]:%H:%M:%S,%e:[%P]:MONITOR:%v"

[[logger]]
name = "WeatherReceiver.monitor"
sinks = ["console_mt"]
pattern = "monitor_console"
```

To configure a file sink, include the log filename and logging level.  If creating a directory for the logs, add the 'create_parent_dir' directive.  

```
[[sink]]
name = "weather_indicator_file_mt"
type = "simple_file_sink_mt"
filename = "log/windicator_file.log"
level = "info"
create_parent_dir = true
```

>Note:  When writing to a log file, output to this file happens in batches so it can take some time for the data to appear in the desired file (on the order of a minute or more).  It is highly recommended to write each component instance to a separate file.  If multiple components are writing to the same file, each batch output will write logs for a single component rotating until all components output a batch set (not interleaved based on time).  During the batch transitions, the component output can become mixed together with another component output.  While all data remains present, batch transition lines become hard to parse for desired information.

Example configuration files for various sink types can be found at under [TOML Configuration Example](https://github.com/guangie88/spdlog_setup#toml-configuration-example).  If a log folder is desired for daily and rotating log files, a simple file sink definition will be needed to create this folder using the 'create_parent_dir' indicator since application folders are dynamically created when the RIAPS application is deployed (not by a system administrator).  The simple file log will be created in the desired log directory, but will not be used if it is not specified by a component logger definition.

The console information output can be viewed as indicated in the [Default Logging](#logging-default) section.  Like the actor-level logging, the log files will be stored in the deployed application location (*/home/riaps/riaps_apps/\<app name>/*).  To access the files, the developer must have root access.

The */home/riaps/riaps_apps/\<app name>* directory is only available when the application is deployed.  Removing an application using the RIAPS controller will delete the log information.  It is anticipated that when an application is deployed in a real system, it could be stopped and restarted or just left continuously running throughout the life of the application on the system.  Therefore, logged information will remain available to system operators.  During application development, the developer will be deploying and removing the application during debugging efforts.  So it is recommended to copy the logs to another location if access is needed at a later time (for debugging).

Each application can specify resource limits on each actor instance, one of which is disk usage (or file space).  This limit is specified in the application model file, see the [Resource Management Specifications section](models.md#rm-model-spec).  All components that create log files will be managed by their respective actor-level file space limitation, where the specified limit applies to the combined file space use of all the components that make up the specific actor instance.


### <a name="logging-framework">RIAPS Framework Logging</a>

By default, minimal logging is configured to indicate the current system status and any errors encountered by the RIAPS framework.  More logging is available for each of the RIAPS framework modules to assist in debugging issues.  The logging configuration file for the RIAPS framework can be found on each of the RIAPS nodes in *usr/local/riaps/etc/riaps-log.conf*  Additional module information can be added by including the desired module (such as 'riaps.deplo.depm') in the comma separated 'keys' list.  The default [framework riaps-log.conf file](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/etc/riaps-log.conf) is available in the RIAPS Github repository.
