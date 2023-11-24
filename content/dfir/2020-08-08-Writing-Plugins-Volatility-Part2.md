---
title: Writing a simple Volatility plugin - Part 2
categories: 
    - DFIR
tags: 
    - Volatility
    - Python
author: Abhiram Kumar
date: 2020-08-08
draft: false
slug: simple-volatility-plugin-part-2
toc: true
---

This post covers the basics of writing a simple plugin for the Volatility framework using Unified Output and using generator functions in python.

<!--more-->

This post is a continuation of the [last post](https://stuxnet999.github.io/volatility/2020/07/04/Writing-Plugins-Volatility.html) that I've written on writing plugins for Volatility 2. In the previous post, we developed the function **render_text()** which was used to produce output to `stdout`.

In this article, I will be talking about why render_text() is less efficient and why an alternative is required to write plugins efficiently.

## Pre-requisites

+ If you are here for the first time, you may want to read the part-1 of this blog series [here](https://stuxnet999.github.io/volatility/2020/07/04/Writing-Plugins-Volatility.html).
+ You must be familiar with the usage of common data types in python (lists, tuples etc...)
+ Experience with using Volatility

As I had said in my previous post, the 3rd prerequisite is not required but its a plus if you have used volatility before.

## Quick recap

In this section, we'll have a quick recap about the final code we wrote in the previous blog post. We will modify the same piece of code and build a more efficient and smarter code using simple python concepts.

```python

import volatility.plugins.common as common
import volatility.utils as utils
import volatility.win32 as win32

class TestPlugin(common.AbstractWindowsCommand):
    """ Works exactly like pslist """

    def calculate(self):

        addr_space = utils.load_as(self._config)
        tasks = win32.tasks.pslist(addr_space)
        return tasks
    
    def render_text(self, outfd, data):
        for tasks in data:
            PID = tasks.UniqueProcessId
            CreateTime = tasks.CreateTime
            Process_name = tasks.ImageFileName
            outfd.write("{0}\t {1}\t {2}\n".format(PID, CreateTime, Process_name))
```
We created the **render_text()** function to list all the active processes in the system and we directed the output to stdout.

And it worked well too.

![output](/images/volatility/final.png)

## Drawbacks of render_text()

Though our previous plugin was working fine, it was not ideal. Let us see why.

+ Using **render_text()** limits us from producing the output of other common, important formats.
+ If we go by this logic, we might have to write a **render_X()** function to generate every other format we may need.
    + X &#8594; text, JSON, SQLite, xlsx, dot etc...

Volatility has a [standard list of renderers](https://github.com/volatilityfoundation/volatility/wiki/Unified-Output#standard-renderers) which are most commonly used in the industry. So we have to write code which is compatible with their standards.

So let us dive in and try to build a better plugin which can render all the standard formats.

## Unified output & generator

```python

def generator(self, data):

    for tasks in data:
        yield (0,[int(tasks.UniqueProcessId), str(tasks.CreateTime), str(tasks.ImageFileName) ])

def unified_output(self,data):
    tree = [
        ("PID", int),
        ("Create Tame", str),
        ("Process Name", str)
        ]
    return TreeGrid(tree, self.generator(data))
```
### Understanding the code

You might be familiar with the some of the terminologies here like **UniqueProcessId**, **ImageFileName**, **CreateTime** etc. I have covered the detailed definition of these terminologies in the previous post and also introduced about the **_EPROCESS** structure

So we have 2 functions now, generator() & unified_output(). 

**TreeGrid** takes in a tuple the name and data type of each column which is produced.

The `generator()` function returns the value which is mapped to the corresponding item in the tree (In this case PID, Process Name, Create Time ).

**Note**: The data type of the corresponding fields in both the `tree` tuple-list and the `yield` in the generator() must be the same.
{:.info}

Also **TreeGrid** is imported from the **volatility.renderers** module.

Now we have everything that we require. Let us incorporate this in our previous code and test it.

## Testing output

So the final code that we have is,

```python

import volatility.plugins.common as common
import volatility.utils as utils
import volatility.win32 as win32

from volatility.renderers import TreeGrid # Importing TreeGrid.

class TestPlugin(common.AbstractWindowsCommand):
    """ Works exactly like pslist """

    def calculate(self):

        addr_space = utils.load_as(self._config)
        tasks = win32.tasks.pslist(addr_space)
        return tasks
    
    def generator(self, data):

        for tasks in data:
            yield (0,[ int(tasks.UniqueProcessId), str(tasks.CreateTime), str(tasks.ImageFileName) ])

    def unified_output(self,data):
        tree = [
            ("PID", int),
            ("Create Time", str),
            ("Process Name", str)
            ]
        return TreeGrid(tree, self.generator(data))
```

We will test the output in some of the standard renderer types and see if our code works perfectly

### Testing TEXT

Let us proceed with the format most commonly used.

```bash
$ volatility --plugins=testplugin/ -f memorydump.vmem --profile=Win7SP1x86 testplugin --output=text
```
![text](/images/volatility-part2/unified-text.png)

### Testing SQLite

```bash
$ volatility --plugins=testplugin/ -f memorydump.vmem --profile=Win7SP1x86 testplugin --output=sqlite --output-file=test.sqlite
```
I'm using DB browser to view the SQLite file.

![sqlite](/images/volatility-part2/unified-sqlite.png)

### Testing DOT

```bash
$ volatility --plugins=testplugin/ -f memorydump.vmem --profile=Win7SP1x86 testplugin --output=dot --output-file=test.dot
```

![dot](/images/volatility-part2/unified-dot.png)

## Final thoughts

Now with this, I complete the series of writing simple volatility plugins. There can be more additions made to this code to make it a fully-fledged plugin, for example adding command-line arguments etc..

With learnings from this blog post, I hope you will be able to read and understand more of Volatility's plugin codebase.

## References

+ Unified output
    + <https://github.com/volatilityfoundation/volatility/wiki/Unified-Output>
+ Plugin development
    + <https://github.com/volatilityfoundation/volatility/wiki/Unified-Output#plugin-developers>
    + It is a template which I used for learning how to write a simple plugin.

Feel free to contact me on Twitter: <https://twitter.com/_abhiramkumar>