---
title: "Synchronizing time on a dual-boot system"
date: 2018-02-18
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: []
categories: []
---

The time displayed by my OS is different between Ubuntu and Windows 10. Opening
the settings screen on Windows on each bootup, turning the time sync off/on was
getting tiring. Here is how I solved the problem in a more permanent manner.

<!--more-->

The root cause of the issue is due to difference in behaviour with how each OS
was treating the time reported by BIOS. Ubuntu treated it as UTC time, but 
Windows treated it as localtime. This meant that when either of the operating
systems would sync time against an online time server, it would set the BIOS
clock in a manner different than the other expected.

Between the two approaches - use BIOS time as either UTC or local, I would 
personally prefer UTC. This would mean that BIOS would not need to be updated 
each time daylight savings came round every year, or if I was travelling and 
switched time-zones. 

Unfortunately, Windows does not make it easy to use the BIOS clock as UTC time.
Technically it's possible, but it involves messing around with the Registry 
Editor and I'm worried that other applications wouldn't play well with the 
changes. 

Which leaves modifying Ubuntu as the other option. Fortunately, this isn't too
difficult. Just run the following command,

```bash
$ timedatectl set-local-rtc 1
```

What the above does, is tell Ubuntu to treat the BIOS clock as localtime. 
Restarting my system a few times and switching between the OS's confirms that
it works.  
