---
layout: post
identifier: python-reference-cycle-collector
title: "Python's reference cycle collector"
date: 2019-10-19
comments: true
---
The standard implementation of CPython 3 has
[both](https://docs.python.org/3/faq/design.html?highlight=garbage%20collection#how-does-python-manage-memory)
_reference counting_ and a generational _garbage collector_. The generational garbage collector
is responsible for collecting reference cycles periodically.
To review, here is an example of a reference cycle (note the use of `.append`).
```python
x = [1, 2, 3]
y = [x]
x.append(y)
print(x)
# [1, 2, 3, [[...]]]
# [...] is a tell-tale sign of a ref cycle
```
Let's see the generational garbage collector in action. Define a few helper functions
for our investigation.
```python
import sys, os, gc, psutil  # ignore E401
import numpy as np

process = psutil.Process(os.getpid())

def print_memory():
    memory_mb = int(np.round(process.memory_info().rss / 1e6))
    print('{}MB'.format(memory_mb))

def create_ref_cycle():
    a = [np.random.rand(2000, 2000)]
    b = [a]
    a.append(b)  # using .append() creates a ref cycle

def no_create_ref_cycle():
    a = [np.random.rand(2000, 2000)]
    b = [a]
    a = a + [b]  # redefining `a` doesn't create a ref cycle
```
Garbage collector is enabled by default. Let's disable the garbage collector and create
lots of reference cycles that become inaccessible to us.
Measuring memory usage is a messy business that depends on lots of factors including the OS type.
We will look at the resident set (`rss`) memory which may not be what your OS's
GUI reports (such as the Activity Monitor in
MacOS). Your results will vary wildly every time you run these snippets.
Our aim is to notice the effect of enabling or disabling the garbage collection instead of
trying to get accurate memory usage. Also note that Python itself
[doesn't](https://rushter.com/blog/python-memory-managment/) always return all of the unused
memory back to the OS.
```python
# In a new python session with helper functions defined
gc.disable()
print_memory()
# 63MB
for _ in range(100):
    create_ref_cycle()  # consumes ~32MB per iteration
    print_memory()
# ...
# 3168MB
# 3200MB
# 3232MB
# 3264MB
gc.enable()
gc.collect()
print_memory()
# 2112MB
```
Let's now run the function that does _not_ create reference cycles. The memory usage for this
function does _not_ keep on increasing as a result of reference cycles
(though there might be minor increases due to other, unrelated reasons).
```python
# In a new python session with helper functions defined
gc.disable()
print_memory()
# 63MB
for _ in range(100):
    no_create_ref_cycle()  # consumes nothing per iteration
    print_memory()
# ...
# 95MB
# 95MB
# 95MB
# 95MB
gc.enable()
gc.collect()
print_memory()
# 95MB
```
The absolute memory numbers are not reliable -- they vary by OS, RAM, other processes,
and lots of other factors. Garbage collection runs periodically (not continuously) based on
[heuristics](https://rushter.com/blog/python-garbage-collector/). You may see a temporary
increase in memory usage until the garbage collection runs again. GC is also
[not perfect and does not](https://pythoninternal.wordpress.com/2014/08/04/the-garbage-collector/)
prevent all memory leaks.

##### Takeaway
Don't create reference cycles in your code. Know that the garbage collection in python is
imperfect.
