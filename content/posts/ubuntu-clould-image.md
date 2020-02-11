---
title: "Ubuntu Cloud Image Installation"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "ubuntu",
    "cloud image",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Download

https://cloud-images.ubuntu.com/

## Upload image to Glance

## Boot an instance from image
**NOTE: don't forget to use the ssh key.**

## Initial configuration

### SSH to the instance

```console
$ ssh -i ~/.ssh/id_rsa.kolla-ansible.liangr ubuntu@172.30.1.18
```

### Create the user `stack`

```console
$ sudo useradd -m -s /bin/bash -G sudo stack

$ sudo passwd stack
```

### Modify `/etc/ssh/sshd_config` to allow login via password

Entry name is `PasswordAuthentication`.