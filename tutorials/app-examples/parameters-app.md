# Parameterizing Actors and Components
RIAPS supports parameterizing software components through the component constructor. These parameters can be set and overridden hierarchically, from the component definition (lowest level) to a deployed actor instance (highest level).

## Component Parameters
Component code is made modular with parameters: a sine-wave generator should be re-usable without having to edit the code to modify the generated frequency or amplitude. The following component snippet shows the syntax for accepting component parameters `frequency` and `amplitude`:
```python
...
class Generator(Component):
    def __init__(self, frequency, amplitude):
        super().__init__(self)
        self.frequency = frequency
        self.amplitude = amplitude
```
> Note that `self` is required, and is always the first argument, as part of Python's inheritance patterns.

Doing this changes the component definition in the `.riaps` model file:
```
component Generator(frequency, amplitude) {
    // port declarations, etc
}
```
You can optionally provide defaults values, like below:
```
component Generator(frequency = 60.0, amplitude = 170) { ... }
```
These values can be overwritten at the `actor` level when the `Generator` component is used. Below, the `Generator` instance `LowFrequencyGen.generator` will have a `frequency` of 60.0 and an `amplitude` of 170. The parameters for `HighFrequencyGen.generator` are overridden with a `frequency` of 400.0 and an `amplitude` of 162.
```
actor LowFrequencyGen {
    ...
    {
        generator : Generator;
    }
}

actor HighFrequencyGen {
    ...
    {
        generator : Generator(frequency = 400.0, amplitude = 162)
    }
}
```

## Actor Parameters
Just as components can be written for reuse, actors can be reused by deploying them multiple times in a single app. Consider the updated definition of actor `LowFrequencyGen` below:
```
actor LowFrequencyGen(frequency, amplitude) {
    ...
    {
        generator : Generator(frequency = frequency, amplitude = amplitude);
    }
}
```
Now, when the `LowFrequencyGen` actor is use in a `.depl` deployment file, it expects two parameters: `frequency` and `amplitude`. These input parameters will then be used when creating an instance of the component `Generator`.

Like with components, actors can optionally define default values for their parameters:
```
actor LowFrequencyGen(frequency = 60, amplitude = 277) { ... }
```
Finally, these parameters can be set in the `.depl` deployment file:
```
app ParametersExample {
    on (riaps-abcd.local) LowFrequencyGen(frequency = 50, amplitude = 208);
    on (riaps-1234.local) LowFrequencyGen;
}
```
