---
title: "OpenStack Introduction"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
]

comment: true
toc: true
auto_collapse_toc: true
---
## Main Services

- Nova
- Neutron
- Glance
- Cinder
- Keystone
- Swift (Optional)

## Nodes

### Control Node
Used to manage the OpenStack env.

Keystone, Glance, Horizon services run on it.

Some Nova management module, Neutron management module,
SQL database, Message Queue, and NTP services run on it.

### Network Node
Provides network including L2 and L3, like route, NAT, DHCP, and etc.

Main Neutron services run on it.

### Storage Node
Provides storage, like Block storage and Object storage.

Main Cinder and Swift services run on it.

### Compute Node
Acts as Hypervisor of VMs.

Main Nova services and Neutron Agent service run on it.

Here is an example of devstack, putting almost all services on one node.

![](/forgetful/images/openstack-intro-devstack_layout.jpg)

