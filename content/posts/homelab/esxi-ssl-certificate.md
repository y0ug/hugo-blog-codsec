---
title: "ESXi SSL certificate setup"
date: 2022-02-21T00:00:00+01:00
draft: false 
description: Setting up  a SSL certificate on an ESXi server 
tags:
- homelab
- vmware 
- esxi 
- ssl 
- certificate 
---

Note on how I set up a let's encrypt SSL certificate on an ESXi (version 7 or 8).
The certificate is generated with ACME.sh, I'm using a Cloudflare DNS verification. Upload on the ESXi host is done with over SCP.

```
 export CF_Account_ID=
 export CF_Token=
acme.sh --issue --dns dns_cf -d nuc-esxi.mazenet.org
cp ~/.acme.sh/nuc-esxi.mazenet.org/nuc-esxi.mazenet.org.cer rui.cert 
cp ~/.acme.sh/nuc-esxi.mazenet.org/nuc-esxi.mazenet.org.key rui.key
scp rui.cert rui.key root@nuc-esxi.mazenet.org:/etc/vmware/ssl/
rm rui.cert rui.key
```

SSH needs to be enabled on the ESXi host. Once the certificate is installed, the interface needs to be restarted. It's possible to do it with the command `dcui` and select **Troubleshooting Options > Restart Management Agents**

Resource from VMWare [Configuring CA signed certificates for ESXi 6.x/7.0 hosts (2113926)](https://kb.vmware.com/s/article/2113926)

