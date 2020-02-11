---
title: "Docker Tips"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "docker",
    "dns",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Cannot Resolve Host Name inside Running Containers

```bash
sudo vim /etc/docker/daemon.json

{
    "dns": ["10.245.177.15"]
}

sudo systemctl restart docker.service
```