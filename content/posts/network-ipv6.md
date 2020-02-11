---
title: "IPv6"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
   "network",
   "ipv6",
]

comment: true
toc: true
auto_collapse_toc: true
---

IPv4 addresses are 32 bits long and IPv6 addresses are 128 bits long.

## Classifying IPv6 Addresses

IPv6 has three types of addresses:
- Unicast: An IPv6 unicast address is used to identify **a single interface**. Packets sent to a unicast address are delivered to that specific interface.
- Anycast: IPv6 anycast addresses identify groups of interfaces, which typically belong to different nodes. Packets destined to an anycast address are sent to the **nearest interface in the group**, as determined by the active routing protocols.
- Multicast: An IPv6 multicast address also identifies a group of interfaces, again typically belonging to different nodes. Packets sent to a multicast address are delivered to **all interfaces in the group**.

NOTE: there is **NO** broadcast addresses in IPv6. The functions served by broadcast addresses in IPv4 are provided by multicast addresses in IPv6.

```
Address Type                  Binary Prefix                       Hex Prefix
Unspecified                   0000...0 (128 bits)                 ::/128
Loopback                      000...01 (128 bits)                 ::1/128
IPv4 Mapped                   00...01111111111111111 (96 bits)    ::FFFF/96
Multicast                     11111111                            FF00::/8
Link-Local Unicast            1111111010                          FE80::/10
Unique Local Unicast (ULA)    1111110                             FC00::/7
Global Unicast	(everything else)
Anycast addresses are taken from the global unicast pool. Anycast and unicast addresses cannot be distinguished based on format.
```

### Multicast Addresses

https://chrisgrundemann.com/index.php/2012/introducing-ipv6-classifying-ipv6-addresses/


### Global Unicast Addresses

https://chrisgrundemann.com/index.php/2012/introducing-ipv6-classifying-ipv6-addresses/

![](/forgetful/images/ipv6-global-unicast-ipv6-address-format.png)

- Global Routing Prefix: The prefix assigned to a site. Typically this is hierarchically structured as it passes from IANA (Internet Assigned Numbers Authority) to the RIR (Regional Internet Registry) to an ISP (Internet Service Provider) or LIR (Local Internet Registry) and then to a customer or a specific customer location. In each of these transactions, a smaller prefix is assigned downstream - creating the hierarchy.
- Subnet ID: The prefix assigned to a particular link or LAN within the site. In the case of a **/48 being assigned to a site**, there are **16 bits available for Subnet IDs**: this allows a maximum of **65536 /64 subnet** prefixes a that location!
- Interface ID: must be unique within a subnet prefix and are used to identify interfaces on a link. Because of this, /64 prefixes are the smallest common subnet you will use in IPv6.

## Special IPv6 Addresses

