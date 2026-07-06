---
layout: post
date: 2023-08-04
title: "Minor python optimisations"
categories: [ python, programming ]
tags: 
description: "Optimisations"
featured: false
hidden: false
image: assets/images/factorials.png
---

So let's say you take some code e.g. getting a list of factorised numbers, filtering out some multiples and summing them (NB arbitrary example here + assumes positive-numbers only are input)

```python
def factorial(num: int):
    return 1 if num <2 else num * factorial(num-1)

def factorial_list(num: int, filter_num: int):
    factorial_list = []
    for idx, x in enumerate(range(0, num+1)):
        if idx != 0 and idx % 3 == 0:
            pass
        else:
            f_result = factorial(x)
            factorial_list.append(f_result)
    return factorial_list

def factorial_sum(num: int, filter_num: int):
    factorial_sum = 0
    f_list = factorial_list(num, filter_num)
    for item in f_list:
        factorial_sum += item
    return factorial_sum
```

Let's say we try to find how to improve this code, using some of the previous Profiling tools I mentioned in the last post. Let's look at the timing over 100 iterations for calling factorial_sum(400, 3)

```python
number_of_factorials = 400
filter_number = 3

iters = 100
total_time = timeit.timeit(stmt='factorial_sum(number_of_factorials, filter_number)', number=iters, globals=globals())
print(total_time/iters)

with Profile() as profile:
    print(factorial_sum(number_of_factorials, 3)) 
    (
         Stats(profile)
         .strip_dirs()
         .sort_stats(SortKey.CUMULATIVE)
         .print_stats(300)
    )
```

When we look into this it turns out that the overhead of calling `factorial()` itself is really high. The time per run is `0.0049061274999985475`.

If we use functools and replace `factorial()` it doesn't actually get much better

```python
def factorial(num: int):
    return functools.reduce(lambda x, y: x * y, [n for n in range(num,1,-1)]) if num > 1 else 1
```

If we run this with timeit we'll see it actually got worse! It's now `0.005225289159934619`. 

If we update all these functions to be list comprehensions or otherwise lambda plus functools oneliners, we're still sitting at around `0.005238784170069266`

```python
def factorial(num: int):
    return functools.reduce(lambda x, y: x * y, [n for n in range(num,1,-1)]) if num > 1 else 1

def factorial_list(num: int, filter_num: int):
    return [factorial(x) for idx, x in enumerate(range(num+1)) if (idx == 0 or idx % filter_num != 0)]

def factorial_sum(num: int, filter_num: int):
    return sum(factorial_list(num, filter_num))
```

Indeed you can turn it entirely into a oneliner if you so wish, but boo hoo for readability and the timeit still sits at `0.005193943330014008`

```python
def factorial_sum(num: int, filter_num: int):
    return sum([functools.reduce(lambda x, y: x * y, [n for n in range(x,1,-1)]) if x > 1 else 1 for idx, x in enumerate(range(num+1)) if (idx == 0 or idx % filter_num != 0)])
```

Now if we try a loop, what happens?

```python
def factorial_loop(num: int):
    total = 1
    if num == 0:
        return 1
    for n in range(1, num+1, 1):
        total *= n
    return total

def factorial_list_loop(num: int, filter_num: int):
    return [factorial_loop(x) for idx, x in enumerate(range(0, num+1)) if (idx == 0 or idx % filter_num != 0)]

def factorial_sum_loop(num: int, filter_num: int):
    return sum(factorial_list_loop(num, filter_num))
```

We're at `0.0031829539550017215`.

So, the quoted optimisations aren't really that helpful here (aka reducing loops, using lambdas, list comprehensions etc).

We can also try to improve the algorithm itself e.g. update factorial_list to use the previously generated factorial rather than regenerating it from scratch:

```python
def factorial_list(num: int, filter_num: int):
    factorial_dict = {}
    for x in range(0, num+1):
        if x % filter_num == 0 and x != 0:
            continue
        elif x not in factorial_dict:
            factorial_dict[x] = factorial(x)
        else:
            factorial_dict[x] = x * factorial_dict[x-1]
    return [*factorial_dict.values()]
```

Not much improvement either - timeit here is still `0.0031815293749968988`.

Referring back to my previous post of profiling, some of this becomes clearer when we break down what lines are taking up pretty much all the time in all these examples: It's the multiplication of large numbers, and the cost scales with the numbers of digits. 

![Multiplication](/assets/images/multiplication.png)

The obvious solution you'd think to use is `numpy` or `math` implementations. 

With numpy, we run into how numpy uses fixed width integer types like int64 or float64. It enables faster calculations and general memory efficiency and it's useful to specify the size limit you want to ensure speed - but it doesn't prevent overflow in operations between numpy integers. It catches the error when a number type is declared and passed through initially - but not after declaration. This means that dealing with additions between large factorials is a bad idea as exceeding the limit will wrap around to the most negative number:

```python
import numpy as np

# this errors
print(np.iinfo(np.int32).max)   # 2147483647
print(np.int32(2147483647+1))

# OverflowError: Python integer 2147483648 out of bounds for int32

# this doesn't error, and wraps around instead
print(np.iinfo(np.int64).max)          # 9223372036854775807
print(np.int64(9223372036854775807))   # 9223372036854775807
print(np.int64(9223372036854775807) + np.int64(1))  # -9223372036854775808
```

Math doesn't have this issue because it uses arbitrary precision numbers, trading away the performance for the flexibility and the limit of an int is the limit of your available RAM. A solution with this looks like


```python
def factorial_list_math(num: int, filter_num: int):
    return [
        math.factorial(x)
        for idx, x in enumerate(range(0, num + 1))
        if (idx == 0 or idx % filter_num != 0)
    ]


def factorial_sum_math(num: int, filter_num: int):
    return sum(factorial_list_math(num, filter_num))
```

At which point the comparison output timing is:

```zsh
recursive           : 0.004980s
loop                : 0.003333s
math.factorial      : 0.000560s
```

Math is x6 faster and clearly the winner. Often this kind of thing is the reason to understand performance bottlenecks per line of code rather than overall time/memory allocation - It allows you to dig into the real issue faster.