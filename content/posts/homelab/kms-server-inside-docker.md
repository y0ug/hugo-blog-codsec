---
title: "KMS server inside Docker to activate Windows VMs"
date: 2022-11-07T19:10:49+01:00
draft: false 
description: Getting windows activated by running a KMS server in Docker
tags:
- homelab
- docker
- windows
- KMS
- server
---

To get Windows activated on VMs is complicated due to the hardware change etc... One way to activate it without difficulty is to use *KMSPico*.

*KMSPico* work by emulating a *Windows KMS server*. You set the system to use the *KMSPico* as KMS server, you set a volume license and Windows get activated. Every once in a while, windows try to connect the KMS server and verify the license. You get it activated for up to 180 days.

I don't really want to run KMSPico as KMS server, it's a Windows binary, it's detected by AVs and getting a clean version is complicated.

There is other open source implementation, I choose to use [py-kms](https://github.com/SystemRage/py-kms), and they even provide a Docker build to be run.

I just had to add it to one of my server running Docker

```
docker run -d --name py-kms --restart always -p 1688:1688 pykmsorg/py-kms
```

They have a good [documentation](https://py-kms.readthedocs.io/) and a list of [volume license ready to be used](https://py-kms.readthedocs.io/en/latest/Keys.html)

To activate Windows 10 

```
cscript //nologo slmgr.vbs /upk 
cscript //nologo slmgr.vbs /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
cscript //nologo slmgr.vbs /skms 192.168.10.2:1688
cscript //nologo slmgr.vbs /ato
cscript //nologo slmgr.vbs /ato
cscript //nologo slmgr.vbs /dlv
```

* `/upk` is to remove all the KMS configuration
* `/ipk` set the license
* `/skms` set the KMS server IP
* `/ato` run the activation, normally you have to run it twice
* `/dlv` show current license information