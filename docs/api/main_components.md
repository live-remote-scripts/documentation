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

## `ElementsBase`

With the `ableton.v3.control_surface.ElementsBase` class you define the interface to your
hardware controls and make them accessible for your Remote Script. Therefore you need to
inherit from the `ElementsBase` and add the control elements you want within its `__init__` function.
You do this by calling the functions available on the `ElementsBase` instance.

### Functions

#### `add_encoder_element`

Add an encoder element for the midi channel `channel` and the message `identifier`.

This function will add a new `EncoderElement` property to the `Elements` class with the name matching
the value for `name` in _lower_snake_case_.

| arguments                                                                        | description                                                                                         |
|----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| `identifier: int`                                                                | The midi message data value to listen to                                                            |
| `name: str`                                                                      | The name of the control that is used to reference it within your project                            |
| `channel: int` (default: `0`)                                                    | The midi message channel to listen to                                                               |
| `msg_type: int` <br/>(default: `ableton.v2.control_surface.MIDI_CC_TYPE`)        | The midi message type of the hardware control. [See all available types](#Valid-midi-message-types) |
| `map_mode: Live.MidiMap.MapMode` <br/>(default: `Live.MidiMap.MapMode.absolute`) | TODO which values are available?                                                                    |
| `mapping_sensitivity: float` (default: `1.0`)                                    | TODO                                                                                                |
| `needs_takeover: bool` (default: `True`)                                         | TODO                                                                                                |
| `is_feedback_enabled: bool` (default: `False`)                                   | TODO                                                                                                |
| `feedback_delay: int` (default: `0`)                                             | TODO                                                                                                |

#### `add_button`

Add a button element for the midi channel `channel` and the message `identifier`.

This function will add a new `ButtonControl` property to the `Elements` class with the name matching
the value for `name` in _lower_snake_case_.

| arguments                                                                 | description                                                                                         |
|---------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| `identifier: int`                                                         | The midi message data value to listen to                                                            |
| `name: str`                                                               | The name of the control that is used to reference it within your project                            |
| `channel: int` (default: `0`)                                             | The midi message channel to listen to                                                               |
| `msg_type: int` <br/>(default: `ableton.v2.control_surface.MIDI_CC_TYPE`) | The midi message type of the hardware control. [See all available types](#Valid-midi-message-types) |
| `is_momentary: bool` <br/>(default: `True`)                               | TODO                                                                                                |
| `led_channel: int` (default: `None`)                                      | TODO                                                                                                |

#### `add_modifier_button`

Add a button that is not used directly as a button but as a modifier for another control element.
This could be a shift button or something similar. Use this in combination with `add_modified_control`.

This function will add a new `ButtonControl` property to the `Elements` class with the name matching
the value for `name` in _lower_snake_case_.

| arguments                                                                 | description                                                                                         |
|---------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| `identifier: int`                                                         | The midi message data value to listen to                                                            |
| `name: str`                                                               | The name of the control that is used to reference it within your project                            |
| `channel: int` (default: `0`)                                             | The midi message channel to listen to                                                               |
| `msg_type: int` <br/>(default: `ableton.v2.control_surface.MIDI_CC_TYPE`) | The midi message type of the hardware control. [See all available types](#Valid-midi-message-types) |
| `is_momentary: bool` <br/>(default: `True`)                               | TODO                                                                                                |
| `led_channel: int` (default: `None`)                                      | TODO                                                                                                |

#### `add_modified_control`

This function will create a new control element with an alternative functionality to an existing one.
The new control element will receive and send signals, whenever its modifier is active. Use in combination with
`add_modifier_button`.

This function will add a new `ControlElement` property to the `Elements` class with the name matching
the value for `name` in _lower_snake_case_.

| arguments                            | description                                                              |
|--------------------------------------|--------------------------------------------------------------------------|
| `control: ControlElement`            | The control element for which to create an alternative functionality     |
| `modifier: ControlElement`           | The modifier which enables the alternative functionality when active     |
| `name: str`                          | The name of the control that is used to reference it within your project |
| `is_private: bool` (default: `True`) | TODO                                                                     |

#### `add_encoder_matrix`

TODO

#### `add_button_matrix`

TODO

### Valid midi message types

The API already defines message types as constants that can be reused to configure
your Remote Script.

```python
from ableton.v2.control_surface import (
    MIDI_NOTE_TYPE,
    MIDI_CC_TYPE,
    MIDI_PB_TYPE,
    MIDI_SYSEX_TYPE,
    MIDI_INVALID_TYPE
)
```

For our example we just want to access the first Encoder and the first button row of the **BCR2000**.

```python
# elements.py

from ableton.v3.control_surface import ElementsBase


class BCR2000Elements(ElementsBase):

    def __init__(self, *a, **k):
        super().__init__(*a, **k)
        # TODO add the elements

```

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


