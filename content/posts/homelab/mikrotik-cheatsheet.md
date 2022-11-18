---
title: "Mikrotik cheatsheet"
date: 2022-11-08T19:52:01+01:00
draft: false
tags:
- homelab
- mikrotik
- cheatsheet
- note
- configuration
---

## Configuration

Safe mode press `ctrl-x` to enter and exit. This allows to revert to the previous state if you cancel or get disconnected.

### System info

```
[admin@switch-10g] > /system resource print
                   uptime: 6d13h36m59s
                  version: 6.49.7 (stable)
               build-time: Oct/11/2022 14:37:24
         factory-software: 6.46.8
              free-memory: 484.2MiB
             total-memory: 512.0MiB
                      cpu: ARMv7
                cpu-count: 1
            cpu-frequency: 800MHz
                 cpu-load: 0%
           free-hdd-space: 2256.0KiB
          total-hdd-space: 16.0MiB
  write-sect-since-reboot: 4580
         write-sect-total: 4580
               bad-blocks: 0%
        architecture-name: arm
               board-name: CRS305-1G-4S+
                 platform: MikroTik
```

### Backup/restore

By default, the encryption password is the user password (start at *v6.43*)

```
/system backup save

# no encryption
/system backup save dont-encrypt=yes

# custom password and name
/system backup save name=myconfig.backup password=MyPassw0rd^
```

List files

```
/file print
```

```
/system backup load name=FileName
```

## Hardening

### SSH

Copying your SSH key on the device with scp

```sh
scp ~/.ssh/.id_rsa.pub admin@switch:
```

On the device

```sh
# import public key for the user
/user ssh-keys import public-key-file=id_rsa.pub user=admin

# remove the file
/file remove id_rsa.pub

# hardening SSH
/ip ssh set strong-crypto=yes
/ip ssh set always-allow-password-login=no
```


### SSL Configuration 

To get SSL certificate issue for a device on my local network without exposing service to the internet, I'm using the DNS verification for ZeroSSL/letsencrypt. For the domain *codsec.com* I'm using *Cloudflare* for DNS provider, so I can use their API with various tools. On my local DNS, I have a zone for *home.codsec.com*, so those domains are never exposed online.


