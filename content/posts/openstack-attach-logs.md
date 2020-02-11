---
title: "OpenStack Nova Attach Process Logs of Newton Version"
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
    "attach",
]

comment: true
toc: true
auto_collapse_toc: true
---


## 1. Reserve the VM

### 1.1 Get a lock on VM by `nova.compute.manager.do_reserve`

### 1.2 Update the DB via `conductor`

### 1.3 Release the lock by `nova.compute.manager.do_reserve`

## 2. Get a lock on VM by `nova.compute.manager.do_attach_volume` (will released in the end)

## 3. GET call to cinder to get information of volume

### 3.1 Print a log `Attaching volume 5ed0cc5c-158c-4857-9bd5-3556884a23f0 to /dev/vdb`

### 3.2 REST request to cinder
```bash
REQ: curl -g -i -X GET \
    http://172.16.1.11:8776/v2/ca2fbd57936b45bf9abdb536dc152a6d/volumes/5ed0cc5c-158c-4857-9bd5-3556884a23f0 \
    -H "User-Agent: python-cinderclient" -H "Accept: application/json" \
    -H "X-Auth-Token: {SHA1}e36480f1c9d79821db2a9ad7a4245db35a3529f8"
```

## 4. Get connector properties

Logs of `get_connector_properties`.

### 4.1 Check `multipathd` status, if `multipath-tools` is not installed or down, exception raises.
```bash
$ multipathd show status

path checker states:
up                  3

paths: 2
busy: False
```

### 4.2 Get the initiator name
```bash
$ cat /etc/iscsi/initiatorname.iscsi
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.1993-08.org.debian:01:7cc8838d1c0
```

### 4.3 Get the fc info
```bash
$ systool -c fc_host -v
Error opening class fc_host
```

## 5. POST call to cinder, action: `os-initialize_connection`

### REST request to cinder
```bash
curl -g -i -X POST \
    http://172.16.1.11:8776/v2/ca2fbd57936b45bf9abdb536dc152a6d/volumes/5ed0cc5c-158c-4857-9bd5-3556884a23f0/action \
    -H "User-Agent: python-cinderclient" -H "Content-Type: application/json" \
    -H "Accept: application/json" -H "X-Auth-Token: {SHA1}e36480f1c9d79821db2a9ad7a4245db35a3529f8" \
    -d '{"os-initialize_connection": {"connector": {"platform": "x86_64", "host": "newton", "do_local_attach": false, "ip": "172.16.1.11", "os_type": "linux2", "multipath": true, "initiator": "iqn.1993-08.org.debian:01:7cc8838d1c0"}}}'
```
**Sync** wait for Cinder result, **about 7 sec**.

### Returned by `initialize_connection`:
```python
{
    u'volume_id': u'e8f58c95-3f8a-40dc-861d-163ce68fcdaf',

    u'target_luns': [192, 192],
    u'target_lun': 192,

    u'target_iqns': [
        u'iqn.1992-04.com.emc:cx.apm00150602415.a0', u'iqn.1992-04.com.emc:cx.apm00150602415.b0'],
    u'target_iqn': u'iqn.1992-04.com.emc:cx.apm00150602415.b0',

    u'target_portals': [u'192.168.1.59:3260', u'192.168.1.60:3260'],
    u'target_portal': u'192.168.1.60:3260'

    u'target_discovered': True,
    u'encrypted': False,
    u'qos_specs': None,
    u'access_mode': u'rw', 
}
```

## 6. Calling `os-brick` to attach iSCSI Volume `connect_volume`

### 6.1 Lock `connect_volume` acquired by `os_brick.initiator.connectors.iscsi.connect_volume`

### 6.2 Multipath discovery for iSCSI enabled

### 6.3 Trying to connect to iSCSI portal 192.168.1.59:3260

**This is a failure case, successful case is like the one to portal 192.168.1.60:3260**

```bash
$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260
# failed. Not Retrying. 
# Exception during request[140621460555248]: Unexpected error while running command.
# Command: iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260
# Exit code: 21
# Stdout: u''
# Stderr: u'iscsiadm: No records found\n'

$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260 --interface default --op new
# New iSCSI node [tcp:[hw=,ip=,net_if=,iscsi_if=default] 192.168.1.59,3260,-1 iqn.1992-04.com.emc:cx.apm00150602415.a0] added

$ iscsiadm -m session
# tcp: [1] 10.245.47.95:3260,2 iqn.1992-04.com.emc:cx.fnm00150600267.a0 (non-flash)
# tcp: [2] 10.245.47.96:3260,1 iqn.1992-04.com.emc:cx.fnm00150600267.b0 (non-flash)

$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260 --login
# failed. Not Retrying.
# Unexpected error while running command.
# Command: iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260 --login
# Exit code: 8
# Stdout: u'Logging in to [iface: default, target: iqn.1992-04.com.emc:cx.apm00150602415.a0, portal: 192.168.1.59,3260] (multiple)\n'
# Stderr: u'iscsiadm: Could not login to [iface: default, target: iqn.1992-04.com.emc:cx.apm00150602415.a0, portal: 192.168.1.59,3260].\niscsiadm: initiator reported error (8 - connection timed out)\niscsiadm: Could not log into all portals\n'
```
**iscsi login failed and the timeout is 2 min.**

