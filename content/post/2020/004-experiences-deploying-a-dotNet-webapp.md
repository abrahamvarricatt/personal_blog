---
title: "Experiences deploying a dotNet web application"
date: 2020-05-05
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: []
categories: []
---

Someone I know had lamented on how difficult he found it to manage a dotNet 
application online; i.e. a webapp running on Microsoft's dotNet Core 
technology. It got me pondering - is it really that hard to run dotNet on an 
Ubuntu box? At a fundamental level, we are still dealing with managing running 
processes/threads in an operating system. So I challenged myself to provision 
just such a system! This post mentions some of the memorable moments I had in 
accomplishing that challenge.  

<!--more-->

# What was the objective?

Managing a web-application online is at its core, an operations issue. It is 
not a trivial matter to keep a server updated, secure and always accessible. 
There are scaling concerns with how to handling load spikes, provisioning 
additional systems ... etc. To concern myself with all that would be a 
full-time job and not something I wanted to do as a side-activity. 

The objective here was to learn what it would take to provision a dotNet webapp 
server from scratch. i.e. to find out the exact sequence of steps required from 
the moment a host system is rented to the point when it can receive and respond 
to network requests. Most importantly, to record those steps in a repeatable 
and precise manner. 

To keep things simple, it would be acceptable for the server to respond to 
web-browser requests from it's IP address. i.e. I did not want to deal with DNS 
concerns. 


# Where is the solution?

I wrote all the provisioning scripts in Ansible, saved it to a Github 
repository and configured CircleCI to run those steps on a remote droplet 
rented from Digital Ocean. Here is the source code,

https://github.com/abrahamvarricatt/droplet_provisioner


# Challenge #01: Running Ansible on CircleCI

CircleCI containers do not come with Ansible installed. My first solution was 
to just install ansible as part of the CircleCI deployment script. It worked .. 
but added almost half a minute to my deployment time. Could things be faster? 
Turns out, yes. 

CircleCI allows us to configure custom docker images on which the deployment 
scripts would be run. After verifiying that I could pull the official Ubuntu 
image, I built my own Dockerfile with ansbile installed, uploaded it to Docker 
Hub as a public image and was able to use it as the container for my deployment 
scripts! 

If needed, CircleCI can be configured to use private images, but a free public 
image works for my purposes. 


# Challenge #02: Server authentication

Ansible scripts connect to remote hosts via SSH, which meant I could either use 
password-based authentication or an SSH private-public key pair. Digital Ocean 
allows me to create droplets with an intial randomly generated password which 
needs to be changed after the first log-in. I was pleasantly surprised that 
they also supported creating droplets with SSH key-pairs. How it works, is that 
I would need to upload a public key to them and ask for it to be included on 
the host during droplet creation. After the droplet is created, I would be able 
to log into the box using my private key.

The idea of uploading the private key to a public Github repository did not sit 
well with me. Thankfully, the option to store the key on CircleCI exists. For 
this to work, I needed to trust CircleCI with the private key and upload it to 
their servers linked to my project. Then as part of the deployment script, 
mention that the SSH key would need to be accessible. 

The final part of authentication was storing where the IP address of my server 
would be saved to. For the time being, I decided to just store it publicly as 
part of the Github repository. If it needs to be hidden, it can be saved as an 
environment variable on CircleCI. 


# Challenge #03: Supporting two versions of dotNet runtime

I planned to run two separate dotNet projects - one a webapp and another a 
background worker. Since there was no plan to actually learn dotNet 
programming, I decided to copy-paste code from online tutorials. The issue was 
that the code I found for both projects relied on two separate versions of 
dotNet.

Comparing this to how Python runtimes are handled, I expected this to be a 
challenge to setup. Turns out, it isn't the case. It was pointed out to me that 
multiple versions of dotNet can co-exist on the same system and that they would 
figure out between themselves which to run what app. 

Which meant that while this wasn't a challenge I needed to solve, it was an 
instance of me getting stuck due to not knowing how things worked. 


# Challenge #04: Installing the Entity Framework

The Entity Framework is a dotNet tool to help manage databases. The trouble I 
had was that it kept getting installed on the root account of my droplet. No 
matter what I tried, it stubbornly refused to work with the webapp user that 
was created. The most bizzare thing about this was that when installing the 
tool as the webapp user, dotNet would report that it was already installed! 
Running it, threw permission issues about how the webapp user account was not 
allowed to access it's folders. 

I traced this issue all the way back to how the dotNet runtime registers itself 
to user accounts on an Ubuntu system. It assumes that one is using the bash 
shell and appends an entry to the user's PATH. The webapp user was not 
configured to use the bash shell by default, which caused the dotNet runtime to 
mis-understand the situation. 

After setting the default shell of the webapp user to ``/bin/bash`` things 
began to work as expected. 

This was perhaps the most challenging issue about the entire process. 

