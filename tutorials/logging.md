# Component Logging Tutorial

The application developer can customize the logging setup for each application component by defining a **riaps-log.conf** file and placing it in the application directory with the components.  There is also an option to turn on global file logging at the actor level and additional RIAPS framework messages to assist in debugging.

> Note:  Since applications are deployed under a generated username, the developer must have root access to view the log messages in the /home/riaps/riaps_apps/<application name> directory.

## Component Logging Format Configuration

This file defines one or more sink types, loggers that utilize one of the sinks, and the output pattern of the log information. Console and file sink types available are:

- ConsoleLogger
  - stdout_sink ```(_mt or _st)```
  - stdout_color_sink ```(_mt or _st)```
  - stderr_sink ```(_mt or _st)```
  - stderr_color_sink ```(_mt or _st)```
- FileLogger
  - simple_file_sink ```(_mt or _st)```
  - rotating_file_sink ```(_mt or _st)```
  - daily_file_sink_st ```(_mt or _st)```

> Note: Utilize either single threaded (```_st```) or thread safe multi-threaded (```_mt```) loggers.  For a full list of loggers, see [Supported Sinks](https://github.com/guangie88/spdlog_setup#supported-sinks)

A logger section should be available for each component with the name indicating the actor name and the component instance.  This allows flexibility for the developer to direct and format the output of each component instance individually.

The pattern section defines the expected format of the logging message, any identifying text, and the message logged by the component in the code.  The pattern flag options are listed in a [spdlog custom formatting tutorial](https://github.com/gabime/spdlog/wiki/3.-Custom-formatting).

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

To configure a file sink include the log filename.  If creating a directory for the logs, add the **create_parent_dir** directive.

```
[[sink]]
name = "weather_indicator_file_mt"
type = "simple_file_sink_mt"
filename = "log/windicator_file.log"
level = "info"
create_parent_dir = true
```

>Note:  When writing to a log file, output to this file happens in batches so can take some time for the data to appear in the desired file (on the order of a minute or more).  If multiple components are writing to the same file, each batch output will write logs for a single component rotating until all components output a batch set (not interleaved based on time).

Example configuration files for various sink types can be found at under [TOML Configuration Example](https://github.com/guangie88/spdlog_setup).  If a log folder is desired for daily and rotating log files, then a simple file sink will be needed that creates the desired directory using the **create_parent_dir** indicator.  The simple file log will be created, but will not be used if it is not specified by a component logger definition.

## Example Python Code for Component Logging

Use of logging within a component is simple.  Utilize the RIAPS component self.logger.<log level> function to provide user information.

```
self.logger.info("on_tempupdate(): Temperature:%s, PID %s, Timestamp:%s" % (temperatureValue, str(now), temperatureTime))
```

Available Log Levels are:
- trace
- debug
- info
- warn
- error
- critical

## Log Application Actor Information to a File

A global way to log Actor information to a file in the deployed application directory is to turn on the RIAPS Framework application log option in the [RIAPS Framework Configuration file](https://github.com/RIAPS/riaps-pycom/tree/develop/src/riaps/etc/README.md).  This file is located on each RIAPS node in **/usr/local/riaps/etc/riaps.conf**.

```
# application logs
# ''        -- stdout
# log       -- app/actor.log file
app_logs = log
```

## RIAPS Framework Logging

By default, minimal logging is configured to indicate the current system status and any errors encountered is the default configuration of the framework logging.  More logging is available for each of the RIAPS framework modules to assist in debugging issues.  The logging configuration can be found on each of the RIAPS nodes in **/usr/local/riaps/etc/riaps-log.conf**.  Additional module information can be added by including the desired module (such as riaps.deplo.depm) in the comma separated **keys** list.
