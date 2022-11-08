---
title: "Mikrotik Note"
date: 2022-11-08T19:52:01+01:00
draft: true
---

Some note on how I set up a LACP between `CRS305-1G-4S+` and the Netgear switch.

I want to use the 1G RJ45 port and the first SFP+ with an SFP RJ45 Copper Transceiver to connect the switch over 1G.

The main issue with this switch is that I will lose the connection, trying to remove the interconnection port from the bridge before being able to create the bond. I had to KVM to one VM that I used *Winbox* with the mac address. A direct COM port would have really helped.

Safe mode press `ctrl-x` to enter and exit.

To set up the bond, I had to remove two interfaces from the current bridge.

```
/interface bridge 
port print
port remove number=1,2
```

Then we need to create the bond with the two interfaces

```
/interface bonding
add mode=802.3ad name=bond1 slaves=ether1,sfp-sfpplus1 transmit-hash-policy=layer-2-and-3
```

Adding the bond to the bridge.

```
/interface bridge port
add bridge=bridge interface=bond1
```
