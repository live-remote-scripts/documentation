# Remote Script API Main Components

## `__init__`

If you are new to python, you might have recognized, that within each directory
in a python project, there is a `__init__.py` file. This file initializes
a [regular python package](https://docs.python.org/3/reference/import.html#regular-packages)
and everything within this file is executed when the package is imported.

The entrypoint for **every** Custom Remote Script is the `create_instance` function
within the `__init__` file. The function needs to return your custom implementation of
the `ControlSurface`. The function has one call argument `c_instance` which I guess is
an instance of some Custom Remote Script abstraction from the Live core which is written
in C++.

Live will (most probably) call this function whenever you select the remote
script in the preferences or on application start if it has been selected previously.

```python
# __init__.py

from ableton.v3.control_surface import ControlSurface
from .bcr_2000 import CustomBCR2000


def create_instance(c_instance) -> ControlSurface:
    return CustomBCR2000(c_instance=c_instance)
```

Whenever the import of this file, or the call of the `create_instance` function
errors, your remote script **will not** be available in Live.

## `Elements`

## `ControlSurfaceSpecification`

The Specification is the base configuration of your Remote Control Script. To create
your own `Specification` you inherit from the `ableton.v3.control_surface.ControlSurfaceSpecification`
class and set the properties you want to configure.

### Properties

| property                   | python type                                  | description                                                                                                                                                                                                                                                                                                     |
|----------------------------|----------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `elements_type`            | `Type[ElementsBase]`                         | Subclass of `ableton.v3.control_surface.ElementsBase` for the definition of your hardware controls such as encoders, buttons etc.                                                                                                                                                                               |
| `create_mappings_function` | `Callable[[ControlSurface], Dict[str, any]]` | A function that maps the names of the hardware controls defined in the `elements_type` to the software controls of your components.                                                                                                                                                                             |
| `component_map`            | `Dict[str, Type[Component]]`                 | A dictionary that maps component names to `ableton.v3.control_surface.Component` types. The defined components within this map will be loaded automatically and do not to be instantiated by your own. You can access them within your `ControlSurface` class via `self.component_map["name-of-the-component"]` |  

```python
# speficication.py

from ableton.v3.control_surface import ControlSurfaceSpecification, ControlSurface, DeviceComponent
from .elements import BCR2000Elements


def create_mappings(_: ControlSurface) -> Dict[str, any]:
    return {
        "Device": {
            "parameter_controls": "device_controls"
        }
    }


class BCR2000Specification(ControlSurfaceSpecification):
    elements_type = BCR2000Elements
    create_mappings_function = create_mappings
    component_map = {
        "Device": DeviceComponent
    }
```

## `ControlSurface`

The Control Surface represents your controller. Each control surface is instantiated once
it is selected in Live. You need to inherit from this class.

```python
# bcr_2000.py

from ableton.v3.control_surface import ControlSurface


class CustomBCR2000(ControlSurface):
# TODO Override init and provide specification
# log a message
# setup function

```

## `Component`


