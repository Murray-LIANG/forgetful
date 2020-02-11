# OSP14 Configure Network for Manila


## OSP Network Topology
![](/forgetful/images/osp-configure-network-for-manila.png)

## Environment

The undercloud is 192.168.1.202. 

The overcloud is deployed on ubuntu-server2 (compute node) and ubuntu-server10 (control node).

Ethernet interfaces on ubuntu-server10 are:
- enp8s0f0:
- enp8s0f1: eth1, used for overcloud tenant network.
- enp8s0f2: eth0, used for OSP Provision/Ctrl Plane network.
- eno1: eth2, not used.
- enp8s0f3:
- ens2f1: eth3, connected to br-ex, used to External network.

Ethernet interfaces on ubuntu-server2 are:
- TBD

Undercloud uses 192.168.139.* subnet to dhcp overcloud nodes. Ubuntu-server2 gets 192.168.139.209 while ubuntu-server10 gets 192.168.139.205.

Ubuntu-server10 is assigned 192.168.1.213 to br-ex as the public API interface.

## Network Configuration

1. Deploy OSP14.
2. Configure control node ubuntu-server10:
   1. `sudo vi /var/lib/config-data/puppet-generated/neutron/etc/neutron/plugins/ml2/ml2_conf.ini`
        ```ini
        ......
        [ml2]

        type_drivers=vlan,flat,gre

        tenant_network_types=vlan,flat

        ......
        [ml2_type_vlan]

        network_vlan_ranges=vlannet:210:219
        ......
        ```
   2. `sudo vi /var/lib/config-data/puppet-generated/neutron/etc/neutron/plugins/ml2/openvswitch_agent.ini`
3. `sudo ovs-vsctl add-br br-eth1`
4. `sudo ovs-vsctl add-port br-eth1 enp8s0f1`
