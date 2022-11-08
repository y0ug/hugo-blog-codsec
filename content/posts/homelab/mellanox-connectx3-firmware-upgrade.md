---
title: "Mellanox ConnectX-3 fimware upgrade"
date: 2022-11-07
draft: false 
tags:
- homelab
- network
- mellanox
- 10G
- SFP+
- firmware
description: Note on upgrading the firmware of a Mellanox MCX3111A-XCAT ConnectX-3 a 10G network card
---

To upgrade some part of the network to 10G I bought two [Mellanox MCX3111A-XCAT ConnectX-3 from China on eBay for $35 each](https://www.ebay.com/itm/115449939717) those cards are EOL but cheap and well-supported.


This is some note on how I updated the firmware. I'm using them with a DAC cable, so nothing fancy.

You can use the official tools

```
mkdir ConnectX-Firmware 
cd ConnectX-Firmware
wget https://www.mellanox.com/downloads/MFT/mft-4.21.0-99-x86_64-deb.tgz
tar xvf mft-4.21.0-99-x86_64-deb.tgz
cd  mft-4.21.0-99-x86_64-deb
sudo ./install.sh
```

Or from apt, in this case `mlxconfig` become `mstconfig` and `flint` become `mstflint`

```
sudo apt install mstflint
```

If you used the official package, you need to start the module with

```
$ sudo mst start
Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
Loading MST PCI configuration module - Success
Create devices
-W- Missing "lsusb" command, skipping MTUSB devices detection
```

You can get the device path by issuing `sudo mst status`

```
$ sudo mst status
MST modules:
------------
    MST PCI module loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4099_pci_cr0          - PCI direct access.
                                   domain:bus:dev.fn=0000:01:00.0 bar=0xf7d00000 size=0x100000
                                   Chip revision is: 01
/dev/mst/mt4099_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:01:00.0 addr.reg=88 data.reg=92 cr_bar.gw_offset=-1
                                   Chip revision is: 01
```

You can query the configuration with `sudo mlxconfig -d /dev/mst/mt4099_pciconf0 query`

```
$ sudo mlxconfig -d /dev/mst/mt4099_pciconf0  query

Device #1:
----------

Device type:    ConnectX3
Device:         /dev/mst/mt4099_pciconf0

Configurations:                              Next Boot
-E- Failed to query device current configuration
```

It's not working 😔 for me. I'm not the only one,  to have this issue. If I need to enable *SRV-IO*  I will have to update the firmware .ini files and flash it with *flint*. More information on this post [https://github.com/Mellanox/mstflint/issues/590](https://github.com/Mellanox/mstflint/issues/590)

The tools to flash the firmware is *flint* You can get the current firmware information with `sudo flint -d /dev/mst/mt4099_pci_cr0 query`

```
$ sudo flint -d  /dev/mst/mt4099_pci_cr0 query full
Image type:            FS2
FW Version:            2.35.5100
FW Release Date:       6.9.2015
MIC Version:           1.5.0
Config Sectors:        1
Product Version:       02.35.51.00
Rom Info:              type=PXE version=3.4.648
Device ID:             4099
Description:           Node             Port1            Port2            Sys image
GUIDs:                 ffffffffffffffff ffffffffffffffff ffffffffffffffff ffffffffffffffff
MACs:                                       e41d2ddd7850     e41d2ddd7851
VSD:
PSID:                  MT_1170110023
```

You can query a downloaded firmware to check the version 

```
$ sudo flint -i ./fw-ConnectX3-rel-2_42_5000-MCX311A-XCA_Ax-FlexBoot-3.4.752.bin query full
Image type:            FS2
FW Version:            2.42.5000
FW Release Date:       5.9.2017
MIC Version:           2.0.0
Config Sectors:        1
PRS Name:              cx3-1_MCX311A.prs
Product Version:       02.42.50.00
Rom Info:              type=PXE version=3.4.752
Device ID:             4099
Description:           Node             Port1            Port2            Sys image
GUIDs:                 0002c9000100d050 0002c9000100d051 0002c9000100d052 0002c9000100d050
MACs:                                       0002c9000001     0002c9000002
VSD:                   n/a
PSID:                  MT_1170110023
```

You want the **PSID** to be the same on the downloaded firmware and device.

Furthermore, you can verify the downloaded firmware

```
$ sudo flint -i ./fw-ConnectX3-rel-2_42_5000-MCX311A-XCA_Ax-FlexBoot-3.4.752.bin verify

     FS2 non failsafe image:

     /0x00000038-0x0000065b (0x000624)/ (BOOT2) - OK
     /0x0000065c-0x0000297f (0x002324)/ (BOOT2) - OK
     /0x00002980-0x00003923 (0x000fa4)/ (Configuration) - OK
     /0x00003924-0x0001d14b (0x019828)/ (ROM) - OK
     /0x0001d14c-0x0001d18f (0x000044)/ (GUID) - OK
     /0x0001d190-0x0001d313 (0x000184)/ (Image Info) - OK
     /0x0001d314-0x0002a6ab (0x00d398)/ (DDR) - OK
     /0x0002a6ac-0x0002b6ff (0x001054)/ (DDR) - OK
     /0x0002b700-0x0002bacf (0x0003d0)/ (DDR) - OK
     /0x0002bad0-0x00069aa7 (0x03dfd8)/ (DDR) - OK
     /0x00069aa8-0x0006e92b (0x004e84)/ (DDR) - OK
     /0x0006e92c-0x00072dff (0x0044d4)/ (DDR) - OK
     /0x00072e00-0x000738f7 (0x000af8)/ (DDR) - OK
     /0x000738f8-0x000a29eb (0x02f0f4)/ (DDR) - OK
     /0x000a29ec-0x000a6597 (0x003bac)/ (DDR) - OK
     /0x000a6598-0x000bb48b (0x014ef4)/ (DDR) - OK
     /0x000bb48c-0x000bb593 (0x000108)/ (DDR) - OK
     /0x000bb594-0x000c0617 (0x005084)/ (DDR) - OK
     /0x000c0618-0x000c1e13 (0x0017fc)/ (Configuration) - OK
     /0x000c1e14-0x000c1e87 (0x000074)/ (Jump addresses) - OK
     /0x000c1e88-0x000c250b (0x000684)/ (FW Configuration) - OK
     /0x00000000-0x000c250b (0x0c250c)/ (Full Image) - OK

-I- FW image verification succeeded. Image is bootable.
```

You can back up the current firmware and the configuration before flashing it with

```
export BACKUP_NAME=fw-ConnectX3-backup

# backup the device configuration
sudo flint -d  /dev/mst/mt4099_pci_cr0 dc > ${BACKUP_NAME}.ini

# backup the firmware
sudo flint -d  /dev/mst/mt4099_pci_cr0 ri ${BACKUP_NAME}.mlx

# backup the ROM
sudo flint -d  /dev/mst/mt4099_pci_cr0 rrom  ${BACKUP_NAME}.dump
```

To flash a new firmware 

```
$ sudo flint -d /dev/mst/mt4099_pci_cr0 -i ./fw-ConnectX3-rel-2_42_5000-MCX311A-XCA_Ax-FlexBoot-3.4.752.bin burn

    Current FW version on flash:  2.35.5100
    New FW version:               2.42.5000

Burn process will not be failsafe. No checks will be performed.
ALL flash, including the Invariant Sector will be overwritten.
If this process fails, computer may remain in an inoperable state.

 Do you want to continue ? (y/n) [n] : y
Burning FS2 FW image without signatures - OK
Restoring signature                     - OK
```

After updating the firmware, you can verify it with *query*. If you didn't reboot, you will see that it's still running on *2.35.5100* and will switch to *2.42.5000* on next boot.

```
$ sudo flint -d  /dev/mst/mt4099_pci_cr0 query full
Image type:            FS2
FW Version:            2.42.5000
FW Version(Running):   2.35.5100
FW Release Date:       5.9.2017
MIC Version:           2.0.0
Config Sectors:        1
PRS Name:              cx3-1_MCX311A.prs
Product Version:       02.42.50.00
Rom Info:              type=PXE version=3.4.752
Device ID:             4099
Description:           Node             Port1            Port2            Sys image
GUIDs:                 ffffffffffffffff ffffffffffffffff ffffffffffffffff ffffffffffffffff
MACs:                                       e41d2ddd7850     e41d2ddd7851
VSD:
PSID:                  MT_1170110023
```

You can check version with *ethtool* too.

```
$ sudo ethtool -i enp1s0
driver: mlx4_en
version: 4.0-0
firmware-version: 2.42.5000
expansion-rom-version:
bus-info: 0000:01:00.0
supports-statistics: yes
supports-test: yes
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: yes

$ sudo ethtool enp1s0
Settings for enp1s0:
        Supported ports: [ FIBRE ]
        Supported link modes:   1000baseX/Full
                                10000baseCR/Full
                                10000baseSR/Full
        Supported pause frame use: Symmetric Receive-only
        Supports auto-negotiation: No
        Supported FEC modes: Not reported
        Advertised link modes:  1000baseX/Full
                                10000baseCR/Full
                                10000baseSR/Full
        Advertised pause frame use: Symmetric
        Advertised auto-negotiation: No
        Advertised FEC modes: Not reported
        Speed: 10000Mb/s
        Duplex: Full
        Auto-negotiation: off
        Port: Direct Attach Copper
        PHYAD: 0
        Transceiver: internal
        Supports Wake-on: d
        Wake-on: d
        Current message level: 0x00000014 (20)
                               link ifdown
        Link detected: yes
```
