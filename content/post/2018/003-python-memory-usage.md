---
title: "Python Memory Usage"
date: 2018-06-03
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: ["python"]
categories: []
---

The Python language has a feature called `Generator Expressions` which were 
introduced with PEP 289. You can think of them as a better way of doing certain
operations involving lists. This post is more interested in the memory benefits
the feature provides. We will first introduce the `memory_profiler` tool which
can be used to measure the memory usage of a python program. We will then 
compare two different pieces of code (one with and the other without generator 
expressions) which perform the same operation, explaining why one is more 
superior than the other. Finally, we will run a few experiments to demonstrate 
and prove our assertion. 

<!--more-->

When we talk about measuring the memory usage of python code, we are usually 
interested in determining memory expense on a line-by-line basis. The 
``memory_profiler`` tool ( ref: https://github.com/pythonprofilers/memory_profiler )
is ideal for this purpose. Here is example code of how it can be used,

```python
from memory_profiler import profile

@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

Executing the above will return the following output,

```text
Line #    Mem usage    Increment   Line Contents
================================================
    22     34.1 MiB     34.1 MiB   @profile
    23                             def my_func():
    24     41.8 MiB      7.7 MiB       a = [1] * (10 ** 6)
    25    194.2 MiB    152.4 MiB       b = [2] * (2 * 10 ** 7)
    26     41.8 MiB   -152.3 MiB       del b
    27     41.8 MiB      0.0 MiB       return a
```

The above clearly shows that one assignment statement is more expensive than the
other as well as indicating that the delete operation actually recovers some
memory back. 

What exactly is a Python generator expression? According to PEP 289, they are a
high performance, memory efficient generalization of list comprehensions. That 
definition is a bit much for my tastes, so let's just jump into an example;

```python
from memory_profiler import profile

@profile(precision=4)
def my_test_function():
    THE_LIMIT = 10000
    PP = sum(x * x for x in range(THE_LIMIT))
    a = 1
    NN = sum([x * x for x in range(THE_LIMIT)])
    b = 1

if __name__ == '__main__':
    my_test_function()
```

Pay attention to the assingments to ``PP`` and ``NN``. We calculate the sum of 
squares of all numbers upto a limit for both of them, but the implementation 
is a bit different. In the latter case, a temporary list is created which holds
all the squares we need. The sum is calculated over this list. But with the 
former situation, no such temporary list is created. The sum gets incremented
during each iteration of the for-loop. It feels very intuitive that one method
will use more memory than the other. Executing the code confirms our 
hypothesis,

```text
Line #    Mem usage    Increment   Line Contents
================================================
    13  34.2031 MiB  34.2031 MiB   @profile(precision=4)
    14                             def my_test_function():
    15  34.2031 MiB   0.0000 MiB       THE_LIMIT = 10000
    16  34.2031 MiB   0.0000 MiB       PP = sum(x * x for x in range(THE_LIMIT))
    17  34.2031 MiB   0.0000 MiB       a = 1
    18  34.5898 MiB   0.3867 MiB       NN = sum([x * x for x in range(THE_LIMIT)])
    19  34.5898 MiB   0.0000 MiB       b = 1
```


A most interesting things happens however, if were were to increase our limit
by a factor of 10;

```text
Line #    Mem usage    Increment   Line Contents
================================================
    13  33.8555 MiB  33.8555 MiB   @profile(precision=4)
    14                             def my_test_function():
    15  33.8555 MiB   0.0000 MiB       THE_LIMIT = 100000
    16  33.8555 MiB   0.0000 MiB       PP = sum(x * x for x in range(THE_LIMIT))
    17  33.8555 MiB   0.0000 MiB       a = 1
    18  37.7578 MiB   0.4219 MiB       NN = sum([x * x for x in range(THE_LIMIT)])
    19  34.2773 MiB  -3.4805 MiB       b = 1
```

Very strangely, the assignment to ``b`` recovers memory from the system! This 
troubled me a lot - it didn't make sense that a random assignment statement 
should recover memory from our running application. Initially, I began to 
suspect that ``memory_profiler`` was flawed - that investigation led me down a 
very deep rabbit hole which I may write about another time. But, for the 
purposes of this post, I do have an explanation for the above behaviour - the
Python Garbage Collector! With a limit of ``100000``, the temporary list kept
triggering garbage collection and ``memory_profiler`` dutifully reports the 
system state as such. 

All-in-all, I'm satisfied with how this analysis turned out - the fact that 
generator expressions *do* save memory and that it's possible to prove the 
fact! 
