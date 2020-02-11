# Kubernetes Overlay Network


Overlay networks such as `vxlan` or `ipsec` **encapsulate the packet into another packet**. This makes entities addressable that are outside of the scope of another machine.

Any solution on L2 or L3 makes a pod addressable on the network. This means a pod is reachable not just within the Docker network, but is **directly addressable from outside the Docker network**. These could be public or private IP addresses.

The way of `route` and `iptables` described [here](kubernetes-networking.md) is a L3 solution (not a `overlay` networking) making a pod addressable on the network.

`Flannel` is a simple network and is the easiest setup option for an `overlay` network.

## Encapsulation

Say you need to send a packet from the computer `172.9.9.8` to the container/pod `10.4.4.4` and it's on another computer `172.9.9.9`. These two computers are far away from each other, we have to address the packet to the IP address `172.9.9.9` first, then where are we going to put the IP address `10.4.4.4`?

Here we need **encapsulation** where you take a network packet and put it inside another network packet.

For example, instead of sending:
```
IP: 10.4.4.4
TCP stuff
HTTP stuff
```
we will send:
```
IP: 172.9.9.9
(extra wrapper stuff)
IP: 10.4.4.4
TCP stuff
HTTP stuff
```

There are at least 2 different ways of doing encapsulation: there's `ip-in-ip` and `vxlan` encapsulation.

### vxlan
Takes your **whole packet** (including the MAC address) and wraps it inside a UDP packet, which looks like:

```
MAC address: 11:11:11:11:11:11
IP: 172.9.9.9
UDP port: 8472 (the `vxlan port`)
MAC address: ab:cd:ef:12:34:56
IP: 10.4.4.4
TCP port: 80
HTTP stuff
```

### ip-in-ip
Slaps on an extra IP header on top of your old IP header. This means you **don't get to keep the MAC address** you wanted to send it to.

```
MAC address: 11:11:11:11:11:11
IP: 172.9.9.9
IP: 10.4.4.4
TCP port: 80
HTTP stuff
```


## Flannel Networking

Flannel creates another flat network which runs above the host network, this is the so-called `overlay` network. All containers/pod will be assigned one IP address in this overlay network, they communicate with each other by call each other's IP address directly.

```
 Cluster CIDR: 10.244.0.0/16

 Pod CIDR: 10.244.1.0/24
           10.244.2.0/24
           ... ...

 256 Nodes or flannel
 256 Pods per node

 10.244.0.0/16 is called overlay network, while 172.16.1.0/24 is underlay network.

+----------------------+   +---------------------------+   +---------------------------+
| Master               |   | Node-1                    |   | Node-2                    |
|                      |   |                           |   |                           |
|                      |   | +-----------------------+ |   | +-----------------------+ |
|                      |   | | Pod ubuntu-1          | |   | | Pod ubuntu-2          | |
|                      |   | |                       | |   | |                       | |
|                      |   | | +-------------------+ | |   | | +-------------------+ | |
|                      |   | | | eth0: 10.244.1.96 | | |   | | | eth0: 10.244.2.66 | | |
|                      |   | +-+-+-----------------+-+ |   | +-+-+-----------------+-+ |
|                      |   |     |                     |   |     |                     |
|                      |   |     |                     |   |     |                     |
|                      |   | +---+--+                  |   | +---+--+                  |
|                      |   | | veth |                  |   | | veth |                  |
| +----------------+   |   | +------+------+           |   | +------+------+           |
| | Bridge cni0    |   |   | | Bridge cni0 |           |   | | Bridge cni0 |           |
| | 10.244.0.1     |   |   | | 10.244.1.1  |           |   | | 10.244.2.1  |           |
| +-------+--------+   |   | +------+------+           |   | +------+------+           |
|         |            |   |        |                  |   |        |                  |
|         |            |   |        |                  |   |        |                  |
| +-------+--------+   |   | +------+---------+        |   | +------+---------+        |
| | flannel.1      |   |   | | flannel.1      |        |   | | flannel.1      |        |
| | 10.244.0.0     |   |   | | 10.244.1.0     |        |   | | 10.244.2.0     |        |
| +-------+--------+   |   | +------+---------+        |   | +------+---------+        |
|         |            |   |        |                  |   |        |                  |
|         |            |   |        |                  |   |        |                  |
| +-------+----------+ |   | +------+------------+     |   | +------+------------+     |
| | ens3: 172.16.1.8 | |   | | ens3: 172.16.1.14 |     |   | | ens3: 172.16.1.13 |     |
+-+-------+----------+-+   +-+------+------------+-----+   +-+----------+--------+-----+
          |                         |                                   |
          |                         |                                   |
          +-------------------------+-----------------------------------+

```

