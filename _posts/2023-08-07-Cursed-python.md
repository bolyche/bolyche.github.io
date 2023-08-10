---
layout: post
date: 2023-08-07
title: "Cursed python"
categories: [ shitpost ]
tags: 
description: "Cursed python"
featured: false
hidden: false
image: assets/images/cursed.jpg
---

There was a really amusing (and informative?) video someone linked a few weeks ago on Reddit for horrifying python to get you fired. I would recommend watching it because it's just great - [link](https://www.youtube.com/watch?app=desktop&v=t863QfAOmlY).

I was thinking about playing around with making some cursed python and here's a snippet of a fun thing you can mess up mentioned in the video.

```
def cursed(a):
    e = 2.71828
    s = "cheese"

    match (a):
        case(result) if a == 10:
            result = result * 10
            print(f"Great news! {result=}")
        case (okay_result) if a == 5:
            try:
                result = okay_result + s / e
                print(f"Not bad! You've got {e/okay_result=}")
            except TypeError as e:
                print(f"Error: {e}")
                pass
        case e:
            print("All other results")
    
    if not result:
        raise ValueError("result failed to be defined")

    print(f"Your results are in! Your bet has won you {result}")
    print(f"The secret number this round was... {e}")
    return result
```

Outputs go as follows

`cursed(10)`:
```zsh
Great news! result=100
Your results are in! Your bet has won you 100
The secret number this round was... 2.71828
```

`cursed(5)`:
```zsh
Error: unsupported operand type(s) for /: 'str' and 'float'
Your results are in! Your bet has won you 5
Traceback (most recent call last):
  File "horrifying.py", line 41, in <module>
    print(cursed(5))
          ^^^^^^^^^
  File "horrifying.py", line 20, in cursed
    print(f"The secret number this round was... {e}")
                                                 ^
UnboundLocalError: cannot access local variable 'e' where it is not associated with a value
``` 
 
`cursed(1)`:
```zsh
All other results
Your results are in! Your bet has won you 1
The secret number this round was... 1
```

My favourite here is the last example. The case statement assigns the variable name (redefining e) and even though the result was never assigned, it was assigned in different case statement and so the function doesn't exit!
