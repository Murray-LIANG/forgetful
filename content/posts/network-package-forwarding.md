---
title: "Network Packages Forwarding"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "network",
    "nat",
    "arp",
]

comment: true
toc: true
auto_collapse_toc: true
---

## NAT
Network Address Translation: 在一个网络内部，自定义合法的ip地址。内网各主机通过内网通讯；与外网通过NAT转换，变成外网合法ip。这样，将内网与外网隔离，各个网络有自己的ip，既可以重叠，又可以通过少数几个ip与外网通讯，在ip大量缺乏的现代，节省了很多。

## Packages Forwarding
1. PC1要访问 www.google.com，需要先知道对应IP地址。域名只起助记作用，互联网访问通过IP进行。

2. 于是，PC1需要像DNS请求，查找 www.google.com 对应的ip，即发送dns请求：PC1查找dns，发现不在同一个网络，不同网段需要网关转发。但是，PC1需要先发送给网关，就需要先知道网关ip。网关用于连接不同网络，并且有自己的IP，PC1需要知道网关ip。于是，通过ARP请求，像内网广播网关ip，网关回复mac地址。PC1得到了网关的mac地址，将ip包封装到**以太网帧**，发送给网关。

3. 网关收到该以太网帧，需要转交给dns服务器。同样，网关可能需要发送ARP请求，得到dns的mac地址。

4. dns服务器收到请求，将 www.google.com 的ip发送给网关，网关再根据NAT会话表项，将目的ip转换成PC1的，再发送给PC1（此过程可能同样需要ARP请求）。

5. PC1收到了目的ip，再可以通过类似上面的方式发送请求（目的ip再可以直接填上获取的ip）。

## Example of Sending a Request to a Remote Server for a Web Page

The example is using 5 layered TCP/IP network model.

1. On the application layer (layer #5), the browser initiates an HTTP connection request message to the remote host, www.example.com, to send back the data comprising the contents of a web page. This is the message, and it includes only the **IP Address** of the remote web server.

2. The transport layer (layer #4) encapsulates the message containing the web page request in a **TCP datagram** with the IP address of the remote web server as the destination. Along with the original request packet, this packet now includes the **source port** from which the request will originate, usually a very high number random port, so that the return data knows which port the browser is listening on; and the **destination port** on the remote host, port 80 in this case.

3. The Internet layer (layer #3) encapsulates the TCP datagram in a **packet** that also contains both the **source and destination IP addresses**.

4. The data Link layer (layer #2) uses the **Address Resolution Protocol (ARP)** to identify the **physical MAC address** of the **default router** and encapsulates the Internet packet in a **frame** that includes both the **source and destination MAC addresses**.

5. The frame is sent over the wire (layer #1), usually CAT5 or CAT6, from the NIC on the local host to the NIC on the default router.

6. The default router opens the datagram and determines the destination IP address. The router uses its own routing table to identify the IP address of the next router that will take the frame onto the next step of its journey. The router then re-encapsulates the frame in a new datagram that contains **its own MAC** as the source and the **MAC address of the next router** and then sends it on through the appropriate interface. The router performs its routing task at layer 3, the Internet layer.

**NOTE: switches are invisible to all protocols at layers two and above, so they do not affect the transmission of data in any logical manner.**

## Details about ARP

It answers the question `Why two hosts with different subnet IP addresses and connected by a switch cannot communicate with each other`.

Before any host puts any packet on the wire, the first thing it does is determine whether the destination IP is on **its own network**, or on **a foreign network**. Let's run through it from the perspective of `Host A`.

`Host A` know **its IP** (10.1.2.1), and its **Subnet Mask** (/24, or 255.255.255.0). With a little subnetting, `Host A` determines that its network spans all the **IP addresses in the range** of 10.1.2.0 through 10.1.2.255. (We'll leave out details of the NetID and BroadcastIP, since for the moment they aren't relevant)

`Host A` also knows its destination IP is 10.1.3.1, which falls o**utside of the range** of IP addresses within `Host A`'s own network. As such, `Host A` would come to the conclusion that the destination IP 10.1.3.1 is on a foreign network, and `Host A` could only reach a foreign network by speaking through a **Router**. Or more specifically, through `Host A`'s **default gateway**.

If `Host A` isn't configured with a Default Gateway at this point, then the process ends here with a general failure. `Host A` can not speak to `Host B`.

If `Host A` is configured with a Default Gateway, it would send out an **ARP Request** (which is itself a `Broadcast` frame), **asking for the MAC address of its default gateway -- NOT the MAC address of the final destination IP**.

The **switch**, having received the broadcast frame would `flood` the packet out **all interfaces**, to include the one `Host B` is connected to. `Host B` would indeed receive the packet, but since the ARP is looking for the Default Gateway's MAC address (and not the MAC address of `Host B`), `Host B` would simply **drop and ignore the ARP Request**, without ever sending any sort of response.

`Host A`, then, would never receive a MAC address for its default gateway, and would therefore be **unable to encapsulate the Layer 3 Packet with a Layer 2 header**. The packet would fail there.

**A switch only does two things: forwards frame for which it know the destination MAC address, or flood frames for which it doesn't know the destination MAC address.** A switch never broadcasts.

**A broadcast is a frame who's destination MAC address is ffff.ffff.ffff.** This is a specially **reserved MAC address**, specifically designed for broadcast frames. When a switch encounters a frame destined to ffff.ffff.ffff, its behavior is to always flood that frame.

You could look at it like this, since ffff.ffff.ffff is a **reserved MAC address, it is un-learn-able by the switch**. Therefore, whenever a switch receives something destined to ffff.ffff.ffff, it is forced to flood it out all ports in the LAN/VLAN that the frame was originally received in.