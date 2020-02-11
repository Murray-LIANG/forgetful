---
title: "Raspberrypi Backup Files Using crontab and rsync"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "raspberrypi",
    "crontab",
    "rsync",
    "private cloud",
]

comment: true
toc: true
auto_collapse_toc: true
---

## UPDATE: raspberrypi is not used any more.

## Mount the disks

`wd_disk` is usb connected to the raspberrypi.

`seagate` is usb connected to the router, which is exported as a NFS share.
```bash
pi@raspberrypi:/ $ cat /etc/fstab
UUID=F474B7AA74B76DCC /mnt/wd_disk ntfs-3g rw,defaults 0 0
192.168.2.1:/mnt/Seagate_Backup_Plus_Drive /mnt/seagate nfs rw,defaults 0 0
```

## Crontab

```bash
pi@raspberrypi:/mnt/wd_disk/backup $ sudo crontab -e

# Run the command at 11:30 PM every Wendsday.
30 23 * * 3 rsync -azv /mnt/seagate/ /mnt/wd_disk/backup/
```
