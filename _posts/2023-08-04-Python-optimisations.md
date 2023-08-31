---
layout: post
date: 2023-08-04
title: "Minor python optimisations"
categories: [ shitpost, programming ]
tags: 
description: "Optimisations"
featured: false
hidden: false
image: assets/images/factorials.png
---

In the desire to get myself to write more frequently I'm tagging upcoming posts as "shitpost". This will be short thoughts or snippets that don't take much time.

For the first of these, some minor python optimisations.

Take some code e.g. getting a list of factorised numbers, filtering out some multiples and summing them 
(note this is arbitrary and just for exploration purposes. I am also completely ignoring negative numbers)

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

& say we look at the timing over 100 iterations for calling factorial_sum(400, 3)

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

If we run this with timeit we'll see it actually got worse! It's now `0.005225289159934619`. I admit I found this surprising and due to being a shitpost am not digging into it overly. 
Something I thought was interesting was that if indeed we actually update all these functions to be list comprehensions or otherwise lambda plus functools oneliners, we're still sitting at around `0.005238784170069266`

```python
def factorial(num: int):
    return functools.reduce(lambda x, y: x * y, [n for n in range(num,1,-1)]) if num > 1 else 1

def factorial_list(num: int, filter_num: int):
    return [factorial(x) for idx, x in enumerate(range(num+1)) if (idx == 0 or idx % filter_num != 0)]

def factorial_sum(num: int, filter_num: int):
    return functools.reduce(lambda x, y: x+y, factorial_list(num, filter_num))
```

Indeed you can turn it entirely into a oneliner if you so wish, but boo hoo for readability and the timeit still sits at `0.005193943330014008`

```python
def factorial_sum(num: int, filter_num: int):
    return functools.reduce(lambda x, y: x+y, [functools.reduce(lambda x, y: x * y, [n for n in range(x,1,-1)]) if x > 1 else 1 for idx, x in enumerate(range(num+1)) if (idx == 0 or idx % filter_num != 0)])
```

Now if we try a loop, what happens?

```python
def factorial(num: int):
    total = 1
    if num == 0:
        return 1
    for n in range(1, num+1, 1):
        total *= n
    return total

def factorial_list(num: int, filter_num: int):
    return [factorial(x) for idx, x in enumerate(range(0, num+1)) if (idx == 0 or idx % filter_num != 0)]

def factorial_sum(num: int, filter_num: int):
    return functools.reduce(lambda x, y: x+y, factorial_list(num, filter_num))
```

We're at `0.0031829539550017215`. How intruiging. We've finally shaved off some time. 

I have at random picked perhaps an interesting example, since the oft quoted optimisations aren't really that helpful here (aka reducing loops, using lambdas, list comprehensions, optimised libraries etc).

The obvious solution should be to use `numpy` or `math` factorial implementations but we can also try to improve the algorithm itself e.g. update factorial_list to use the previously generated factorial rather than regenerating it from scratch:

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

Alas the timeit here is still `0.0031815293749968988`!

Referring back to my previous post of profiling, some of this becomes clearer when we break down what lines are taking up pretty much all the time in all these examples. Indeed, the multiplication/addition of absolutely enormous numbers is not entirely trivial at all.

![Multiplication](/assets/images/multiplication.png)

A small note just as I've finished writing this post: I noticed the enumerate doesn't add anything. This is why code review exists folks. That and re-reading ones work :)