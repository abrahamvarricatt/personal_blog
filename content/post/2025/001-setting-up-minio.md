---
title: "Setting up Minio"
date: 2025-03-09T18:13:59-04:00
lastmod: 2025-03-09T18:13:59-04:00
draft: false
keywords: []
description: ""
tags: ["minio", "unraid"]
categories: []
---

I need to run some terraform experiments with my home infrastructure and need 
a better backend for the statefiles than the current directory. According to 
the official Terraform docs, S3 compatible backends *are* supported. A quick 
internet search leads me to the [Minio] project - it does a lot of things, but 
for purposes of this post, am going to talk about what I did to get a Minio 
service up and running on my Unraid server. 

<!--more-->

My usual way to install new services in Unraid is to open my web-browser, 
navigate to the server URL, log into the interface, navigate to the `APPS` tab, 
search for `minio`, select the result with a banner that says "OFFICIAL" and 
install it. 

The problem is that there is another banner in the description section for the 
`minio` app that says the following,

```
Attention:
Unfortunately due to changes in Minio, the unRAID file system is no longer 
supported. The only way to get Minio to work on unRAID now is by mapping a 
single disk directly or setting up a V-Disk.
```

which was very concerning.  What is going on here?

A bit of internet searching led me to this issue on Github,

https://github.com/minio/minio/issues/15054

My takeaway is that by-default, Unraid creates a filesystem named `shfs` 
(probably something based off FUSE) which it uses the expose mounts for 
different docker applications. This works for most situations, but due to the 
nature of what Minio does - it needs certain guarantees; specifically support 
for `O_DIRECT` - which is not supported by `shfs` and is instead faked. 

Thankfully, this does not mean that one cannot run `minio` in Unraid - just 
that some special precautions need to be taken. The solution is to create a 
dedicated share for `minio` and mount that instead. 

From the Unraid web interface, I navigated to the "SHARES" tab and clicked the 
"ADD SHARE" button. Named the share `atlantic_minio` and left the rest of the 
options at their default values. After creating the share, more options showed 
up - SMB settings on how this should be exposed on the network. No need to do 
that since the entire point is to have `minio` expose it as an S3 compatible 
interface. I set the "Security" dropdown to `Private` and denied access to all 
users.

Once that was done, it was back to the "APPS" tab to install `minio`! Here are 
some important settings,
```
Name: Minio
Repository: minio/minio
Network Type: Custom: atlantic_ocean

Web UI: 9000
Console UI: 9001
Data: /mnt/disk1/atlantic_minio/
Config: /mnt/user/appdata/minio

MINIO_ROOT_USER: <HIDDEN>
MINIO_ROOT_PASSWORD: <HIDDEN>
```

The default port numbers were different from what was exposed by the container. 
It is easier to remember `9000`, so I decided to go with that. 

After the container was up and running, I navigated to the `DOCKER` tab and 
checked the option to `AUTOSTART` the container. This means that if something 
were to go wrong with the unraid box, on boot, it would bring this service back 
online. Also enabled auto-update for the container via `SETTINGS > Auto Update 
Applications > Docker Auto Update Settings`. 

Final step was to add records on my home DNS servers to avoid remembering IP 
addresses. After that was done, I could navigate to 
`http://minio.home.blueapple.ca:9000` and get redirected to the login page. Yay!

## Creating a user for terraform

It is dangerous to work with the root (or admin) user account for a service. So 
I need to create a new user account to use with terraform. This means navigating 
to `Administrator > Identity > Users` and clicking the `Create User` button. 
When creating the user, I assigned it the `readwrite` policy. That will grant 
that account access to any bucket on this server. For immediate home-use, it 
should be fine. And if things get more complicated later, it should not be hard 
to change permissions. 

After the user is created, the next thing is to create an access key pair. 
This is what will be passed into terraform for authentication purposes. 

## Creating a new bucket

Last step is to create a bucket to store things in. Will name the bucket 
`home-infrastructure`. There are some features here like `Versioning`, 
`Object Locking` ..etc - am going to keep them disabled. 



[Minio]: https://min.io/product/s3-compatibility
