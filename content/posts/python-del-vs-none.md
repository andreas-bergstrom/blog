+++
title = "Python del vs assigning to None"
tags = [
    "python",
    "memory management",
]
description = ""
slug = "python-del-vs-assign-none"
date = "2019-03-01"
draft = false
+++

Python utilizes garbage collection to free the developer from the hassle of manually handling allocating and de-allocating memory. But there are still some details that could suprise you unless you are aware of them.

Because even though the Python runtime will take care of memory management, sometimes developers will want to manually tell the garbage colletor that a variable is no longer needed. Either because they are doing some edge-case optimization and really know what they are doing or they think they are smart but have no idea what they are doing. Most likely the latter as the Python runtime most often does not release unused memory back to the operating system.

But when someone need to somehow mark a variable as ready to be freed by the garbage collector, there are usually two solutions they come across:

{{< highlight python >}}
x = None
{{< / highlight >}}

or

{{< highlight python >}}
del x
{{< / highlight >}}

The difference is that **x = None** will free whatever it referenced but keep the name around even though it's just referencing None (which is a type, NoneType).

On the other hand **del x** will completely remove both the name and what it referenced. If you thereafter try to use x an NameError will be thrown (or AttributeError in case of a object property).

So in practice, by assigning None to a name you can still use it in expressions while using del the name is completely removed. In the first case a few bytes is needed to keep the name in memory, while the later completely clears all memory usage.

{{< highlight python "linenos=table" >}}
import sys
import gc

x = 'Some text here to give the variable a decent size'
y = 2
print('x value before deletion: {}'.format(x))
print('x size before deletion: {} bytes'.format(sys.getsizeof(x)))
print('y value before deletion: {}'.format(y))

x = None
del y
gc.collect() # Not really needed, just to show garbage collection has been done hereafter

print('x value after deletion: {}'.format(x))
print('x size after deletion: {} bytes'.format(sys.getsizeof(x))) # A few bytes needed to keep symbol name
print('x type after deletion: {}'.format(type(x)))

if not x:
    print('Can still use x!')

print('y value after deletion: {}'.format(y)) # Will throw NameError (AttributeError in case of class property)
{{< / highlight >}}

Output:

{{< highlight bash >}}
x value before deletion: Some text here to give the variable a decent size
x size before deletion: 98 bytes
y value before deletion: 2
x value after deletion: None
x size after deletion: 16 bytes
x type after deletion: <class 'NoneType'>
Can still use x!
Traceback (most recent call last):
  File "Untitled.py", line 21, in <module>
    print('y value after deletion: {}'.format(y)) # Will throw NameError (AttributeError in case of class property)
NameError: name 'y' is not defined
{{< / highlight >}}

As you can see in the error above, if you for whatever reason need to hook into the memory management, unless you know the difference between assigning to None and using **del** it could confuse you.

Let us now look at the bytecode to see the technical difference:

{{< highlight python "linenos=table" >}}
import gc
import dis

def using_none():
	x = 1

	x = None # <-- ASSIGNING TO NONE

def using_del():
	x = 1

	del x # <-- USING DEL

print("Bytecode when assigning to None:")
dis.dis(using_none)
print("\nBytecode when using del:")
dis.dis(using_del)
{{< / highlight >}}

Output:

{{< highlight bash >}}
Bytecode when assigning to None:
  5           0 LOAD_CONST               1 (1)
              2 STORE_FAST               0 (x)

  7           4 LOAD_CONST               0 (None)
              6 STORE_FAST               0 (x)
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE

Bytecode when using del:
 10           0 LOAD_CONST               1 (1)
              2 STORE_FAST               0 (x)

 12           4 DELETE_FAST              0 (x)
              6 LOAD_CONST               0 (None)
              8 RETURN_VALUE
{{< / highlight >}}

So using **del** results in only one bytecode, DELETE_FAST, while assigning to None actually runs a full assignment using LOAD_CONST and STORE_FAST. The last LOAD_CONST before RETURN_VALUE in both functions is simply the return value None that every Python function implicitly returns.