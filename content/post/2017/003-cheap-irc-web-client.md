---
title: "Cheap IRC web client - The Lounge"
date: 2017-11-13
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: []
categories: []
---

For a year, I subscribed to IRCCloud. It's a good service that let's me stay 
connected to different chatrooms. In the year that I've been a customer, it has
served me well. So, why change? It costs USD $50/year. Recently, it has come to
my attention that you can rent servers on vultr for just USD $2.5/month.

Granted, it's a really low-end server, but for the needs of a single user it 
might just fit my needs. To my pleasant surprise, it works!

<!--more-->

The basic idea works like this - rent a server, install thelounge on it, setup
credentials, activate it as a service and connect via a web browser. 

Renting a server was easy - just pay vultr, tell them I need the cheapest
ubuntu server they could offer and save the root credentails - SSH to verify 
that the system was provisioned and I was in business. 

Installing thelounge was a bit tricky - I didn't know where to start! But that
was solved by looking up the dependency tree. The Lounge was built on the
nodejs interpretor. Which meant that I needed to get that installed. There were 
a few versions available, and I decided on the v6.x LTS series. Steps went like,
install GPG key from nodesource, add the repository to apt-get and install
nodejs.

Installing louge was straight-forward after that. I wasn't interested in running
anything else on the server, so I just installed it globally via the npm install
command. Not really sure what that means from a system hardening perspective,
but for a single user system, should be fine. I think. Worst case, just throw
out the server, get a new one and repeat. 

Next step was to ensure that thelounge would be running as a service. Time to
configure systemd. I made a configuration file and copied that over to my
vultr server. 

Additionally, I also created a user.json file, self-signed SSL credentials and
restarted the server. 

Last step was to connect to it over a web browser via IP. (Didn't want to setup
DNS for this)

And it worked!

For my next step, I'm thinking of automating the process. Perhaps I'll make a
dedicated project on how to deploy the server and share it on github. I have
an ansible script that I built, but that's for another time. 
