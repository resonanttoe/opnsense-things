# Mellanox Cards in opnsense 21.x and beyond

From: https://www.routerperformance.net/opnsense/mellanox-connecx-management-in-opnsense/

### Enable module loading

To enable it open the file /boot/loader.conf.local via CLI and add: (If you only add to /boot/loader.conf, it'll get trounced in a reboot)

```
mlx4en_load=“YES“
```

Firmware was available - http://www.mellanox.com/page/firmware_table_ConnectX3EN  
Mellanox Management Tools - http://www.mellanox.com/page/management_tools

### Set up tools
```
fetch http://www.mellanox.com/downloads/MFT/FreeBSD/mft-*.tgz
tar xvfz mft-4.11.0-103.tgz
cd mft-4.11.0-103
pkg install bash
bash install.sh
ln -s /usr/local/bin/python3.7 /usr/bin/python
```

### Check the PCI ID of your card(s):
```
$ /usr/bin/mst status
MST devices:
————
pci0:1:0:0 – MT27500 Family [ConnectX-3]
```


### Firmware stuff:
Install GCC if opnsense complains about libstdc++.so files

```
$ pkg install gcc
```

Use Flint_ext to query

```
$ /usr/bin/flint_ext -d pci0:1:0:0 q
Image type: FS2
FW Version(Running): 2.36.5000
FW Release Date: 5.9.2017
Product Version: 02.42.50.00
Rom Info: type=PXE version=3.4.752
Device ID: 4099
Description: Node Port1 Port2 Sys image
GUIDs: ffffffffffffffff ffffffffffffffff ffffffffffffffff ffffffffffffffff
MACs: 248a07f75b30 248a07f75b31
VSD:
PSID: MT_1080120023
```
 
Update firmware with bin from above:
```
$ /usr/bin/flint_ext -d pci0:1:0:0 -i fw-ConnectX3-rel-2_42_5000-MCX312A-XCB_A2-A6-FlexBoot-3.4.752.bin b

Current FW version on flash: 2.36.5000
New FW version: 2.42.5000

Burning FS2 FW image without signatures – 48%
```
 
Reboot


### LACP needs to be active on Mikrotik switches

LAG ports need to be set to Active (LACP)