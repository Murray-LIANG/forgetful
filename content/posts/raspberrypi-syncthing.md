---
title: "Raspberrypi Syncthing"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "raspberrypi",
    "syncthing",
    "private cloud",
]

comment: true
toc: true
auto_collapse_toc: true
---

## UPDATE: raspberrypi and syncthing are not used any more.

1. sudo apt install syncthing
1. sudo systemctl enable syncthing@pi
1. sudo systemctl start sycnthing@pi

Autostart: https://docs.syncthing.net/users/autostart.html#linux

NFS share is enabled on router. Then syncthing on raspberrypi could store the files from phones/PCs to the usb drive.