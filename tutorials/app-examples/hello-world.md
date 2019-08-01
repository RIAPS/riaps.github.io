## "Hello World" from RIAPS Tutorials Episode 1

 The RIAPS "Hello World" example features a single actor, component, and timer port running on a single RIAPS node. Tutorial 1 introduces distributed applications and the actor model of distributed programming, then walks through building the "Hello World" of Smart Grid applications. This application (called "SmartGrid") uses a timer port to trigger a component, which then generates a measurement from an imagined Phase Measurement Unit (PMU).

 [RIAPS Tutorials Episode 1](https://youtu.be/18AmX5FRCXo "RIAPS Tutorials Episode 1")

Learn more about RIAPS by reusing this application in [Episode 2](https://riaps.github.io/tutorials/app-examples/pub-sub.html "Link to Episode 2")

### Model File (SmartGrid.riaps)
```
app SmartGrid {

  component PMUSampler {
    timer clock 1000;
  }

  actor PMU {
    {
      mySampler : PMUSampler;
    }
  }
}
```

### Deployment File (SmartGrid.depl)
```
app SmartGrid {
  on (<RIAPS NODE IP ADDRESS>) PMU;
}
```

### Component Code (PMUSampler.py)
```python
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
# riaps:keep_clock:end

# riaps:keep_impl:begin
    def takeSample(self,t):
        voltage = 480*sin(2*pi*60*t)
        current = 200*sin(2*pi*60*t)
        return (voltage,current)

# riaps:keep_impl:end
```
