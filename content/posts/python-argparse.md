---
title: "argparse - modern Python command line arguments"
date: 2019-03-05T21:10:21+01:00
draft: false
toc: false
images:
tags: 
  - python
  - command line
---

If you search for information on how to write Python modules that accepts command line arguments, you are very likely to stumble upon sys.argv and getopt() and go with it. But since Python 3.2 there is a better alternative that requires less boilerplate and less re-inventing the wheel: argparse.

Just like getopt() argparse relies on sys.argv but it is more high-level and provides more functionality out of the box. At the core of argparse is the class ArgumentParser, which returns an object that you can use to create the argument interface. This interface will automatically provide argument parsing and generate the help text that users expect when using -h or --help.

We will start with a simple program that accepts two positional arguments:

{{< highlight python "linenos=table" >}}
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("positional_argument_1")
parser.add_argument("positional_argument_2")
args = parser.parse_args()

print(args.positional_argument_1)
{{< / highlight >}}

So first a ArgumentParser-object is created, which we then add two (requied) positional arguments to and finally we dispatch the parser. This will then look into sys.argv and figure out how to parse its contents (which is all arguments provided by the user).

That looks very neat and object-oriented. But we could instead access any positonal arguments directly through sys.argv with much less code:

{{< highlight python "linenos=table" >}}
import sys

print(sys.argv[1])
# note: 0 = the name of the script, first positional argument at 1
{{< / highlight >}}

So what is the point of using argparser? Let's look at a more functional example:

{{< highlight python "linenos=table" >}}
import argparse
 
parser = argparse.ArgumentParser()
parser.add_argument("-s", "--string", required=True,
	help="string to uppercase")
args = vars(parser.parse_args())
 
print(f"Uppercase of {args['string']} is {args['string'].upper()}")
{{< / highlight >}}

Here we can see argparse starts to shine. This time we add an argument as a flag argument instead of positional. It is also set as required and a small help text is provided. This is what is returned if we run the script without any arguments:

{{< highlight bash >}}
usage: main.py [-h] -s STRING
main.py: error: the following arguments are required: -s/--string
{{< / highlight>}}

So argparse will automatically make sure all required arguments are provided. And it also displays usage information on what arguments this program expects, something familiar to anyone who has used command line programs. If we provide only the -h flag futher information is shown:

{{< highlight bash >}}
usage: main.py [-h] -s STRING

optional arguments:
  -h, --help            show this help message and exit
  -s STRING, --string STRING
                        string to uppercase
{{< / highlight>}}

This format should be even more familiar if you have played around on the command line. And argparse will automatically provide this help interface from the arguments that is specified, we did not have to set anything up ourselves!

So now we know how to use this program and if we provide -s andreas it will return:

{{< highlight bash >}}
Uppercase of andreas is ANDREAS
{{< / highlight>}}

For this input sys.argv will contain either ['main.py', '-s', 'andreas'], or ['main.py', '--string', 'andreas'] if we had used the --string flag instead. But thanks to argparser we no longer have to parse this manually and can ignore all possible edge-cases and order.

This is just the basics and I plan to cover more advanced usage sometime. The [docs](https://docs.python.org/3.3/library/argparse.html) has a more complete reference.

While argparser only handles arguments sent when starting the program, you can use [python-prompt-toolkit](https://github.com/prompt-toolkit/python-prompt-toolkit) to create more interactive command prompts.

{{< todo >}}
## Combining optional and positional arguments
- Combine positional (with help="help text") and optinal arguments
- Argument types, number of arguments, metavar
- defaults
- nargs (0-9 or +)
- ArgumentParser(prog="name")
- group = parser.add_mutually_exclusive_group()
- action (store_true)
- advanced example like accumulate in docs with dest, const, default
- custom actions
{{< / todo >}}