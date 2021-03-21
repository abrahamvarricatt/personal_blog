---
title: "Raspberry Pi 3 for headless wifi"
date: 2017-05-13
lastmod: 2021-03-21
draft: false
keywords: []
description: ""
tags: ["raspberry-pi",]
categories: []
---

After following these instructions you will have a bootable microSD card which
can be inserted into a Raspberry Pi 3 to load up the Raspbian Lite image,
configured to auto-connect to your wifi network, with a server you can SSH into.
For good measure, we will also enable the UART serial console.

All the work is being done on a traditional x86-based Linux laptop - the Pi
won't be needed until you actually want to boot the image!

<!--more-->

# Downloads

There are 2 files we'll need to download. The first and most obvious is the
official Raspbian Lite image available here,

https://www.raspberrypi.org/downloads/raspbian/

I've tested against the April 2017 version. The second is Etcher; a
cross-platform tool to flash images. You can get it from,

https://etcher.io/


# Flashing the image

Etcher makes the process of flashing Raspbian images ridiculously easy! Open the
application, select our Raspbian Lite image, target memory device is
auto-selected and finally click the 'Flash!' button. In about 5 minutes the
flashed memory device will be verified and ready to be ejected from the
computer!


# Activating the SSH server

Re-insert the memory device. It will show up with two partitions - a FAT boot
partition and another ext4 partition. Let's refer to the latter as the system
partition.

You want to navigate into the boot partition and within the ``/boot`` directory,
create a file called ``ssh``. The contents don't matter - only the existence of
the file is checked to enable the SSH server.

Pay attention to the owners and permissions within the ``/boot`` directory.


# Enable the serial UART console

Within the same ``/boot`` folder is a file called ``config.txt``. You need to
edit this text file and append the following,

```text
enable_uart=1
```

That's all it takes to enable the UART console. Bear in mind that using
Bluetooth after this might come with a few hiccups.

As usual, pay attention to the owners and permissions within the folder.


# Configuring Wifi

Now you need to move into the system partition. Navigate to the folder called
``/etc/wpa_supplicant``. You'll need to append the following to the file named
``wpa_supplicant.conf``,

```text
network={
    ssid="YOUR_WIFI_SSID"
    psk="YOUR_WIFI_PASSWORD"
}
```

The quotes are important. These should be enough if the wifi ssid is being
broadcast and WPA2 Personal security settings are used. If that's not the case,
you'll want to refer to these instructions instead,

https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

Pay special attention to the file permissions and ownership settings here. I
think I needed to use ``sudo`` to make the edits.


# Conclusion

We're done. You can now eject the microSD card from your computer, insert it
into the Raspberry Pi 3 board, power it on, wait for booting to finish and ssh
into it from another system on the wifi network!

Assuming that you don't run into any hardware trouble (like corrupted microSD
cards!), you can use the default credentials to login to your Pi -

```text
Username : pi
Password : raspberry
```

Remember to give the Pi enough time to boot before you attempt to SSH into it.
About 1 - 3 minutes works for me with a Class 10 microSD card. For convenience,
you might want to configure the DHCP server on your network to allocate a
static IP address to the Pi 3 board.
