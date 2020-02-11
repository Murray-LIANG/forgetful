---
title: "Linux Bridge and VLAN"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "linux",
    "linux bridge",
    "vlan",
    "network",
]

comment: true
toc: true
auto_collapse_toc: true
---

## What is Linux Bridge?

There is a VM on the Host. And only one ethernet port on the Host,
named `eth0`.

If bind `eth0` to the VM, then the Host and other new VMs have no port
to access the internet.

So the solution is to create a bridge on the Host, name it `br0`.
Assign a virtual card to VM, name it `vnet0`.
br0 connects the vnet0 and `eth0`.

Linux bridge is a device, which can help achieve the Layer 2 package
transition, just like a physical switch or hub.

Linux bridge will transit the packages to all the devices connected to it.

![](http://7xo6kd.com1.z0.glb.clouddn.com/upload-ueditor-image-20160317-1458221779001036894.png)

## What is LAN?

Normally the devices in the same LAN are connected via Hub or Switch.
LAN is also the scope of broadcast.

VLAN: The switch with VLAN supported separates its port into several LANs
(using tag). The devices connected to different tags cannot access to
each other on Layer 2.
VLAN 的隔离是二层上的隔离，A 和 B 无法相互访问指的是二层广播包（比如 arp）
无法跨越 VLAN 的边界。但在三层上（比如IP）是可以通过路由器让 A 和 B 互通的。

![](http://7xo6kd.com1.z0.glb.clouddn.com/upload-ueditor-image-20160324-1458779560717052345.jpg)

The Host creates a virtual switch. `vnet0`, `brvlan10`, `eth0.10` are all
devices connected to the *Access Ports* of the virtual switch.
Then `eth0` is connected to the *Trunk Ports*.

All the devices connected to `brvlan10` will automatically join the VLAN 10.

`eth0` and `eth0.10` appears as a pair. One parent device `eth0` can be
mapping to more than one children devices like `eth0.10`, `eth0.20`, and etc.

A physical switch has many ports, in which any of two can be connected
to a VLAN. So the package can be transitted to another device with the
same VLAN ID. However, `eth0` separates the L2 of `eth0.10` and `eth0.20`.
And the Linux bridge `brvlan10` takes the role of transitting the packages
with the same VLAN ID.
Thus, the Linux bridge and VLAN together do the job of a physical switch.