# Init7 IPTV/igmpproxy setup

From: https://forum.opnsense.org/index.php?topic=17865.0

## Install plugin

To get Multicast to work on OPNsense we are going to use os-igmp-proxy.

## Configure IGMP Proxy

Interface: WAN
Type: Upstream Interface
Threshold: 1
Option 1: Networks (single entry): 77.109.129.0/25
Option 2: Networks (multiple entries, single hosts):

```
77.109.129.16/32
77.109.129.17/32
77.109.129.18/32
77.109.129.19/32
```

Interface: LAN
Description: LAN_DOWN
Threshold: 1
Networks: Enter your local network here (e.g. 192.168.1.0/24)

## Firewall Rules

All rules have to allow options in the advanced settings

### LAN

First we have to enable allow options on the default LAN rule Default allow LAN to any rule - In Advanced options

### WAN

Protocol: IGMP
Source: WAN net
Destination: Single host or Network -> 224.0.0.0/4

Protocol: PIM
Source: WAN net
Destination: Single host or Network -> 224.0.0.0/4

Protocol: UDP
Source: Single host or Network -> 77.109.129.0/25
Destination: Single host or Network -> 239.0.0.0/8
Destination port range: Other -> from: 5000 -> to: 5000

### Option B (multiple rules, single host):

Protocol: UDP
Source: Single host or Network -> 77.109.129.16/32
Destination: Single host or Network -> 239.0.0.0/8
Destination port range: Other -> from: 5000 -> to: 5000
Description: init7: Allow Multicast Traffic

Clone and change source to Change source to 77.109.129.17, 77.109.129.18, 77.109.129.19

### Floating

Every Multicast IP address resolves into a predefined Multicast MAC address
Here are some information about it including a calculator: http://www.dqnetworks.ie/toolsinfo.d/multicastaddressing.html

Interface: WAN
Direction: out
Protocol: IGMP
Source: WAN address
Destination: Single host or Network -> 224.0.0.0/4