### 6.4 Trying to connect to iSCSI portal 192.168.1.60:3260
```bash
$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.b0 -p 192.168.1.60:3260
# failed. Not Retrying.
# Exception during request[140621460555248]: Unexpected error while running command.
# Command: iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.b0 -p 192.168.1.60:3260
# Exit code: 21
# Stdout: u''
# Stderr: u'iscsiadm: No records found\n'

$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.b0 -p 192.168.1.60:3260 --interface default --op new
# New iSCSI node [tcp:[hw=,ip=,net_if=,iscsi_if=default] 192.168.1.60,3260,-1 iqn.1992-04.com.emc:cx.apm00150602415.b0] added\n

$ iscsiadm -m session
# tcp: [1] 10.245.47.95:3260,2 iqn.1992-04.com.emc:cx.fnm00150600267.a0 (non-flash)
# tcp: [2] 10.245.47.96:3260,1 iqn.1992-04.com.emc:cx.fnm00150600267.b0 (non-flash)

$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.b0 -p 192.168.1.60:3260 --login
# Logging in to [iface: default, target: iqn.1992-04.com.emc:cx.apm00150602415.b0, portal: 192.168.1.60,3260] (multiple)
# Login to [iface: default, target: iqn.1992-04.com.emc:cx.apm00150602415.b0, portal: 192.168.1.60,3260] successful.

$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.b0 -p 192.168.1.60:3260 --op update -n node.startup -v automatic

$ iscsiadm -m node --rescan
# Rescanning session [sid: 1, target: iqn.1992-04.com.emc:cx.fnm00150600267.a0, portal: 10.245.47.95,3260]
# Rescanning session [sid: 2, target: iqn.1992-04.com.emc:cx.fnm00150600267.b0, portal: 10.245.47.96,3260]
# Rescanning session [sid: 3, target: iqn.1992-04.com.emc:cx.apm00150602415.b0, portal: 192.168.1.60,3260]

$ iscsiadm -m node --rescan
# Same result
```

ISCSI volume not yet found at: [u'/dev/disk/by-path/ip-192.168.1.59:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.a0-lun-192', u'/dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192']. Will rescan & retry.  Try number: 0.
```bash
# Two times more rescan
iscsiadm -m node --rescan
# Same result
```

### 6.5 Multipath discovery for iSCSI enabled.

### 6.6 Trying to connect to iSCSI portal 192.168.1.59:3260.

```bash
$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260

$ iscsiadm -m session
# tcp: [1] 10.245.47.95:3260,2 iqn.1992-04.com.emc:cx.fnm00150600267.a0 (non-flash)
# tcp: [2] 10.245.47.96:3260,1 iqn.1992-04.com.emc:cx.fnm00150600267.b0 (non-flash)
# tcp: [3] 192.168.1.60:3260,5 iqn.1992-04.com.emc:cx.apm00150602415.b0 (non-flash)

$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260 --login
# failed. Not Retrying.
# Unexpected error while running command.
# Command: iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.a0 -p 192.168.1.59:3260 --login
# Exit code: 8
# Stdout: u'Logging in to [iface: default, target: iqn.1992-04.com.emc:cx.apm00150602415.a0, portal: 192.168.1.59,3260] (multiple)\n'
# Stderr: u'iscsiadm: Could not login to [iface: default, target: iqn.1992-04.com.emc:cx.apm00150602415.a0, portal: 192.168.1.59,3260].\niscsiadm: initiator reported error (8 - connection timed out)\niscsiadm: Could not log into all portals\n'
```

**iscsi login failed and the timeout is 2 min.**

### 6.7 Trying to connect to iSCSI portal 192.168.1.60:3260
```bash
$ iscsiadm -m node -T iqn.1992-04.com.emc:cx.apm00150602415.b0 -p 192.168.1.60:3260

$ iscsiadm -m session
# tcp: [1] 10.245.47.95:3260,2 iqn.1992-04.com.emc:cx.fnm00150600267.a0 (non-flash)
# tcp: [2] 10.245.47.96:3260,1 iqn.1992-04.com.emc:cx.fnm00150600267.b0 (non-flash)
# tcp: [3] 192.168.1.60:3260,5 iqn.1992-04.com.emc:cx.apm00150602415.b0 (non-flash)

$ iscsiadm -m node --rescan
# Rescanning session [sid: 1, target: iqn.1992-04.com.emc:cx.fnm00150600267.a0, portal: 10.245.47.95,3260]
# Rescanning session [sid: 2, target: iqn.1992-04.com.emc:cx.fnm00150600267.b0, portal: 10.245.47.96,3260]
# Rescanning session [sid: 3, target: iqn.1992-04.com.emc:cx.apm00150602415.b0, portal: 192.168.1.60,3260]

$ iscsiadm -m node --rescan
# Same result
```

