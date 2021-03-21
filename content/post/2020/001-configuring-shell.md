---
title: "Customizing my default shell to zsh"
date: 2020-04-27
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: ["shell"]
categories: []
---

I like to change the default shell environment on systems that I use to ``zsh``
and theme them a little. This post describes my usual changes. 

<!--more-->

# Installing zsh

Here is how to install ``zsh`` on an Ubuntu system,

```shell
$ sudo apt-get install zsh
```

Next we need to change the default login shell for the current user. Here is the
command (this is a bash commmand),

```shell
$ chsh -s $(which zsh)
```

After running the command, you need to log-off and log-in again. Just closing
and re-opening the terminal isn't enough. If you want to confirm the changes 
before logging off, you can visit ``/etc/passwd`` to see the default shell 
configured for the different users. I suggest not to edit that file directly. 


# Installing oh-my-zsh

This could be done automatically, but I feel it gives more control to do it 
manually. First we need to clone the repository,

```shell
$ git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
```

Next, to create a new zsh configuration file,

```shell
$ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

The above command will over-write the existing ``.zshrc`` file. Considering that
this was a clean install, I'm not concerned about making backups. 


# Configuring powerlevel10k

One of the reasons I like to use zsh - the wonderful theme support. There used 
to be an older 9k version of this theme which got depreciated. To install this
version it is a simple matter of cloning the repo :)

```shell
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
    ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
```

And finally, to set ``ZSH_THEME="powerlevel10k/powerlevel10k"`` in the 
``~/.zshrc`` file. 

After all this is done, opening a new shell will start a configuration utility 
for the theme. We can edit ``~/.p10k.zsh`` to customize the theme. Some of the 
changes I like to tweak are,

* disable ``virtualenv`` config, since ``pyenv`` takes care of it 
* enable 2-line support 
* changing the prompt to ``$`` - am just more used to it


# Environment customizations

Tools like ``pyenv`` or ``nvm`` need some customizations which I throw into the 
``~/.zshrc`` file at this point (after installing oh-my-zsh ). 