### Flannel Backends
- vxlan
- host-gw
- UDP

The topology diagram above is using `vxlan` as backend. The one using `UDP` is almost the same.

### Packet Traverse of `vxlan` Backend

#### 0. Kubernetes deployment where pod `ubuntu-1` with IP `10.244.1.96`, pod `ubuntu-2` with IP `10.244.2.66`.

The deployment yaml files for `ubuntu-1` and `ubuntu-2`.
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu-1
  template:
    metadata:
      labels:
        app: ubuntu-1
    spec:
      containers:
        - name: ubuntu-1
          image: anilkreddyr/ubuntu_ping
      nodeSelector:
          kubernetes.io/hostname: k8s-node1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu-2
  template:
    metadata:
      labels:
        app: ubuntu-2
    spec:
      containers:
        - name: ubuntu-2
          image: anilkreddyr/ubuntu_ping
      nodeSelector:
          kubernetes.io/hostname: k8s-node2
```

```
$ kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
ubuntu-1-f949f858-trrn9      1/1     Running   0          19h     10.244.1.96   k8s-node1   <none>           <none>
ubuntu-2-648cdb99dc-dv898    1/1     Running   0          19h     10.244.2.66   k8s-node2   <none>           <none>
```

#### 1. Pod `ubuntu-1` pings `ubuntu-2` successfully.

```console
$ kubectl exec ubuntu-1-f949f858-trrn9 -it bash

$ # ubuntu-1 has IP 10.244.1.96
root@ubuntu-1-f949f858-trrn9:/# ip addr show eth0
3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default
    link/ether 2a:77:20:40:d1:bf brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.1.96/24 scope global eth0
       valid_lft forever preferred_lft forever

root@ubuntu-1-f949f858-trrn9:/# ping 10.244.2.66
PING 10.244.2.66 (10.244.2.66) 56(84) bytes of data.
64 bytes from 10.244.2.66: icmp_seq=1 ttl=62 time=2.51 ms
64 bytes from 10.244.2.66: icmp_seq=2 ttl=62 time=1.15 ms

$ # ubuntu-1 routes to ubuntu-2 via the 2nd routing rule,
$ # because ubuntu-2's IP 10.244.2.66 is in the huge subnet of 10.244.0.0/16.
$ # So, the next hop is `eth0`.
root@ubuntu-1-f949f858-trrn9:/# ip route get 10.244.2.66
10.244.2.66 via 10.244.1.1 dev eth0  src 10.244.1.96
    cache

root@ubuntu-1-f949f858-trrn9:/# ip route
default via 10.244.1.1 dev eth0
10.244.0.0/16 via 10.244.1.1 dev eth0
10.244.1.0/24 dev eth0  proto kernel  scope link  src 10.244.1.96
```

As we know, a `veth` pair is created, one end of the veth pair is `eth0` of pod, and the other end `vethxxx` is an interface on the bridge and created in root ns. You could follow below steps to view the veth pair.

```console
$ # 1. Check the sandbox of ubuntu-1's parent container,
$ # because ubuntu-1 pod is using the parent's network.
root@k8s-node1:~# docker inspect k8s_POD_ubuntu-1-f949f858-trrn9_default_5ecd1edb-a93d-11e9-9842-fa163ece4076_0 | grep Sandbox
            "SandboxID": "6d0f066e5e19076683b9ddd53ca25fc088b42ec5dacee8ad1a33b51cda196203",
            "SandboxKey": "/var/run/docker/netns/6d0f066e5e19",

$ # 2. View the pod's network information by entering the network namespace.
root@k8s-node1:~# nsenter --net="/var/run/docker/netns/6d0f066e5e19" ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default
    link/ether 2a:77:20:40:d1:bf brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.1.96/24 scope global eth0
       valid_lft forever preferred_lft forever

$ # 3. Identify the `peer_ifindex`
root@k8s-node1:~# nsenter --net="/var/run/docker/netns/6d0f066e5e19" ethtool -S eth0
NIC statistics:
     peer_ifindex: 7

$ # 4. Get the peer in root ns
root@k8s-node1:~# ip -d link | grep '7: '
7: vethae159e77@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue master cni0 state UP mode DEFAULT group default
```

**NOTE: Why does the network namespace need `nsenter` and cannot work with `ip netns`?**
Because `ip netns` only works with namespace symlinks in `/var/run/netns` but the namespaces in docker are under `/var/run/docker/netns`. If you create a symlinks under `/var/run/netns` for docker network namespace, you can use `ip netns` command.

#### 2. Now the packet leaves from the pod and enters bridge `cni0`'s `veth` interface.

```console
$ # vethae159e77 interface is on bridge cni0
root@k8s-node1:~# brctl show
bridge name     bridge id               STP enabled     interfaces
cni0            8000.da54dcf2f3fe       no              veth8b9bf99b
                                                        vethae159e77
