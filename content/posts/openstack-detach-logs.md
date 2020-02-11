---
title: "OpenStack Nova Detach Process Logs of Newton Version"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "nova",
    "cinder",
    "detach",
]

comment: true
toc: true
auto_collapse_toc: true
---


## 1. Get connector properties (219ms)

```bash
get_connector_properties: call {
    'execute': None,
    'my_ip': '172.16.1.11',
    'enforce_multipath': True,
    'host': 'newton',
    'root_helper': 'sudo nova-rootwrap /etc/nova/rootwrap.conf',
    'multipath': True}
```

### 1.1 Check `multipathd` status, if `multipath-tools` is not installed or down, exception raises 

```bash
$ multipathd show status
path checker states:
up                  3

paths: 2
busy: False
```

### 1.2 Get the initiator name
```bash
$ cat /etc/iscsi/initiatorname.iscsi
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.1993-08.org.debian:01:7cc8838d1c0
```

### 1.3 Get the fc info
```bash
systool -c fc_host -v
Error opening class fc_host
```

## 2. Detach volume from VM

Detach volume 5ed0cc5c-158c-4857-9bd5-3556884a23f0 from mountpoint /dev/vdb

### 2.1 Attempting initial detach for device `vdb` with retry
The xml used by `libvirt` to detach device from VM
```xml
<disk type="block" device="disk">
  <driver name="qemu" type="raw" cache="none" io="native"/>
  <source dev="/dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9"/>
  <target bus="virtio" dev="vdb"/>
  <serial>5ed0cc5c-158c-4857-9bd5-3556884a23f0</serial>
</disk>
```

## 3. Start retrying detach until device `vdb` is gone

## 4. Calling os-brick to detach iSCSI volume (790ms)

```python
disconnect_volume: call {
    'args': (
        <os_brick.initiator.connectors.iscsi.ISCSIConnector object at 0x7fe51cd174d0>,
        {
            u'target_luns': [197, 197],
            u'target_iqns': [
                u'iqn.1992-04.com.emc:cx.fnm00150600267.a0',
                u'iqn.1992-04.com.emc:cx.fnm00150600267.b0' ],
            u'device_path': u'/dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9',
            u'target_discovered': True,
            u'encrypted': False,
            u'qos_specs': None,
            u'target_iqn': u'iqn.1992-04.com.emc:cx.fnm00150600267.a0',
            u'target_portals': [u'10.245.47.95: 3260', u'10.245.47.96: 3260'],
            u'volume_id': u'5ed0cc5c-158c-4857-9bd5-3556884a23f0', 
            u'target_lun': 197,
            u'access_mode': u'rw',
            u'target_portal': u'10.245.47.95: 3260'
        },
        None),
    'kwargs': {}
}
```

### 4.1 Lock `connect_volume` acquired by `os_brick.initiator.connectors.iscsi.disconnect_volume`


### 4.2 Rescan multipath
```bash
$ multipath -r
reload: 36006016015e03a00ab2df25bb82d0ea9 undef DGC,VRAID
size=3.0G features='1 queue_if_no_path' hwhandler='1 emc' wp=undef
|-+- policy='round-robin 0' prio=50 status=undef
| `- 2:0:0:197 sdd 8:48 active ready running
`-+- policy='round-robin 0' prio=10 status=undef
  `- 3:0:0:197 sdc 8:32 active ready running
```

### 4.3 Get the multipath WWN
```bash
$ /lib/udev/scsi_id --page 0x83 --whitelisted /dev/disk/by-path/ip-10.245.47.95:3260-iscsi-iqn.1992-04.com.emc:cx.fnm00150600267.a0-lun-197
36006016015e03a00ab2df25bb82d0ea9

# Find Multipath device file for volume WWN 36006016015e03a00ab2df25bb82d0ea9

# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 exists yet.

# /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 has shown up.

# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 is read-only.

$ lsblk -o NAME,RO -l -n
sdc                                0
36006016015e03a00ab2df25bb82d0ea9  0
sdd                                0
36006016015e03a00ab2df25bb82d0ea9  0
sdf                                0
vda                                0
vda1                               0
loop0                              0

# Block device /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 is not read-only.

```

### 4.4 Remove multipath device `/dev/sdd`

```bash
$ multipath -l /dev/sdd
36006016015e03a00ab2df25bb82d0ea9 dm-0 DGC,VRAID
size=3.0G features='2 queue_if_no_path retain_attached_hw_handler' hwhandler='1 emc' wp=rw
|-+- policy='round-robin 0' prio=0 status=active
| `- 2:0:0:197 sdd 8:48 active undef running
`-+- policy='round-robin 0' prio=0 status=enabled
  `- 3:0:0:197 sdc 8:32 active undef running

# Found multipath device = /dev/mapper/36006016015e03a00ab2df25bb82d0ea9
```

### 4.5 Flush multipath device 36006016015e03a00ab2df25bb82d0ea9
```bash
$ multipath -f 36006016015e03a00ab2df25bb82d0ea9

```

### 4.6 Get multipath LUNs to remove
```bash
multipath LUNs to remove 
[
    {'device': u'/dev/sdd',
     'host': u'2',
     'id': u'0',
     'channel': u'0',
     'lun': u'197'},
    {'device': u'/dev/sdc',
     'host': u'3', 
     'id': u'0', 
     'channel': u'0', 
     'lun': u'197'}]
```

### 4.7 Flushing IO for device `/dev/sdd`
```bash
$ blockdev --flushbufs /dev/sdd
```

### 4.8 Remove SCSI device `/dev/sdd`
```bash
$ tee -a /sys/block/sdd/device/delete
```

### 4.9 Flushing IO for device `/dev/sdc`
```bash
$ blockdev --flushbufs /dev/sdc
```

### 4.10 Remove SCSI device `/dev/sdc`
```bash
$ tee -a /sys/block/sdc/device/delete
```

### 4.11 Disconnect multipath device /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9
```bash
$ multipath -ll
# no output

$ multipath -r
# no output
```

### 4.12 Lock `connect_volume` released by `os_brick.initiator.connectors.iscsi.disconnect_volume`

Held **0.790s**


## 5. POST call to cinder, action `os-terminate_connection`

### REST request
```bash
$ curl -g -i -X POST \
    http://172.16.1.11:8776/v2/ca2fbd57936b45bf9abdb536dc152a6d/volumes/5ed0cc5c-158c-4857-9bd5-3556884a23f0/action \
    -H "User-Agent: python-cinderclient" -H "Content-Type: application/json" \
    -H "Accept: application/json" -H "X-Auth-Token: {SHA1}deea2e5c2c82807813e48cc3ce629c6ad98c9a7a" \
    -d '{"os-terminate_connection": {"connector": {"platform": "x86_64", "host": "newton", "do_local_attach": false, "ip": "172.16.1.11", "os_type": "linux2", "multipath": true, "initiator": "iqn.1993-08.org.debian:01:7cc8838d1c0"}}}'
```

## 6. POST call to cinder, action `os-detach`

### REST request
```bash
$ curl -g -i -X POST \
    http://172.16.1.11:8776/v2/ca2fbd57936b45bf9abdb536dc152a6d/volumes/5ed0cc5c-158c-4857-9bd5-3556884a23f0/action \
    -H "User-Agent: python-cinderclient" -H "Content-Type: application/json" \
    -H "Accept: application/json" -H "X-Auth-Token: {SHA1}deea2e5c2c82807813e48cc3ce629c6ad98c9a7a" \
    -d '{"os-detach": {"attachment_id": "c0e36b23-5be9-49af-b1fb-f6635e5c186e"}}'
```