---
title: "Installing docker and docker-machine on Ubuntu"
date: 2020-04-28
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: ["shell"]
categories: []
---

I want to be able to quickly spin up game servers for small sessions and tear 
them down afterwards. Online investigation reveals that ``docker-machine`` 
paired  with DigitalOcean as a provider might be just what is needed. That claim 
needs to be tested, but a pre-requisite is that the ``docker`` and 
``docker-machine`` tools be available on the local system. This post will 
describe how to install them on an Ubuntu box.

<!--more-->

# Repository setup

First we need to get all our dependencies installed,

```shell
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

The purpose of installing the above is to allow the ``apt`` command to be able 
to fetch repositories over HTTPS. Next we need to grab Docker's official GPG 
key,

```shell
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

It will be good to manually verify that we have the correct key with the 
following fingerprint = ``9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88``. 
We can search for installed fingerprints by the last few characters, 

```shell
$ sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
    Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```

Finally, we can use the following command to setup the stable repository,

```shell
$ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```


# Installing Docker Engine

With the above done, it becomes very straight-forward to install the docker
engine with the following command,

```shell
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```


# Installing Docker Machine 

Strangely enough, the official suggestion is to download the Docker Machine 
binary and extract it to the user's PATH via this command,

```shell 
$ base=https://github.com/docker/machine/releases/latest/download &&
    curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
    sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
    chmod +x /usr/local/bin/docker-machine
```

At the time of typing, the lastest version for ``docker-machine`` is ``0.16.2``.
