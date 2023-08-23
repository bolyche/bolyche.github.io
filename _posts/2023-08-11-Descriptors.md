---
layout: post
date: 2023-08-11
title: "Descriptors"
categories: [ shitpost ]
tags: 
description: "Python descriptors"
featured: false
hidden: false
image: assets/images/getset.png
---

As the documentation tells us, a descriptor is just something that implements the `__get__`, `__set__` or `__delete__` method(s). You've probably used them yourself and not known it - examples include `classmethod`, `staticmethod` and  `property` decorators (though they are likely implemented in C or C++, not in python). You call them inside classes and can use them as decorators in handy ways (see my previous post about properties for example).

You can also implement one yourself. Examples in which they're often then implemented are as validators. These allow you to ensure certain conditions are met for your class attributes (such as a number being above 0 or a string having to be one of a set numbers of options).

The below Validator class is from the [HowTo page for Descriptors]() and has excellent examples. One frequent use I've had for validation has been things like date formats or where variables need to be one of a set of strings.

```python
import datetime
from abc import ABC, abstractmethod

class Validator(ABC):

    def __set_name__(self, owner, name):
        self.private_name = '_' + name

    def __get__(self, obj, objtype=None):
        return getattr(obj, self.private_name)

    def __set__(self, obj, value):
        self.validate(value)
        setattr(obj, self.private_name, value)

    @abstractmethod
    def validate(self, value):
        pass

class DateFormat(Validator):

    def validate(self, value):
        try:
            datetime.datetime.strptime(value, "%Y-%m-%d")
        except ValueError:
            print(f"'{value}' does not match YYYY-MM-DD format")
            raise
        
class Example:
    a = DateFormat()

    def __init__(self, a) -> None:
        self.a = a

ex = Example("2023-111-02")
```

They're also often used for logging e.g.

```python
logging.basicConfig(level=logging.INFO)

class LogThis:
    def __set_name__(self, owner, name):
        print(f"Instance of {self} created by {owner} assigned to {name}")
        self.attrib = name
        self._attrib = "_private_" + name
    
    def __get__(self, origin, owner=None):
        val = getattr(origin, self._attrib)
        logging.info(f"Access of {origin}.{self.attrib}={val}")
        return val

    def __set__(self, origin, val):
        setattr(origin, self._attrib, val)
        logging.info(f"Modification of {origin}.{self.attrib}={val}")


class B:    
    x = LogThis()
    
    def __init__(self, x):
        self.x = x

    def change_x(self, val):
        self.x = val

b = B("test")
b.change_x("another")
b.x
```

Outputting
```zsh
Instance of <__main__.LogThis object at 0x100b036d0> created by <class '__main__.B'> assigned to x
INFO:root:Modification of <__main__.B object at 0x100bab7d0>.x=test
INFO:root:Modification of <__main__.B object at 0x100bab7d0>.x=another
INFO:root:Access of <__main__.B object at 0x100bab7d0>.x=another
```

Though I've found implementing a decorator for this can be maybe nicer (more convenient?)