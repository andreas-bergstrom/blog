---
title: "Python Tips and Gotchas"
date: 2019-03-28T12:24:57+01:00
draft: false
toc: true
images:
tags: 
  - python
---

<hr>
<div style="text-align: center">
**This post is a work in progress and will expand over time.**
</div>
<hr>

## Replace literals with constants in if-expressions

Whenever you need to check the value of a basic data type, the most straight-forward solution is to use literals:

{{< highlight python "linenos=table" >}}
if direction == 'up':
  # ...
elif direction == 'down':
  # ...
{{< / highlight >}}

This might seem like pythonic code, but its simplicity could bite you. If you have a lot of conditions and perhaps in multiple locations in your code, what happens if you mistype any of those string literals? Python will not complain, and there will not be any runtime errors. But you will end up with logical errors that could be insanely hard to track down.

Numeric literals are double trouble as the also comes with the issue of [magic numbers](https://en.wikipedia.org/wiki/Magic_number_(programming)). In the following example we not only have to guess what the number 545 really means, but it is almost easier to mistype than a string since we can not use common sense to detect a spelling error:

{{< highlight python "linenos=table" >}}
if error_code == 545:
  # ...
{{< / highlight >}}

In order to make your code more readable and to make it crash on any typos, using constants is often recommended. Simply extract the raw values into appropriately located constants (in Python just uppercased variables) and use the constants in the all if-expressions:

{{< highlight python "linenos=table" >}}
UP = 'up'
DOWN = 'down'

if direction == UP:
  # ...
{{< / highlight >}}

And with numeric literals:

{{< highlight python "linenos=table" >}}
SERVER_DOWN = 100
CONNECTION_ERROR = 200

if error_code == SERVER_DOWN:
  # ...
{{< / highlight >}}

In the first example Python will throw a NameError if you by mistake type "UPP", "uP" or "OP". Had we used string literals, Python would have had no way to recognize your typing error.

In larger projects you could group all constants in one or more modules. Since Python 3.4 there is also a new tool to help organize this: enums.

Enums are declared as classes but inherit from enum.Enum. This allows us to group related constants under a canonical name:

{{< highlight python "linenos=table" >}}
import enum

class ErrorCode(enum.Enum):
  SERVER_DOWN = 100
  CONNECTION_ERROR = 200

error = 100

if error == ErrorCode.SERVER_DOWN.value:
    print('Server is down')
{{< / highlight >}}

Enums can be compared with the ***is*** keyword. If you are in control of how each operand in a comparison is created, you could therefor use enums throughout your code to increase the readability even futher:

{{< highlight python "linenos=table" >}}
import enum

class ErrorCode(enum.Enum):
  SERVER_DOWN = 100
  CONNECTION_ERROR = 200

error = ErrorCode.CONNECTION_ERROR

if error is ErrorCode.CONNECTION_ERROR:
    print('There is a connetion error')
{{< / highlight >}}

Do however keep in mind that [enum attribute lookups are slower than regular attributes lookups](https://stackoverflow.com/questions/30812793/how-to-use-python-3-4s-enums-without-significant-slowdown).

## Microseconds and accuracy in time.sleep()

The sleep function in the time module can be used to halt execution for a given number of seconds. But it is easy to miss that the argument can be a float as well. In other words, you can use microseconds and not just whole seconds:

{{< highlight python "linenos=table" >}}
import time
time.sleep(0.05)
{{< / highlight >}}

This can be useful when you need to make a short pause but not necesseraly in seconds. For example if you are hammering an internal or external API in some batch job it could be useful to let the other end catch its breath for a while by pausing for a few hundred microseconds.

Though you should not trust the accuracy of this function down to the microseconds, wether you are using seconds or microseconds. Since Python's sleep() uses the current operating system's underlying sleep-functionality, unless you are using a real-time system this function will not be 100% accurate:

{{< highlight python "linenos=table" >}}
import time
import timeit

time = timeit.timeit('time.sleep(0.05)', number=100)
print(time)
time = timeit.timeit('time.sleep(0.05)', number=100)
print(time)
time = timeit.timeit('time.sleep(0.05)', number=100)
print(time)
{{< / highlight >}}

{{< highlight python >}}
5.273200581
5.261563578
5.282216143000001
{{< / highlight >}}

## Be more effecient with the REPL

The REPL is certainly one of Python's most unique features. While there are other languages that also provides some kind of interactive prompt, none is as fleshed out as in Python. And while it is easy to get started with, there a few tips that will help you work faster.

### Interactive execution of modules and packages

Probably the most useful feature of the REPL is the ability to load modules and packages in interactive mode. This is essentially the same as just doing ***import module_name*** while in the REPL, but this can also be done when launching the REPL. Simply use the -i flag and provide the name of the module or package (the package most be executable, in other words contain a __main__.py file):

{{< highlight python >}}
python3 -i filename.py
>>> locals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x10e02c630>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, '__cached__': None, 'lst': [1, 2, 3, 4, 5]}
>>> lst # imported from provided module at start
[1, 2, 3, 4, 5]
{{< / highlight >}}

If the module/package relies on a venv you obviosly need to activate that venv before launching the REPL. Also note that you can only provide one module or package with the -i flag, any futher arguments following the first will be ignored.

### Passing arguments to the REPL

Another sometimes useful feature is the ability to pass regular arguments to the REPL, just as is possible when executing Python-files. This allows you to for example pipe information from bash in to your REPL-session. These arguments become available through sys.argv just as when executing a file:

{{< highlight python >}}
python3 - arg1 arg2 arg3
Python 3.7.2 (v3.7.2:9a3ffc0492, Dec 24 2018, 02:44:43) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys; sys.argv
['-', 'arg1', 'arg2', 'arg3']
{{< / highlight >}}

Or if you prefer [argparse]({{< ref "python-argparse.md" >}}) just configure it like:

{{< highlight python >}}
>>> import argparse
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('args', metavar='N', type=str, nargs='+', help='arguments')
_StoreAction(option_strings=[], dest='args', nargs='+', const=None, default=None, type=<class 'str'>, choices=None, help='arguments', metavar='N')
>>> args = vars(parser.parse_args())
>>> args
{'args': ['arg1', 'arg2', 'arg3']}
{{< / highlight >}}

One useful example is if you need the contents of a SSH-key in the REPL:

{{< highlight python >}}
python3 - $(cat ~/.ssh/id_rsa.pub)
Python 3.7.2 (v3.7.2:9a3ffc0492, Dec 24 2018, 02:44:43) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys;sys.argv
['-', 'ssh-rsa', 'AAAAB3NzaC1yc2EAAAADAQABA...
{{< / highlight >}}

### Use underscore instead of temporary variables

Often when the REPL is used to experiment you will need to juggle values back and forth. In those scenarios it is common to create a lot of temporary variables just to keep the result of the last command around. But there is actually an easy way to do that without creating variables left and right.

All expressions evaluated in the REPL return a value, and if that value is **not** None it will actually automatically be stored in a built-in variable: _ (underscore):

{{< highlight python >}}
>>> 1 + 2
3
>>> _
3
>>> _ + 3
6
>>> _
6
>>> a = 1 + 2
>>> _
6
{{< / highlight >}}

Pay attention to the last lines, where the result of an addition operation is assigned to a but _ still contains 6 since the assignment expression returned None.

You can still use _ as a variable name and assign it normally, but that will cancel its default behaviour. To reset it, just run **del _**:

{{< highlight python >}}
>>> 1 + 1
2
>>> _
2
>>> _ = 'test'
>>> 1 + 2
3
>>> _
'test'
>>> del _
>>> 5 + 5
10
>>> _
10
{{< / highlight >}}

{{< todo >}}
## Metaclasses
- Meta class vs class decorators vs inheritance (base class)
- Meta class is to a class what a class is to a object. Classes are instances of a metaclass (type)
- Class factory with type(), class decorators and meta classes
- the type metaclass (type of type is type?)
- Allows for protocol-pattern, although at runtime instead of compile-time
- __init_subclass__(cls, *a, **kwargs)
## Import system
- modules and packages
- locals()
import x runs __import__() and binds to local scope. Running __import__() manually imports and updates caches (sys.modules etc.), but does not bind to local scope
- sys.modules
- sys.path, PYTHONPATH added to sys.path
- append to sys.path at run-time
- the python import order with built-ins, packages and modules
- standard library modules varies slightly between platforms, winreg only on Windows
- import flow row by row, AttrErrror on cross-import. module with print() and class with print() executed on import.
- import executes only once, when first imported. Appropriate for i.e. module level collections
- __file__ and __path__ for modules and packages
- __init__ in packages runs on import, useful to hoist modules/classes to top level for cleaner usage (look at requests api-file)
- __all__ in __init__ for from X import *
- namespace package has no __init__.py, useful to split up large packages
- namespace package split between different directories same logical package, test with __path__
- __main__.py for executable directory
{{< / todo >}}