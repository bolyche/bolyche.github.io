---
layout: post
date: 2023-08-10
title: "Properties"
categories: [ shitpost, programming ]
tags: 
description: "Python properties"
featured: false
hidden: false
image: assets/images/properties.jpg
---

There are various cool python builtins and `property` is one of them. It's pretty much an easy way to implement a descriptor using method decoration. In python we don't actually have private attributes and so custom getters/setters/deleters are a nice way to allow attribute access since you can use them in the same way. For example

```python
class A:

    def __init__(self) -> None:
        self._record = None

    @property
    def record(self):
        return self._record
    
    @record.setter
    def record(self, val):
        self._record = val

    @record.deleter
    def record(self):
        del self._record

a = A()
a.record = "Annie"
print(a.record)
```

Which outputs 

```zsh
Annie
```

Now this in itself isn't that interesting but I did think of an amusing way to make it weird. 

If you change `_record` to be the class-private `__record` - As expected you get the name mangling and looking at `dir(a)` you'll see an output of `_A__record`. It's not actually private as mentioned above but it's meant to deter accidents during class inheritance or other shenanigans.

```zsh
['_A__record', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'record']
```

What I found amusing was that running this can suddenly feel really misleading!

```python
class A:

    def __init__(self) -> None:
        self.__record = None

    @property
    def record(self):
        return self.__record
    
    @record.setter
    def record(self, val):
        self.__record = val

    @record.deleter
    def record(self):
        del self.__record
    
    def mult(self):
        return self.fake * 6


a = A()

a.record = "Sally Sanderson"
print(a.record)         # Sally
print(a.__record)       # AttributeError: 'A' object has no attribute '__record'. Did you mean: '_A__record'?
print(a._A__record)     # Sally

a.__record = "Tom Waits"
print(a.record)         # Sally
print(a.__record)       # Tom Waits
print(a._A__record)     # Sally
```

I've put the outputs as comments alongside each print.

It's misleading on first glance. But of course it's python and you can call, implement or add all sorts of funky things in your code. The memory address for `record` (and correspoding `_A_record`) is different to the new thing we added of `a.__record`. Just in the same way I can add this method to class A without having the instance attribute actually existing

```python
    def mult(self):
        return self.fake * 6
```

and create all this 

```
a.__fake = "Jennifer's"
a._fake = "Novel"
a.fake = "Approach"

print(a.mult())
```

And it'll all run and print out `ApproachApproachApproachApproachApproachApproach`.