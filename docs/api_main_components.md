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

Live will (most probably) call this function upon application start.

```python
def create_instance(c_instance) -> ControlSurface:
    return MyControlSurface(c_instance=c_instance)
```

Whenever the import of this file, or the call of the `create_instance` function
errors, your remote script **will not** be available in Live.

## `ControlSurface`

The Control Surface...

## `Component`

## `Elements`

## `Specification`


