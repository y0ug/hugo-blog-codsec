---
title: "Ubiquiti Edgerouter SSL certificate setup"
date: 2022-02-21T00:00:00+01:00
draft: false 
description: Setting up a SSL certificate on an ubiquiti edgerouter 
tags:
- ubiquiti 
- edgerouter
- ssl 
- certificate 
- letsencrypt
---

Note on how I set up a let's encrypt SSL certificate on an ESXi (version 7 or 8).
The certificate is generated with ACME.sh, I'm using a Cloudflare DNS verification. 

```
 export CF_Account_ID=
 export CF_Token=
acme.sh --issue --dns dns_cf -d edgerouter.mazenet.org
cat .acme.sh/edgerouter.mazenet.org/edgerouter.mazenet.org.cer .acme.sh/edgerouter.mazenet.org/edgerouter.mazenet.org.key > ~/.acme.sh/edgerouter.mazenet.org/edgerouter.mazenet.org.cert
scp ~/.acme.sh/edgerouter.mazenet.org/edgerouter.mazenet.org.cert ubnt@edgerouter.mazenet.org:
scp ~/.acme.sh/edgerouter.mazenet.org/fullchain.cer ubnt@edgerouter.mazenet.org:
```

On the router shell

```
sudo mkdir /config/ssl/
sudo mv edgerouter.mazenet.org.cert /config/ssl/server.pem
sudo mv fullchain.cer /config/ssl/ca.pem

configure
set service gui cert-file /config/ssl/server.pem
set service gui ca-file /config/ssl/ca.pem
commit
```


If automatic renew is needed, see [https://github.com/j-c-m/ubnt-letsencrypt](https://github.com/j-c-m/ubnt-letsencrypt)