docker0         8000.0242a9fee8af       no

$ # cni0 has IP 10.244.1.1 which is the next hop of above routing
root@k8s-node1:~# ip -d -4 addr show cni0
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default qlen 1000
    link/ether da:54:dc:f2:f3:fe brd ff:ff:ff:ff:ff:ff promiscuity 0
    bridge forward_delay 1500 hello_time 200 max_age 2000 ageing_time 30000 stp_state 0 priority 32768 vlan_filtering 0 vlan_protocol 802.1Q
    inet 10.244.1.1/24 scope global cni0
       valid_lft forever preferred_lft forever

$ # Capture packets on interface vethae159e77
root@k8s-node1:~# tcpdump -vv -ni vethae159e77 icmp
tcpdump: listening on vethae159e77, link-type EN10MB (Ethernet), capture size 262144 bytes

06:15:13.046136 IP (tos 0x0, ttl 64, id 52778, offset 0, flags [DF], proto ICMP (1), length 84)
    10.244.1.96 > 10.244.2.66: ICMP echo request, id 39, seq 1, length 64
```

#### 3. Determine the next hop on the root netns.

Bridge `cni0` is in the root netns.

```console
root@k8s-node1:~# ip route
default via 172.16.1.1 dev ens3
10.244.0.0/24 via 10.244.0.0 dev flannel.1 onlink
10.244.1.0/24 dev cni0  proto kernel  scope link  src 10.244.1.1
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink

root@k8s-node1:~# ip route get 10.244.2.66
10.244.2.66 via 10.244.2.0 dev flannel.1  src 10.244.1.0
    cache

$ # Check the ARP cache for the next hop `10.244.2.0`
$ # A Layer 3 device looks into the ARP table to figure out what MAC address
$ # to put into the packet header. Here it needs to find the MAC address of
$ # IP 10.244.2.0.
root@k8s-node1:~# ip neighbor show 10.244.2.0
10.244.2.0 dev flannel.1 lladdr 52:17:43:ba:2e:56 PERMANENT

$ # A Layer 2 device inspects the destination MAC address of the frame and
$ # look to its FDB table for information on where to send that specific
$ # Ethernet frame. If the FDB table doesn't have any information on that
$ # specific MAC address it will flood the Ethernet frame out to all ports in
$ # the broadcast domain.
root@k8s-node1:~# bridge fdb show dev flannel.1 | grep 52:17:43:ba:2e:56
52:17:43:ba:2e:56 dst 172.16.1.13 self permanent

root@k8s-node1:~# ip neighbor show 172.16.1.13
172.16.1.13 dev ens3 lladdr fa:16:3e:cc:78:67 STALE
```

**NOTE: Who writes bridge FDB table entries?** Because `flannel` pod is deployed as a `daemon set`. When a new node joins the cluster, `kube-flannel` will be launched on it, and it'll publish the new node's `public-ip` and `VtepMAC` as annotations. And, each node can read this information and the FDB entry is programmed.

#### 4. Now the packet gets encapsulated and sent out to remote VTEP 172.16.1.13.

```console
$ # Pay attention to the `dstport` 8472 which is the destination port of the
$ # encapsulated packet.
root@k8s-node1:~# ip -d link show flannel.1
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/ether 26:62:24:f0:61:01 brd ff:ff:ff:ff:ff:ff promiscuity 0
    vxlan id 1 local 172.16.1.14 dev ens3 srcport 0 0 dstport 8472 nolearning ageing 300 addrgenmode eui64

$ # The packet is encapsulated indeed. You can see the packet is a ICMP
$ # wrapped by a UDP. 
root@k8s-node1:~# tcpdump -vv -ni ens3 udp
tcpdump: listening on ens3, link-type EN10MB (Ethernet), capture size 262144 bytes

07:26:48.356032 IP (tos 0x0, ttl 64, id 1164, offset 0, flags [none], proto UDP (17), length 134)
    172.16.1.14.53811 > 172.16.1.13.8472: [no cksum] OTV, flags [I] (0x08), overlay 0, instance 1
IP (tos 0x0, ttl 63, id 50392, offset 0, flags [DF], proto ICMP (1), length 84)
    10.244.1.96 > 10.244.2.66: ICMP echo request, id 40, seq 1, length 64
