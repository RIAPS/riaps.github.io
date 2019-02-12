# Component Logging Tutorial

The RIAPS framework utilizes spdlog (a fast C++ logging library) to provide a logging mechanism for both the RIAPS framework and the application components.  There are 3 ways to log information within a component:  default, actor-level, and customizable component-level logging.  

## Default Logging

Use of logging within a component is simple.  Utilize the RIAPS component ```self.logger.<log level>``` function to provide user information.  Here is an example:

```
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

```
import spdlog as spd

logger.set_level(spd.LogLevel.INFO)
```

The log messages are sent to the console output on the node.  If working on the development VM and starting the **riaps_deplo** using either Eclipse or ```sudo -E riaps_deplo``` in a terminal window, the console output will be available.  The information provided will be both the RIAPS platform and the application component logging information.

The BBB nodes run **riaps_deplo** as a systemd service, so the console log information is placed in the system logs.  To view the information as the application is running, ssh into a BBB node and run the following command:

```
sudo journalctl -u riaps-deplo.service -f
```


## Actor-Level Logging

A global way to log **Actor** information to a file in the deployed application directory is to turn on the RIAPS Framework application log option in the [RIAPS Framework Configuration file](https://github.com/RIAPS/riaps-pycom/tree/develop/src/riaps/etc/README.md).  This is done by adding the 'log' designations to the 'app_logs =' configuration line, as shown below.  This file is located on each RIAPS node in **/usr/local/riaps/etc/riaps.conf**.  Each actor defined in the application model file will have a log file in ```/home/riaps/riaps_apps/<app name>/```.  

```
# application logs
# ''        -- stdout
# log       -- app/actor.log file
app_logs = log
```

For the remote RIAPS nodes, the riaps.conf can be modified on the development VM and then moved to the remote nodes using the [fabfile utility](https://github.com/RIAPS/riaps-pycom/tree/master/bin). Use the ```fab riaps.updateConfig``` command to setup the remote nodes.

> MM TODO:  test this command to make sure it is correct or if something else is needed

> Note:  Since applications are deployed under a generated username, the developer must have root access to view the log messages in the ```/home/riaps/riaps_apps/<app name>``` directory.


## Customizable Component-Level Logging

The application developer can customize the logging setup for each application component instance by defining a **riaps-log.conf** file and placing it in the application directory with the components.  This file defines the logging configuration and output pattern for each of the application components instances, thus providing flexibility for the developer to direct and format the output of each component instance individually.  

The logging configuration (or ```[[logger]]``` section) indicates the component instance for the specific configuration definition, how the output will be handles, and the output format used.  When the actors were defined in the application model file (.riaps), a name was given for each instance of the application component used in the actor.  Using the [WeatherMonitor example application](https://github.com/RIAPS/riaps-apps/blob/master/apps-vu/WeatherMonitor/Python/wmonitor.riaps), an actor definition is shown below.  The **name** for this component instance will be  **WeatherIndicator.sensor**, where 'WeatherIndicator' is the actor's name and 'sensor' is the component instance name.

```
    actor WeatherIndicator {
       {  // TempSensor publishes 'TempData' messages
          sensor : TempSensor;				
       }
    }
```

How the output will be handled is defined by the chosen **sink** method.  

> MM TODO:  
- need to define 'sink' in user terms in the above paragraph.  
- Include the concept of a sink and a logger.  
- What are the sink types, what they do

In spdlog terms, a sink is an interface between a logger instance and its desired target. Each logger is associated with one or more sink objects that actually write the log contents to a specific target such as a file, console or database. Each sink also contains its own private instance of a formatter object, which is responsible for the visual representation of the logged data. It can be customized with user-defined patterns that specify the log format. Sinks can also specify colored output.

Console and file sink types available are:

- ConsoleLogger
  - stdout_sink ```(_mt or _st)``` - standard output.
  - stdout_color_sink ```(_mt or _st)``` - standard output colored.
  - stderr_sink ```(_mt or _st)``` - standard error.
  - stderr_color_sink ```(_mt or _st)``` - standard error colored.
- FileLogger
  - simple_file_sink ```(_mt or _st)``` - Basic file sink that writes to a given log file.
  - rotating_file_sink ```(_mt or _st)``` - Rotating log files. When the maximum size is reached, a new file is created and the logger switches to a new file. The maximum size of each file and the maximum number of files can be configured.
  - daily_file_sink_st ```(_mt or _st)``` - Creates a new file every day at a specified time instance.
  
  Apart from these, users can also define their own sinks manually (example shown below).

> Note: Utilize either single threaded (```_st```) or thread safe multi-threaded (```_mt```) loggers.  For a full list of loggers, see [Supported Sinks](https://github.com/guangie88/spdlog_setup#supported-sinks)


The **pattern** section defines the expected format of the logging message, any identifying text, and the message logged by the component in the code.  The pattern flag options are listed in a [spdlog custom formatting tutorial](https://github.com/gabime/spdlog/wiki/3.-Custom-formatting).

Using the example [WeatherMonitor application](https://github.com/RIAPS/riaps-apps/tree/master/apps-vu/WeatherMonitor/Python), an example  **riaps-log.conf** is as follows, showing custom patterns for each component being sent to the console for display:

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

To configure a file sink, include the log filename and logging level.  If creating a directory for the logs, add the **create_parent_dir** directive.  

```
[[sink]]
name = "weather_indicator_file_mt"
type = "simple_file_sink_mt"
filename = "log/windicator_file.log"
level = "info"
create_parent_dir = true
```

>Note:  When writing to a log file, output to this file happens in batches so it can take some time for the data to appear in the desired file (on the order of a minute or more).  It is highly recommended to write each component instance to a separate file.  If multiple components are writing to the same file, each batch output will write logs for a single component rotating until all components output a batch set (not interleaved based on time).  During the batch transitions, the component output can become mixed together with another component output.  While all data remains present, batch transition lines become hard to parse for desired information.

Example configuration files for various sink types can be found at under [TOML Configuration Example](https://github.com/guangie88/spdlog_setup).  If a log folder is desired for daily and rotating log files, a simple file sink definition will be needed to create this folder using the **create_parent_dir** indicator since application folders are dynamically created when the RIAPS application is deployed (not by a system administrator).  The simple file log will be created in the desired log directory, but will not be used if it is not specified by a component logger definition.


## RIAPS Framework Logging

By default, minimal logging is configured to indicate the current system status and any errors encountered by the RIAPS framework.  More logging is available for each of the RIAPS framework modules to assist in debugging issues.  The logging configuration file for the RIAPS framework can be found on each of the RIAPS nodes in **/usr/local/riaps/etc/riaps-log.conf**.  Additional module information can be added by including the desired module (such as riaps.deplo.depm) in the comma separated **keys** list.  The default [framework riaps-log.conf file](https://github.com/RIAPS/riaps-pycom/blob/master/src/riaps/etc/riaps-log.conf) is available in the RIAPS Github repository.
