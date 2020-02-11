---
title: "OpenStack Neutron"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "neutron",
]

comment: true
toc: true
auto_collapse_toc: true
---

Software Defined Network

## Functionality
### L2 switching
Virtual switch: support `Linux Bridge` and `Open vSwitch`.
Based on the `Linux Bridge` and `OVS`, Neutron supports VLAN. Besides,
Tunneling overlay network like VxLan and GRE are supported.
### L3 routing
Virtual router: leverages IP forwarding and iptables to do routing and NAT.
### Load balancing
### Firewall

## Key points
### Network
- Local: Instance can only access to the instances on the same node.
- Flat: no VLAN tagging. Instance could access to the instances in the same network, regardless the node.
- VLAN: Instances with the same VLAN tag can access to each other no matter they are on the same nodes or not. Otherwise, a router is needed.
- VxLan: overlay network based on tunneling. Each VxLan network has unique `Segmentation ID` (aka VNI). In VxLan packets are transmitted by packaging into UDP packets via VNI. The packets orignal in L2 is enpacked into L3, it breaks the limitation of VLAN.
- GRE: like VxLan. The main difference is that it uses IP packets instead UDP.

Any two `Networks` here are isolated from the view of L2.
Each `Network` should belong to one `Project`/`Tenant`. But one `Project/Tenant` could have more
than one `Network`.

### Subnet
Each two `Subnet` in one `Network` cannot have the same CIDR.
But two `Subnet` in two `Network` could.
If two instances in two `Network` have the same IP. The Neutron router can
take care of it. How? Use `Linux Network Namespace`. The routers in different
Linux network namespace have isolated route tables.

### Port
Like the ports of the virtual switch. The virtual interface (`VIF`) of instances can
be bound to several `Port`s.

### The mapping between
Project 1 : n Network 1 : n Subnet 1 : n Port 1 : 1 VIF n : 1 Instance

## Arch
### Neutron server
Entry of Neutron request.
### Plugin
### Agent
### Network provider
Linux bridge or Open vSwitch
### Queue
### Database

`Plugin`, `Agent`, `Network provider` are a set.
For example, if the network provider is `Linux bridge`, then the plugin
and the agent should be the one for `Linux bridge`.

There are `Core Plugin` and `Service Plugin`. `Core Plugin` is used to
manage the Network, Subnet and Port. While `Service Plugin` is for routing,
firewall, load balance.

## Deploy

### Control + Compute
Control:
- neutron server (including Core Plugin and Service Plugin inside),
- neutron-plugin-agent
- neutron-dhcp-agent
- neutron-metadata-agent
- neutron-l3-agent
- neutron-lbaas-agent

Compute:
- neutron-plugin-agent

### Control + Network + Compute
Control:
- neutron server (including Core Plugin and Service Plugin inside),

Network:
- neutron-plugin-agent
- neutron-dhcp-agent
- neutron-metadata-agent
- neutron-l3-agent
- neutron-lbaas-agent

Compute:
- neutron-plugin-agent

## ML2
Modular Layer 2 is a core plugin.
Before ML2, only one core plugin can be used. So it limits that
every node would be configured to use the same core plugin agent,
like Linux bridge or Open vSwitch.

ML2 is used to support the hybrid agent. When using ML2 core plugin,
the node could use any agent.

![](/forgetful/images/openstack-neutron-services-plugins.jpg)


## Real-world deployment
![](/forgetful/images/openstack-neutron-realworld-deployment.jpg)
