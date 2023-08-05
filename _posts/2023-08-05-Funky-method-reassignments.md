---
layout: post
date: 2023-08-05
title: "Funky method reassignments"
categories: [ shitpost ]
tags: 
description: "Python method reassignment"
featured: true
hidden: true
image: assets/images/funkyballs.jpg
---

I saw this kind of example recently and thought it was funky. I wrote out an example

```python
import profile

class ReassignMethods:
  def useful_function(self,a):
    print("case 1")
    self._answer = a*10
    self.useful_function = self.handle_second_case
    return self._answer

  def handle_second_case(self,b):
    print("case 2")
    self.useful_function =  self.handle_third_case
    return b*200
  
  def handle_third_case(self,c):
    print("case 3")
    return c*3000

def example():
  r = ReassignMethods()
  for x in range(0,3):
    print(f"Answer for x = {x}:",r.useful_function(x))

profile.run("example()")
```

If you run this, the output will be something like 

```zsh
case 1
Answer for x = 0: 0
case 2
Answer for x = 1: 200
case 3
Answer for x = 2: 6000
         14 function calls in 0.006 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.000    0.000 :0(exec)
        6    0.000    0.000    0.000    0.000 :0(print)
        1    0.006    0.006    0.006    0.006 :0(setprofile)
        1    0.000    0.000    0.000    0.000 <string>:1(<module>)
        1    0.000    0.000    0.006    0.006 profile:0(example())
        0    0.000             0.000          profile:0(profiler)
        1    0.000    0.000    0.000    0.000 test.py:26(useful_function)
        1    0.000    0.000    0.000    0.000 test.py:32(handle_second_case)
        1    0.000    0.000    0.000    0.000 test.py:37(handle_third_case)
        1    0.000    0.000    0.000    0.000 test.py:41(example)
```

In all honestly I'm not sure what the use case is for this. 

It's useful if you want a method to do something specific the first n times then have a default later. If you're wanting to have the same method called with the same class instance returning differently for various calls, it's not clear if it's more efficient. 

If I write another implementation doing the same thing I agree it does look ugly! 

```python
class NonReassigMethods:
  def __init__(self):
    self._useful_func_counter = 0

  def useful_function(self,a):
    if self._useful_func_counter == 0:
      print("case 1")
      self._useful_func_counter += 1
      return a*10
    elif self._useful_func_counter == 1:
      print("case 2")
      self._useful_func_counter += 1
      return a*200
    else:
      print("case 3")
      return a*3000

def example2():
  r = NonReassigMethods()
  for x in range(0,3):
    print(f"Answer for x = {x}:",r.useful_function(x))

profile.run("example2()")
```

On a timeing scale with profile there's no difference. When we use timeit and get to a million iterations there's a small efficiency gain but we're talking `3.3525129199551884e-07` vs `2.9735416699259077e-07`.

All in all, funky but I feel like the code needs better design than either of these examples. But then, absolutely depends on what you're trying to achieve.