```

#### 5. The packet is routed to `k8s-node2` in nodes' network. On `k8s-node2` it gets decapsulated, then sent to `cni0` bridge and its `veth` interface in root netns, finally sent to pod `ubuntu-2`.

```console
$ # 10.244.2.66 is in the subnet of 10.244.2.0/24 (the 4th one)
k8s@k8s-node2:~$ ip route
default via 172.16.1.1 dev ens3
10.244.0.0/24 via 10.244.0.0 dev flannel.1 onlink
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink
10.244.2.0/24 dev cni0  proto kernel  scope link  src 10.244.2.1
169.254.169.254 via 172.16.1.2 dev ens3
172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.13
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1 linkdown

k8s@k8s-node2:~$ ip route get 10.244.2.66
10.244.2.66 dev cni0  src 10.244.2.1
    cache

$ # 10.244.2.1 is the IP of cni0
k8s@k8s-node2:~$ ip addr show cni0
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default qlen 1000
    link/ether 96:d3:bf:4f:14:2d brd ff:ff:ff:ff:ff:ff
    inet 10.244.2.1/24 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::94d3:bfff:fe4f:142d/64 scope link
       valid_lft forever preferred_lft forever

k8s@k8s-node2:~$ brctl show cni0
bridge name     bridge id               STP enabled     interfaces
cni0            8000.96d3bf4f142d       no              veth69924cbc
                                                        veth75afbddc
                                                        veth93c66b54

root@k8s-node2:~# docker inspect k8s_POD_ubuntu-2-648cdb99dc-dv898_default_5ecbede9-a93d-11e9-9842-fa163ece4076_0 | grep Sandbox
            "SandboxID": "bfc09105b49d438278cc677d67afdaa05466cb326563203ffc5e3deb2591ef75",
            "SandboxKey": "/var/run/docker/netns/bfc09105b49d",

root@k8s-node2:~# nsenter --net="/var/run/docker/netns/bfc09105b49d" ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default
    link/ether be:33:b3:56:88:41 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.2.66/24 scope global eth0
       valid_lft forever preferred_lft forever

root@k8s-node2:~# ip addr
... ...
8: veth69924cbc@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue master cni0 state UP group default
    link/ether 32:db:75:2f:0e:3e brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::30db:75ff:fe2f:e3e/64 scope link
       valid_lft forever preferred_lft forever

```

## UDP Encapsulation vs vxlan Encapsulation

`vxlan` is the default backend of `flannel`, because its performance is better than `UDP`. You can think `vxlan` as an additional layer above the `TCP/UDP` layer and `vxlan` knows how to route vxlan packets (having `vxlan` header) with its own protocol.

While the `UDP` backend copies the packet back and forth from the user space to kernel space, so it isn't recommended to use in production as performance concern. But we can still use `UDP` for debugging and testing purpose.

![](/images/kubernetes-overlay-network-flannel-udp.png)

Like in `vxlan`, the packet is sent to `flannel0` according to the routing table.
The difference is `flannel0` here is a **TUN** device which is created by `flanneld` daemon process. `TUN` is a software interface implemented in linux kernel, it can pass raw ip packet between user program and the kernel. It works in two directions:
- when it writes IP packet to the `flannel0` device, the packet will be send to kernel directly, and kernel will route the packet according to its route table.
- when an IP packet arrives to the kernel, and the route tables says it should be routed to `flannel0` device, kernel will send the packet directly to the process which created this device, which is the `flanneld` daemon process.

So, it has to copy the packet 3 times between the user space and the kernel space, which will increase the network overhead in a significant way.

![](/images/kubernetes-overlay-network-flannel-udp-packet-copy.png)

## Flannel IPAM (IP Address Management)
Flannel doesn't allocate and maintain the IP addresses to pods, `host-local` CNI plugin is responsible for this.

We can check the `ubuntu-1`'s network.
```console
$ # Get the id of parent container
root@k8s-node1:~# docker ps | grep k8s_POD_ubuntu-1
0dc487cc6e4f        k8s.gcr.io/pause:3.1             "/pause"                 23 hours ago        Up 23 hours                             k8s_POD_ubuntu-1-f949f858-trrn9_default_5ecd1edb-a93d-11e9-9842-fa163ece4076_0

$ # It is using `cbr0` and 10.244.1.0/24 IPAM.
root@k8s-node1:~# cat /var/lib/cni/flannel/0dc487cc6e4faf25ddd504d0e59339f2edf94f10ea664f9765be514bb3219cff
{"hairpinMode":true,"ipMasq":false,"ipam":{"routes":[{"dst":"10.244.0.0/16"}],"subnet":"10.244.1.0/24","type":"host-local"},"isDefaultGateway":true,"isGateway":true,"mtu":1400,"name":"cbr0","type":"bridge"}

root@k8s-node1:~# cat /var/lib/cni/networks/cbr0/10.244.1.96
0dc487cc6e4faf25ddd504d0e59339f2edf94f10ea664f9765be514bb3219cff
```
