---
title: "Keepalived"
date: 2020-09-19T10:32:42+08:00
lastmod: 2020-09-19T10:32:51+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "high availability",
    "ha",
    "keepalived",
]

comment: true
toc: true
auto_collapse_toc: true
---

# Keepalived and VRRP

VRRP: Virtual Router Redundancy **Protocol**, RFC3768 (version 2) and RFC5798 ( version 2) specify the behavior of VRRP. And Keepalived provides both a version 2 and version 3 implementation of this protocol.

VRRP uses the concept of a virutal IP address (VIP). One or more hosts participate in an election to determine the host that will control that VIP.
Only one host (the master) controls the VIP at a time. If the master fails, VRRP provides mechanisms for detecting that failure and quickly failing over to a standby host.

The first step in VRRP's operations is the election of a master to determine which server will hold the shared IP address. VRRP servers are configured with a priority value, which can be thought of like a weight. The server with the highest priority will be the owner of a VRRP address.

Once a master is established, **all other servers listen for periodic messages sent by the master to indicate that it is still alive**. The master sends out these advertisements at a regular interval. As long as the master is alive, it will service traffic for the VIP and send advertisements. If the master goes offline for some reason, then the backup server with the highest priority will take over. Similarly, **a feature called preemption can allow any server that has a higher priority to become master automatically when it comes online**.

When a master first comes online and takes over an IP address, it broadcasts a gratuitous ARP. This message informs other servers on the network of the MAC address associated with the VIP so that they can address their traffic correctly at Layer 2. It also makes VIP failover faster: hosts don't have to wait for their ARP timers to expire and can simply update their ARP tables with the correct MAC address for the host that owns the VIP.

**VRRP is a Layer 4 (Transport Layer) protocol, it's neither TCP nor UDP**. It uses IP protocol number 112.

# Keepalived vs HAProxy

Keepalived is working in layer 4 so doesn't have layer 7 knowledge at all. By using HAProxy and Keepalived together you can get benefit of having some options that HAProxy provides in layer 7 like Stickiness, Sampling and converting information, ACLs and conditions, Content switching, Stick-tables, Formated strings, HTTP rewriting and redirection, Server protection, etc.

If you only need to have a load balancer without any manipulating or any high level decision (layer 7) you can use only Keepalived and it will be faster because it works in layer 4.

# Check Script in Keepalived

Example of check script:
```bash
vrrp_script chk_myscript {
  script       "/usr/local/bin/mycheckscript.sh"
  interval 2   # check every 2 seconds
  fall 2       # require 2 failures for KO
  rise 2       # require 2 successes for OK
}

vrrp_instance MyVRRPInstance {
  state MASTER
  interface eth0
  virtual_router_id 5
  priority 200
  advert_int 1
  virtual_ipaddress {
    192.168.1.1/32 dev eth0
  }
  track_script {
    chk_myscript
  }
}
```

`mycheckscript.sh` needs to return 0 - everything is fine, 1 - something went wrong.

As soon as the `track_script` returns another code than `0` two times, the VRRP instance will change the state to `FAULT`, removes the IP `192.168.1.1` from `eth0` and stops sending multicast VRRP packets.