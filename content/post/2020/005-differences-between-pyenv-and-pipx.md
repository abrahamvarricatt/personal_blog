---
title: "Differences between Pyenv and Pipx"
date: 2020-12-13
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: ["python"]
categories: []
---

I have been told that ``pipx`` is a very good tool to install and run Python 
applications in isolated environments. That description almost immediately 
reminds me of another tool - ``pyenv``. Even though they address different 
needs, I find myself mixing them up. The purpose of this post is to describe 
what the tools try to accomplish and how they go about doing so. A pre-requisite 
to this discussion is an introduction to how the system ``$PATH`` environment 
variable is used.

<!--more-->

# Introducing $PATH

At a fundamental level, ``$PATH`` is an environment variable which consists of
a list of directories separated by colons (``:``). Here is a quick example,

```shell
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

When a command is entered, the OS scans this group of directories (in the order 
given) for a corresponding executable to run. For example, assume we wish to 
run the ``docker-compose`` executable. Here are the different attempts the OS
makes, 

```text
/usr/local/sbin/docker-compose  (NOT FOUND)
/usr/local/bin/docker-compose  (NOT FOUND)
/usr/sbin/docker-compose  (NOT FOUND)
/usr/bin/docker-compose  (FOUND)
```

If it is unable to find the executable within any of the directories on the 
``$PATH`` , an error message is returned to the user.

A ... divergence, or an exception to the above behaviour is when ``shims`` are 
involved. You can think of them as light-weight utilities which process the 
name of the executable searched for within the directory and respond 
accordingly. 


# About pyenv

``pyenv`` is a tool to help manage different Python environments on the same 
system. For example, one might have Python 2.7 installed for one project and 
Python 3.8 installed for another. On this system, if we were to randomly run 
``$ python --help`` which version would we reasonably expect to answer? 

One approach is to manually manage everything, adding or removing entries from 
``$PATH`` as needed. This gets messy and troublesome. A better alternative is 
to use the ``pyenv`` tool. When installed and initialized, ``pyenv`` adds a 
custom ``shim`` to the ``$PATH`` environment variable. The end result looks 
something like this,

```shell
$ echo $PATH
/home/abrahamv/.pyenv/shims:/usr/local/sbin:/usr/local/bin ...
```

At the point, every command executed is processed via ``pyenv`` (via the shim). 
If it decides that it is a Python-related command, ``pyenv`` will redirect 
execution to the corrsponding Python executable. 

For the most part, this works very well. An interesting side-effect of this 
design, is that the ``pyenv`` tool needs to be active on the system. In 
contrast, ``pipx`` does not. 


# About pipx

As per the official documentation "pipx is a tool to help you install and run 
end-user applications written in Python". It will be helpful to understand what 
we mean by "end-user applications" in this context. The background is that 
Python and PyPI allow developers to distribute code with "console script entry 
points". These allow users to call into the Python code from the command line, 
effectively acting as standalong applications. 

pipx is a tool to install and run any of these standalone applications in a 
safe, convenient and reliable way. 

On hearing all that, I expected to install ``pipx`` as a global tool on my 
system, similar to what I did for ``pyenv``. It greatly confused me that this 
was not the case. In fact if you want to keep environments isolated, the 
easiest way to use pipx is to create a new Python virtual envrionment and just
install it via ``pip`` ! Something like this,

```shell
$ pyenv virtualenv 3.8.5 temp_for_pipx
$ pyenv activate temp_for_pipx
$ pip install pipx
```

At this point, let us assume that we want to install the ``pycowsay`` CLI tool 
from PyPI. Since ``pipx`` is already present in our current virtualenv, all we 
need to do is, 

```shell
$ pipx install pycowsay
```

Running the above command will create a new virtualenv at 
``/home/USER/.local/pipx/venvs`` and a symbolic link in the 
``/home/USER/.local/bin`` directory which points to the executable within the 
virtualenv. 

What is most remarkable is that we can now delete the virtualenv created for 
``pipx``, but our installation of ``pycowsay`` will still be present!

```shell
$ pyenv deactivate temp_for_pipx
$ pyenv uninstall temp_for_pipx
$ pycowsay mooo
zsh: ... bad interpreter: /home/USER/.local/pipx/venvs/pycowsay/bin/python: no such file or directory
```

Oookay. That was disappointing. Looks like my original intuition about ``pipx`` 
being dependant on the virtualenv from where it gets installed from is accurate. 

In this scenario, what I would like to happen is for ``pycowsay`` to pick up my 
"root" Python 3.8.5 executable. Hmm... according to the ``pipx`` docs, that 
might be possible. Lets give this a shot,

```shell
$ pyenv virtualenv 3.8.5 temp_for_pipx
$ pyenv activate temp_for_pipx
$ pip install pipx
$ pipx uninstall pycowsay     # to remove the earlier installation
$ pipx install --python $(pyenv root)/versions/3.8.5/bin/python pycowsay
$ pyenv deactivate temp_for_pipx
$ pyenv uninstall temp_for_pipx
$ pycowsay mooo

----
< mooo >
----
    \   ^__^
    \  (oo)\_______
        (__)\       )\/\
            ||----w |
            ||     ||
```

Yay! That worked!!

In conclusion, this is going to my general approach to installing things via 
``pipx``,

```shell
$ pipx install --python $(pyenv root)/versions/VERSION-NUMBER/bin/python <TOOL>
```