- Unspecified address (::/128): This all-zeros address refers to the host itself when the host does not know it's own address. Tha unspecified address is typically used in the source field by a device seeking to have its IPv6 address assigned.
- Loopback address (::1/128): IPv6 has a single address for the loopback function, instead of a whole block as is the case in IPv4 (127.0.0.1).
- IPv4-mapped addresses (::FFFF/96): A /96 prefix leaves 32 bits, exactly enough to hold an embedded IPv4 address. IPv4-mapped IPv6 addresses are used to represent an IPv4 node's address as an IPv6 address. This address type was defined to help with the transition from IPv4 to IPv6.
- Link-local unicast addresses (FE80::/10): As the name implies, Link-local addresses are unicast addresses to be used on a single link. Packets with a Link-local source or destination address will not be forwarded to other links. These addresses are used for neighbor discovery, automatic address configuration and in circumstances when no routers are present.
- Unique local unicast addresses (FC00::/7): Commonly known as [ULA](http://en.wikipedia.org/wiki/Unique_local_address), this group of addresses is for use locally, within a site or group of sites. Although globally unique, these addresses are not routable on the global Internet. This author looks at ULA as a kind of upgraded rfc1918 (private) address space for IPv6. It is equivalent of private networks in IPv4, 10.0.0.0/8, 172.16.0.0/12 and 192.168.0.0/16.

## Practice
1. You could generate a ULA (like private addresses 192.168 in IPv4) on the website: https://simpledns.com/private-ipv6
2. `fd52:04f1:7987:08a1::/64` (`fd52:04f1:7987:08a1:xxxx:xxxx:xxxx:xxxx`) is generated in step #1.
3. Convert the MAC address `00:1e:67:6d:ee:d0` of `ubuntu-server1 eth1`to EUI-64 on the website: https://www.vultr.com/tools/mac-converter (`ubuntu-server1` is the ubuntu gateway of OpenStack env)
4. Copy the `Contained EUI-48 (U/L)` box output: `02:1e:67:ff:fe:6d:ee:d0`. Convert it to the interface id for IPv6 `021e:67ff:fe6d:eed0`.
5. Combining with the prefix, we get the IPv6 address: `fd52:04f1:7987:08a1:021e:67ff:fe6d:eed0`.
6. Configure the IPv6 address to `ubuntu-server1 eth1`: `ip -f inet6 addr add fd52:04f1:7987:08a1:021e:67ff:fe6d:eed0/64 dev eth1`.
7. Now you have two IPv6 addresses configured, including the one configured on `eth0` by IT:
    ```console
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:1e:67:6d:ee:cf brd ff:ff:ff:ff:ff:ff
    ......
    inet6 2620:0:170:1d30:21e:67ff:fe6d:eecf/64 scope global dynamic
       valid_lft 2591719sec preferred_lft 604519sec
    inet6 fe80::21e:67ff:fe6d:eecf/64 scope link
       valid_lft forever preferred_lft forever

    4: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:1e:67:6d:ee:d0 brd ff:ff:ff:ff:ff:ff
    ......
    inet6 fd52:4f1:7987:8a1:21e:67ff:fe6d:eed0/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::21e:67ff:fe6d:eed0/64 scope link
       valid_lft forever preferred_lft forever
    ``` 
    And routes of IPv6 are configured automatically:
    ```console
    root@ubuntu-server1:~# ip -f inet6 route show
    2620:0:170:1d30::/64 dev eth0  proto kernel  metric 256  expires 2591859sec
    fd52:4f1:7987:8a1::/64 dev eth1  proto kernel  metric 256
    fe80::/64 dev eth2  proto kernel  metric 256
    fe80::/64 dev eth0  proto kernel  metric 256
    fe80::/64 dev eth1  proto kernel  metric 256
    fe80::/64 dev eth4  proto kernel  metric 256
    default via fe80::226:98ff:fe04:b042 dev eth0  proto ra  metric 1024  expires 1659sec
    ```
8. To test the connectivity, switch to another server like `ubuntu-client1`, configure an IPv6 address: `sudo ip -f inet6 addr add fd52:04f1:7987:08a1:021e:67ff:fe0c:328d/64 dev enp1s0f1`
   
    NOTE: `enp1s0f1` is in the same network with `ubuntu-server1 eth1`, and `fd52:04f1:7987:08a1:021e:67ff:fe0c:328d` is generated the same way as above.
9.  We can ping `ubuntu-server1` and `ob-h1132`'s IPv6 mgmt interface via IPv6.
    ```console
    $ ping6 fd52:04f1:7987:08a1:021e:67ff:fe6d:eed0
    PING fd52:04f1:7987:08a1:021e:67ff:fe6d:eed0(fd52:4f1:7987:8a1:21e:67ff:fe6d:eed0) 56 data bytes
    64 bytes from fd52:4f1:7987:8a1:21e:67ff:fe6d:eed0: icmp_seq=1 ttl=64 time=0.403 ms
    --- fd52:04f1:7987:08a1:021e:67ff:fe6d:eed0 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 999ms
    rtt min/avg/max/mdev = 0.203/0.303/0.403/0.100 ms
    
    $ ping6 2620:0:170:1d30::1
    PING 2620:0:170:1d30::1(2620:0:170:1d30::1) 56 data bytes
    64 bytes from 2620:0:170:1d30::1: icmp_seq=3 ttl=64 time=0.783 ms
    --- 2620:0:170:1d30::1 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 1998ms
    rtt min/avg/max/mdev = 0.776/0.811/0.874/0.044 ms

    $ ping6 2620:0:170:1d65:260:1600:47e0:1da
    PING 2620:0:170:1d65:260:1600:47e0:1da(2620:0:170:1d65:260:1600:47e0:1da) 56 data bytes
    64 bytes from 2620:0:170:1d65:260:1600:47e0:1da: icmp_seq=2 ttl=63 time=0.249 ms
    --- 2620:0:170:1d65:260:1600:47e0:1da ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 999ms
    rtt min/avg/max/mdev = 0.249/0.513/0.777/0.264 ms
    ```
10. To persist after reboot, modify the file `/etc/network/interfaces`
    ```bash
    # example of ubuntu-server1
    auto eth1
    iface eth1 inet6 static
            pre-up modprobe ipv6  # just to make sure the ipv6 module is loaded
            address fd52:04f1:7987:08a1:021e:67ff:fe6d:eed0
            # the next line turns off ipv6 autoconfig
            up echo 0 > /proc/sys/net/ipv6/conf/all/autoconf
            netmask 64
    ```