Found iSCSI node [u'/dev/disk/by-path/ip-192.168.1.59:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.a0-lun-192', u'/dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192'] (after 1 rescans)

### 6.8 Getting the device WWN
```bash
/lib/udev/scsi_id --page 0x83 --whitelisted /dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192
# 36006016074e03a008d3cf25b04e2daae
```

Device WWN = '36006016074e03a008d3cf25b04e2daae'

### 6.9 Trying the multipath using device WWN

Find Multipath device file for volume WWN **36006016074e03a008d3cf25b04e2daae**.

First try the path like: `/dev/disk/by-id/dm-uuid-mpath-<wwn>`.

Then the path like: `/dev/mapper/<wwn>`.

- Failed multipath case

```bash
# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016074e03a008d3cf25b04e2daae doesn't exists yet.
# Failed attempt 1
# Sleeping for 2 seconds
# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016074e03a008d3cf25b04e2daae exists yet.
# Failed attempt 2
# Sleeping for 4 seconds
# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016074e03a008d3cf25b04e2daae exists yet.
# Failed attempt 3

# Checking to see if /dev/mapper/36006016074e03a008d3cf25b04e2daae exists yet.
# Failed attempt 1
# Sleeping for 2 seconds
# Checking to see if /dev/mapper/36006016074e03a008d3cf25b04e2daae exists yet.
# Failed attempt 2
# Sleeping for 4 seconds
# Checking to see if /dev/mapper/36006016074e03a008d3cf25b04e2daae exists yet.
# Failed attempt 3
```

**couldn't find a valid multipath device path for 36006016074e03a008d3cf25b04e2daae**.

- Successful multipath case
```bash
# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 exists yet.
# Failed attempt 1
# Sleeping for 2 seconds
# Checking to see if /dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 exists yet.
#/dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9 has shown up. 
```

### 6.10 Trying the single path if multipath failed
```bash
multipath -l /dev/sdf

# Unable to find multipath device name for volume. Using path /dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192 for volume.
```

### 6.11 Checking the path read-only or not
```bash
# Checking to see if /dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192 is read-only.

# lsblk -o NAME,RO -l -n
# sdc                                0
# 36006016015e03a00ab2df25bb82d0ea9  0
# sdd                                0
# 36006016015e03a00ab2df25bb82d0ea9  0
# sdf                                0
# vda                                0
# vda1                               0
# loop0                              0

# Block device /dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192 is not read-only.
```

### 6.12 Lock `connect_volume` released (comparing the time used)

- Multipath case
```bash
connect_volume: return (3162ms) {
    'path': u'/dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9',
    'scsi_wwn': u'36006016015e03a00ab2df25bb82d0ea9',
    'type': 'block',
    'multipath_id': u'36006016015e03a00ab2df25bb82d0ea9'}
```

- Single path case
```bash
connect_volume: return (254525ms) {
    'path': u'/dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192', 
    'scsi_wwn': u'36006016074e03a008d3cf25b04e2daae',
    'type': u'block'}
```

## 7. `libvirt` attaches device to VM
### 7.1 `libvert` command
- Multipath case
```bash
attach device xml: <disk type="block" device="disk">
  <driver name="qemu" type="raw" cache="none" io="native"/>
  <source dev="/dev/disk/by-id/dm-uuid-mpath-36006016015e03a00ab2df25bb82d0ea9"/>
  <target bus="virtio" dev="vdb"/>
  <serial>5ed0cc5c-158c-4857-9bd5-3556884a23f0</serial>
</disk>
 attach_device /opt/stack/nova/nova/virt/libvirt/guest.py:304
```
- Single path case
```bash
attach device xml: <disk type="block" device="disk">
  <driver name="qemu" type="raw" cache="none" io="native"/>
  <source dev="/dev/disk/by-path/ip-192.168.1.60:3260-iscsi-iqn.1992-04.com.emc:cx.apm00150602415.b0-lun-192"/>
  <target bus="virtio" dev="vdb"/>
  <serial>e8f58c95-3f8a-40dc-861d-163ce68fcdaf</serial>
</disk>
```
### 7.2 Update DB via `conductor`

## 8. POST call to cinder, action `os-attach`

### 8.1 REST request
```bash
curl -g -i -X POST \
    http://172.16.1.11:8776/v2/ca2fbd57936b45bf9abdb536dc152a6d/volumes/5ed0cc5c-158c-4857-9bd5-3556884a23f0/action \
    -H "User-Agent: python-cinderclient" -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    -H "X-Auth-Token: {SHA1}e36480f1c9d79821db2a9ad7a4245db35a3529f8" \
    -d '{"os-attach": {"instance_uuid": "62b302e5-6d95-4e61-8dd4-6aa8b1049545", "mountpoint": "/dev/vdb", "mode": "rw"}}'
```

### 8.2 DellEMC drivers don't implement `os-attach`

## 9. Release the lock by `nova.compute.manager.do_attach_volume`