The script used [acme.sh](https://github.com/acmesh-official/acme.sh) 

```bash
export CF_Account_ID=
export CF_Token=
DOMAIN=$1
echo Issuing certificate for $DOMAIN
CERT_PATH=$HOME/.acme.sh/$DOMAIN/
acme.sh --issue --dns dns_cf -d $DOMAIN
cat $CERT_PATH/$DOMAIN.cer $CERT_PATH/$DOMAIN.key > $CERT_PATH/$DOMAIN.cert
```

```
$ Documents/gen_cert.sh switch-10g.home.codsec.com
Issuing certificate for switch-10g.home.codsec.com
[Fri Nov 18 06:30:23 UTC 2022] Using CA: https://acme.zerossl.com/v2/DV90
[Fri Nov 18 06:30:23 UTC 2022] Creating domain key
[Fri Nov 18 06:30:23 UTC 2022] The domain key is here: /acme.sh/switch-10g.home.codsec.com/switch-10g.home.codsec.com.key
[Fri Nov 18 06:30:23 UTC 2022] Single domain='switch-10g.home.codsec.com'
[Fri Nov 18 06:30:23 UTC 2022] Getting domain auth token for each domain
[Fri Nov 18 06:30:32 UTC 2022] Getting webroot for domain='switch-10g.home.codsec.com'
[Fri Nov 18 06:30:32 UTC 2022] Adding txt value: 65Igo7dHQOhwzBsofoPXcoDoWtZmVK16-aq2Rlpnbjs for domain:  _acme-challenge.switch-10g.home.codsec.com
[Fri Nov 18 06:30:39 UTC 2022] Adding record
[Fri Nov 18 06:30:40 UTC 2022] Added, OK
[Fri Nov 18 06:30:40 UTC 2022] The txt record is added: Success.
[Fri Nov 18 06:30:40 UTC 2022] Let's check each DNS record now. Sleep 20 seconds first.
[Fri Nov 18 06:31:01 UTC 2022] You can use '--dnssleep' to disable public dns checks.
[Fri Nov 18 06:31:01 UTC 2022] See: https://github.com/acmesh-official/acme.sh/wiki/dnscheck
[Fri Nov 18 06:31:01 UTC 2022] Checking switch-10g.home.codsec.com for _acme-challenge.switch-10g.home.codsec.com
[Fri Nov 18 06:31:02 UTC 2022] Domain switch-10g.home.codsec.com '_acme-challenge.switch-10g.home.codsec.com' success.
[Fri Nov 18 06:31:02 UTC 2022] All success, let's return
[Fri Nov 18 06:31:02 UTC 2022] Verifying: switch-10g.home.codsec.com
[Fri Nov 18 06:31:04 UTC 2022] Processing, The CA is processing your order, please just wait. (1/30)
[Fri Nov 18 06:31:10 UTC 2022] Processing, The CA is processing your order, please just wait. (2/30)
[Fri Nov 18 06:31:15 UTC 2022] Processing, The CA is processing your order, please just wait. (3/30)
[Fri Nov 18 06:31:21 UTC 2022] Success
[Fri Nov 18 06:31:21 UTC 2022] Removing DNS records.
[Fri Nov 18 06:31:21 UTC 2022] Removing txt: 65Igo7dHQOhwzBsofoPXcoDoWtZmVK16-aq2Rlpnbjs for domain: _acme-challenge.switch-10g.home.codsec.com
[Fri Nov 18 06:31:28 UTC 2022] Removed: Success
[Fri Nov 18 06:31:28 UTC 2022] Verify finished, start to sign.
[Fri Nov 18 06:31:28 UTC 2022] Lets finalize the order.
[Fri Nov 18 06:31:28 UTC 2022] Le_OrderFinalize='https://acme.zerossl.com/v2/DV90/order/DOACnCIAbVYX8qZjjK9iRA/finalize'
[Fri Nov 18 06:31:32 UTC 2022] Order status is processing, lets sleep and retry.
[Fri Nov 18 06:31:32 UTC 2022] Retry after: 15
[Fri Nov 18 06:31:48 UTC 2022] Polling order status: https://acme.zerossl.com/v2/DV90/order/DOACnCIAbVYX8qZjjK9iRA
[Fri Nov 18 06:31:51 UTC 2022] Downloading cert.
[Fri Nov 18 06:31:51 UTC 2022] Le_LinkCert='https://acme.zerossl.com/v2/DV90/cert/-v7m8p9iZRNNuGcv2SB75A'
[Fri Nov 18 06:31:53 UTC 2022] Cert success.
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
[Fri Nov 18 06:31:53 UTC 2022] Your cert is in: /acme.sh/switch-10g.home.codsec.com/switch-10g.home.codsec.com.cer
[Fri Nov 18 06:31:53 UTC 2022] Your cert key is in: /acme.sh/switch-10g.home.codsec.com/switch-10g.home.codsec.com.key
[Fri Nov 18 06:31:53 UTC 2022] The intermediate CA cert is in: /acme.sh/switch-10g.home.codsec.com/ca.cer
[Fri Nov 18 06:31:53 UTC 2022] And the full chain certs is there: /acme.sh/switch-10g.home.codsec.com/fullchain.cer

$ scp ~/.acme.sh/switch-10g.home.codsec.com/switch-10g.home.codsec.com.cert switch-10g.mngt.mazenet.arpa:
$ scp ~/.acme.sh/switch-10g.home.codsec.com/fullchain.cer switch-10g.mngt.mazenet.arpa:
```

Importing into the device

```
[admin@switch-10g] > /certificate import file-name=fullchain.cer
passphrase:
     certificates-imported: 3
     private-keys-imported: 0
            files-imported: 1
       decryption-failures: 0
  keys-with-no-certificate: 0

[admin@switch-10g] > /certificate import file-name=switch-10g.home.codsec.com.cert
passphrase:
     certificates-imported: 0
     private-keys-imported: 1
            files-imported: 1
       decryption-failures: 0
  keys-with-no-certificate: 0

[admin@switch-10g] > /file print
 # NAME                                               TYPE                                                    SIZE CREATION-TIME
 0 id_rsa.pub                                         ssh key                                                  392 jan/18/1970 20:01:25
 1 switch-10g.home.codsec.com.cert                    .cert file                                              3996 jan/19/1970 13:16:35
 2 fullchain.cer                                      .cer file                                             6.6KiB jan/19/1970 13:16:54
 3 flash                                              disk                                                         jan/18/1970 19:59:23
 4 flash/skins                                        directory                                                    jan/01/1970 00:00:27
 5 flash/auto-before-reset.backup                     backup                                               14.2KiB jan/01/1970 00:00:30
 [admin@switch-10g] > /ip service set www-ssl certificate=fullchain.cer_0
 [admin@switch-10g] > /ip service enable www-ssl
 ```

Checking the certificate

```
$ curl -vI https://switch-10g.home.codsec.com/
*   Trying 192.168.10.253:443...
* Connected to switch-10g.home.codsec.com (192.168.10.253) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: /etc/ssl/certs
...
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=switch-10g.home.codsec.com
*  start date: Nov 18 00:00:00 2022 GMT
*  expire date: Feb 16 23:59:59 2023 GMT
*  subjectAltName: host "switch-10g.home.codsec.com" matched cert's "switch-10g.home.codsec.com"
*  issuer: C=AT; O=ZeroSSL; CN=ZeroSSL RSA Domain Secure Site CA
*  SSL certificate verify ok.
* TLSv1.2 (OUT), TLS header, Supplemental data (23):
> HEAD / HTTP/1.1
> Host: switch-10g.home.codsec.com
> User-Agent: curl/7.81.0
> Accept: */*
>
* TLSv1.2 (IN), TLS header, Supplemental data (23):
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
```


### Services

Disabling various tool

```sh
# Disable RoMON
/tool romon set enabled=no

# Disable the bandwith server
/tool bandwidth-server set enabled=no

# Disable MikroTik Neighbor discovery protocol 
/ip neighbor discovery-settings set discover-interface-list=none 

# Ensure the DNS server is not running
/ip dns set allow-remote-requests=no
```

Disabling unnecessary services

```sh
# list current service
/ip service print

# Ensure the socks server other various service are disable
/ip socks set enabled=no
/ip proxy set enabled=no
/ip upnp set enabled=no
/ip cloud set ddns-enabled=no update-time=no

# Disable telnet and ftp
/ip service disable telnet,ftp

# Disable none SSL
/ip service disable www,api,api-ssl

/ip service print
```

Disabling the MAC-access on all interface (could maybe be enable on management network). This is usefull mostly to configure the bridge.

```sh
# Disable mac-telnet services,
/tool mac-server set allowed-interface-list=none

# Disable mac-winbox services
/tool mac-server mac-winbox set allowed-interface-list=none

# Disable mac-ping service,
/tool mac-server ping set enabled=no
```

## LCAP

Some note on how I set up a LACP between `CRS305-1G-4S+` and the Netgear switch.

I want to use the 1G RJ45 port and the first SFP+ with an SFP RJ45 Copper Transceiver to connect the switch over 1G.

The main issue with this switch is that I will lose the connection, trying to remove the interconnection port from the bridge before being able to create the bond. I had to KVM to one VM that I used *Winbox* with the mac address. A direct COM port would have really helped.

To set up the bond, I had to remove two interfaces from the current bridge.

```sh
/interface bridge 
port print
port remove number=1,2
```

Then we need to create the bond with the two interfaces

```sh
/interface bonding
add mode=802.3ad name=bond1 slaves=ether1,sfp-sfpplus1 transmit-hash-policy=layer-2-and-3
```

Adding the bond to the bridge.

```sh
/interface bridge port
add bridge=bridge interface=bond1
```


```
[admin@switch-10g] > /interface print
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  RS ether1                              ether            1500  1592      10218 2C:C8:1B:A9:4D:60
 1  RS sfp-sfpplus1                        ether            1500  1592      10218 2C:C8:1B:A9:4D:61
 2  RS sfp-sfpplus2                        ether            1500  1592      10218 2C:C8:1B:A9:4D:62
 3   S sfp-sfpplus3                        ether            1500  1592      10218 2C:C8:1B:A9:4D:63
 4   S sfp-sfpplus4                        ether            1500  1592      10218 2C:C8:1B:A9:4D:64
 5  RS bond1                               bond             1500  1592            2C:C8:1B:A9:4D:60
 6  R  ;;; defconf
       bridge                              bridge           1500  1592            2C:C8:1B:A9:4D:60
       
[admin@switch-10g] > /interface bonding print
Flags: X - disabled, R - running
 0  R name="bond1" mtu=1500 mac-address=2C:C8:1B:A9:4D:60 arp=enabled arp-timeout=auto slaves=ether1,sfp-sfpplus1 mode=802.3ad
      primary=none link-monitoring=mii arp-interval=100ms arp-ip-targets="" mii-interval=100ms down-delay=0ms up-delay=0ms
      lacp-rate=30secs transmit-hash-policy=layer-2-and-3 min-links=0
```