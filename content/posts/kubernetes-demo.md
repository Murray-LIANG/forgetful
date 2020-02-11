---
title: "Kubernetes Demo"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "kubernetes",
    "k8s",
    "demo",
    "unity",
    "csi",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Pre-requisites

1. Pre-configurations on Unity.

    1. Create a Pool and a NasServer.
   
        **NOTE: the NFSv4 needs to be enabled on the NasServer.**

    2. Create hosts for the K8S nodes, `k8s-node1` and `k8s-node2` and manually register the initiators. Use below command to view the initiator name:

    ```console
    $ cat /etc/iscsi/initiatorname.iscsi
    InitiatorName=iqn.<rest of host iscsi initiator>
    ```

2. The docker daemon on all Nodes in K8S should be configured with DNS.
    ```bash
    sudo vim /etc/docker/daemon.json

    { 
        "dns": ["10.245.174.10"]
    }

    sudo systemctl restart docker.service
    ```

## References

- Container image: quay.io/murray_liang/unity-csi
- Code repo: https://github.com/Murray-LIANG/unity-csi

## Commands:

### Mount type volumes

```console
$ kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- bash

$ # Use the ip of `kubectl describe pods mysql-745d95675c-c4kmx`
$ mysql -h 10.244.2.13 -p
password

$ mysql> use mysql

$ mysql> create table my_id(id int(4));

$ mysql> insert my_id values(111);

$ mysql> select * from my_id;

# Shutdown the Node

# Monitor the service is up on another Pod and Node.
```

### Raw block type volumes

```console
# fdisk /dev/xvda
n
w
# mkfs.ext4 /dev/xvda
# mkdir /mnt/data
# mount /dev/xvda /mnt/data
```
Or use `dd`
```console
# dd if=/dev/zero of=/dev/xvda bs=2MB count=1K
```