---
title: "NAS on Raspberry Pi with USB backup"
date: 2017-05-28
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: ["raspberry-pi",]
categories: []
---

This post details the steps I took to configure a Network Accessable Storage
(NAS) using the Samba protocol on a Raspberry Pi 3 running the Raspbian Jessie
Lite distribution. This was done using the openmediavault (OMV) project.

For redundancy purposes, a USB memory stick was configured as a backup location
for the shared folder.

<!--more-->

# A new admin user

After logging in with the default ``pi`` root user, you can create a new root
account with the following commands,

```bash
$ sudo adduser admin
$ sudo usermod -aG sudo admin
$ su - admin  # Login as admin to verify things worked
```

At this point, you can create a new SSH connection into the Rpi using the new
``admin`` user and delete the default account,

```bash
$ sudo deluser --remove-home pi
```


# Configuring Timezone

The Rpi assumes UK as it's default timezone. This isn't something I usually care
for in experimental projects, but for a NAS where backups are a concern it is
probably best to use the local timezone. There is a linux utility which helps
set this up. You can begin the process with the following command,

```bash
$ sudo dpkg-reconfigure tzdata
```


# Installing openmediavault

The openmediavault project maintains an official debian repository with support
for the ARM architecture. You can add this as a source and install OMV using
the following commands,

```bash
$ cd /etc/apt/sources.list.d/
$ sudo touch openmediavault.list
$ sudo echo "deb http://packages.openmediavault.org/public erasmus main" > openmediavault.list
$ cd ~
$ sudo apt-get update
$ sudo apt-get install openmediavault-keyring postfix resolvconf
$ sudo apt-get update
$ sudo apt-get install openmediavault
$ sudo omv-initsystem
```

Bear in mind that some of these steps, ESPECIALLY the installation ones take
time to run - anywhere between 5 to 30 minutes. I think this is more to do
with the hardware limitations of the Rpi than network availability. And speaking
of the installation steps, the ``openmediavault`` one asks you a few questions
about mail service, mdadm and ProFTPD settings.


# EXT4 formatting

You can insert a USB memory stick to act as the primary location for our network
share. After that, using a web-browser on another system, you can login to the
administrative interface of openmediavault using the ``admin`` username and
password created earlier. The URL to use is the IP address assigned to the Rpi
system.

Once logged in navigate to ``Storage > Physical Disks``, select the USB device
to use as primary share and ``Wipe`` it. This will clear out any data on it.

Next navigate to ``Storage > File systems``, click the ``Create`` button and
format the primary USB memory device in EXT4. You can ``Mount`` the drive using
the web interface after that.


# User account for sharing

Navigate to ``Access Rights Management > User`` menu and create a new user
account. We'll use this as the account to login to the NAS with from client
devices.


# Configuring a SMB share folder

Navigate to ``Services > SMB/CIFS``. Select the ``Shares`` tab and add a new
shared folder. You might need to create one on our primary USB memory device.
Set the permissions as appropiate, return to the ``Settings`` tab and ``Enable``
the SMB share.

At this point, you should be able to use another system on same network to
connect to the network share (IP address of the Rpi) using the user account we
created in the earlier step. While it might be possible to use the ``admin``
account, I think it's bad practise to share a root-level account like that.


# USB backup

OMV doesn't come with USB backup as part of it's default configuration. You need
to add it in as a plugin. Navigate to ``System > Plugins`` and you should find
it listed under ``Section:Backup``. Install it and you'll find a new option
``Services > USB Backup``. You can create new backup jobs which will trigger
the first time they are created and whenever the USB drive is inserted into
the board. I select the ``From shared folder to external storage`` mode for
these jobs. It might be first required to insert the backup drives and reformat
them to EXT4 before making the jobs.


