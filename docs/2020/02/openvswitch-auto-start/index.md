# Openvswitch Bridge Auto Boot with Ubuntu 16.04


The bridges of ovs are down after reboot the host. We need to run the commands manually to bring the bridges up and set IP to it.

[This page](https://www.opencloudblog.com/?p=240) gives out the workaround.

Here lists the summary of the workaround.

**It writes Ubuntu16.10 fixes this issue. So this is for 16.04.**

- Install openvswitch-switch tool. **NOTE: it will remove all the bridges which were created by kolla-ansible.**
- Edit the config file: `/lib/systemd/system/openvswitch-nonetwork.service`.

```
[Unit]
Description=Open vSwitch Internal Unit
PartOf=openvswitch-switch.service

#https://bugs.launchpad.net/ubuntu/+source/openvswitch/+bug/1448254

# Without this all sorts of looping dependencies occur doh!
DefaultDependencies=no

#precedants pulled from isup@ service requirements
After=apparmor.service local-fs.target systemd-tmpfiles-setup.service

#subsequent to this service we need the network to start
Wants=network-pre.target openvswitch-switch.service
Before=network-pre.target openvswitch-switch.service

[Service]
Type=oneshot
RemainAfterExit=yes
EnvironmentFile=-/etc/default/openvswitch-switch
ExecStart=/usr/share/openvswitch/scripts/ovs-ctl start \
          --system-id=random $OVS_CTL_OPTS
ExecStop=/usr/share/openvswitch/scripts/ovs-ctl stop
```

- Edit the config file: `/etc/network/interfaces`

```
auto br-ex
allow-ovs br-ex
iface br-ex inet static
        ovs_type OVSBridge
        ovs_ports eth3
        address 192.168.4.3
        netmask 255.255.255.0
        broadcast 192.168.4.255

allow-br-ex eth3
iface eth3 inet manual
        ovs_bridge br-ex
        ovs_type OVSPort
```

