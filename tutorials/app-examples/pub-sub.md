## "Publish/Subscribe" from RIAPS Tutorials Episode 2

The RIAPS Pub/Sub example expands upon the application from [Episode 1](https://riaps.github.io/tutorials/app-examples/hello-world.html "Link to Episode 1") to add a RIAPS node logging data from the PMU. Tutorial 2 introduces messages, message topics, and the publish/subscribe messaging pattern. It then adds these features to the application from Episode 1, which then samples (rather, simulates sampling) a PMU and publishes the measurement on a RIAPS message topic, which is received by a subscriber, then logged. 

### [RIAPS Tutorials Episode 2](https://www.youtube.com/watch?v=4bE0cddRiSE "RIAPS Tutorials Episode 2")



### Model File (SmartGrid.riaps)
```
app SmartGrid {
	
  message PmuData;

  component PMUSampler {
    timer clock 1000;
    pub pubport : PmuData;
  }
  
  component PhaseDataListener {
    sub subport : PmuData;
  }

  actor PMU {
    {
      mySampler : PMUSampler;
    }
  }
  
  actor DataLogger {
    {
      myListener : PhaseDataListener;
    }
  }
}
```

### Deployment File (SmartGrid.depl)
```
app SmartGrid {
  on (<RIAPS NODE IP ADDRESS>) PMU;
  on (<RIAPS NODE IP ADDRESS>) DataLogger;
}
```

### Component Code (PMUSampler.py)
```
# riaps:keep_import:begin
from riaps.run.comp import Component
import spdlog
import capnp
import smartgrid_capnp
from math import sin, pi

# riaps:keep_import:end

class PMUSampler(Component):

# riaps:keep_constr:begin
    def __init__(self):
        super(PMUSampler, self).__init__()
# riaps:keep_constr:end

# riaps:keep_clock:begin
    def on_clock(self):
        timestamp = self.clock.recv_pyobj()
        measurement = self.takeSample(timestamp)
        self.logger.info("Measured %f volts and %f amps" % measurement)
        self.pubport.send_pyobj(measurement)
# riaps:keep_clock:end

# riaps:keep_impl:begin
    def takeSample(self,t):
        voltage = 480*sin(2*pi*60*t)
        current = 200*sin(2*pi*60*t)
        return (voltage,current)

# riaps:keep_impl:end
```

### Component Code (PhaseDataListener.py)
```
# riaps:keep_import:begin
from riaps.run.comp import Component
import spdlog
import capnp
import smartgrid_capnp

# riaps:keep_import:end

class PhaseDataListener(Component):

# riaps:keep_constr:begin
    def __init__(self):
        super(PhaseDataListener, self).__init__()
# riaps:keep_constr:end

# riaps:keep_subport:begin
    def on_subport(self):
        measurement = self.subport.recv_pyobj()
        self.logger.info("Logging %s" % str(measurement))
# riaps:keep_subport:end

# riaps:keep_impl:begin

# riaps:keep_impl:end
```
