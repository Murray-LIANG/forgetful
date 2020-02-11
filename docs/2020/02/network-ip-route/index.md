# Command `ip route`


## Display a routing table with ip route show

http://linux-ip.net/html/tools-ip-route.html

```console
[root@tristan]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.99.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
127.0.0.0       0.0.0.0         255.0.0.0       U     0      0        0 lo
0.0.0.0         192.168.99.254  0.0.0.0         UG    0      0        0 eth0
[root@tristan]# ip route show
192.168.99.0/24 dev eth0  scope link 
127.0.0.0/8 dev lo  scope link 
default via 192.168.99.254 dev eth0
```
The network `192.168.99.0/24` is available on `eth0` with a scope of `link`, which means that the network is valid and reachable through this device `eth0`.
The scope could be any of:
- global, valid everywhere
- site, valid only within this site (IPv6)
- link, valid only on this device
- host, valid only inside this host (machine)


The example of our OpenStack env.
```console
root@ubuntu-server1:~# ip route show
default via 10.245.48.1 dev eth0  metric 100
10.245.48.0/24 dev eth0  proto kernel  scope link  src 10.245.48.67
172.30.0.0/16 dev eth1  proto kernel  scope link  src 172.30.0.1
192.168.1.0/24 dev eth1  proto kernel  scope link  src 192.168.1.1
192.168.2.0/24 dev eth2  proto kernel  scope link  src 192.168.2.1
192.168.4.0/24 dev eth4  proto kernel  scope link  src 192.168.4.1
```

```console
[root@tristan]# ip route show table local
local 192.168.99.35 dev eth0  proto kernel  scope host  src 192.168.99.35 
broadcast 127.255.255.255 dev lo  proto kernel  scope link  src 127.0.0.1 
broadcast 192.168.99.255 dev eth0  proto kernel  scope link  src 192.168.99.35 
broadcast 127.0.0.0 dev lo  proto kernel  scope link  src 127.0.0.1 
local 127.0.0.1 dev lo  proto kernel  scope host  src 127.0.0.1 
local 127.0.0.0/8 dev lo  proto kernel  scope host  src 127.0.0.1
```

- `local 192.168.99.35`, `broadcast 192.168.99.255`

    tells us whether the route is for a broadcast address or an IP address or range locally hosted on this machine.

- `dev eth0`
  
    informs us through which device the destination is reachable.

- `proto kernel`

    means that the kernel has added these routes as part of bringing up the IP layer interfaces.

- `scope host`, `scope link`
    
    For each IP hosted on the machine, it makes sense that the machine should restrict accessibility to that IP or IP range to itself only. `local 192.168.99.35` has a `host` scope. Because the host has this IP, there's no reason for the packet to be routed off the box. Similarly, a destination of localhost (127.0.0.1) does not need to be forwarded off this machine.

    For broadcast addresses, which are intended for any listeners who happen to share the IP network, the destination only makes sense as for a scope of devices connected to the same link layer.

- `src 192.168.99.35`, `src 127.0.0.1`
    
    a hint to the kernel about what IP address to select for a source address on outgoing packets on this interface.

