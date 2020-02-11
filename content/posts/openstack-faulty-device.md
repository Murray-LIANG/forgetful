---
title: "OpenStack Faulty Device"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "faulty device",
]

comment: true
toc: true
auto_collapse_toc: true
---

## References

- https://www.cnblogs.com/sting2me/p/8849689.html
- https://www.cnblogs.com/sting2me/p/8888420.html

## Faulty device的产生

### What are faulty devices?
`os-brick` module is mainly used to connect/disconnect the devices.

Interface `connect_volume` is used to discover the block devices of storage systems, `disconnect_volume` is for cleaning the block devices.

When the multipath tool on hosts cannot access the host path, it marks the device as faulty.

### How are faulty devices generated?

The faulty devices show up under the race condition occurs between `connect_volume` and `disconnect_volume`.

_Copied from Peter's blog._

![](/forgetful/images/openstack-faulty-device-os-brick-connect-disconnect-volume.png)

如上的流程在非并发的情况下是表现正常的，host上的device都可以正常连接和清理。

但是，以上逻辑有个实现上的问题，当高并发情况下，会产生faulty device， 考虑以下执行顺序：

1. 右边的`disconnect_volume`执行完毕，存储上LUN对应的device path(在`/dev/disk/by-path`下可以看到）和multipath descriptor（`multipath -l`可以看到）。
2. 这个时候，`connect_volume`锁被释放，左边的`connect_volume`开始执行，而右边的`terminate_connection`还没有执行，也就是说，存储上还没有移除host访问LUN的权限，任何host上的scsi rescan还是会发现这个LUN的device。
3. 接着，`connect_volume`按正常执行，iscsi rescan 和multipath rescan都相继执行，造成在步骤 1）中已经删除的device又重新被scan出来。
4. 然后，右边的`terminate_connection`在存储上执行完成，移除了host对LUN的访问，最终就形成的所谓的faulty device，看到的multipath 输出如下(两个multipath descriptor都是faulty的）：
    ```bash
    $ sudo multipath -ll

    3600601601290380036a00936cf13e711 dm-30 DGC,VRAID
    size=2.0G features='1 retain_attached_hw_handler' hwhandler='1 alua' wp=rw
    |-+- policy='round-robin 0' prio=0 status=active
    | `- 11:0:0:151 sdef 128:112 failed faulty running
    `-+- policy='round-robin 0' prio=0 status=enabled
    `- 12:0:0:151 sdeg 128:128 failed faulty running

    3600601601bd032007c097518e96ae411 dm-2 DGC,VRAID
    size=1.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
    |-+- policy='round-robin 0' prio=0 status=active
    `- #:#:#:# -   #:#   active faulty running
    ```

The problem was that in `connect_volume` and `disconnect_volume` `os-brick` used `multipath -r` and `iscsiadm -m session -R` to rescan all the LUNs below to all iSCSI targets.

### How does `os-brick` avoid to re-scan all LUNs?

Currently `os-brick` narrows down the scan scope. It only re-scans the LUN of specified iSCIS target.

First let's recall the way of linux kernel locates a block device. The address of a SCSI block device contains 4 parts：
1. SCSI adapter number - host (h).
2. Channel number - bus (c).
3. ID number - target (t).
4. LUN ID - (l). 

Example:
```bash
$ ls -l /sys/class/iscsi_host/host3/device/session1/
total 0
drwxr-xr-x 4 root root    0 Apr 21 21:54 connection1:0
drwxr-xr-x 3 root root    0 Apr 21 21:54 iscsi_session
drwxr-xr-x 2 root root    0 Apr 21 21:55 power
drwxr-xr-x 5 root root    0 Apr 21 21:54 target3:0:0   <<< 3:0:0 is h:c:t
-rw-r--r-- 1 root root 4096 Apr 21 21:54 uevent
```

To rescan the specified LUN, `os-brick` cannot use `multipath -r` or `iscsiadm -m session -R` to rescan all the LUNs any more. It uses something like `echo '0 0 1' | tee -a /sys/class/scsi_host/host3/scan` to scan the specified channel - 0, target - 0, lun - 1.