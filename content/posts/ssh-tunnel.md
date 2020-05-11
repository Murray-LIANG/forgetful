---
title: "SSH Tunnel"
date: 2020-04-13T21:08:31+08:00
lastmod: 2020-04-13T21:08:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "ssh",
    "tunnel",
    "port forwarding",
]

comment: true
toc: true
auto_collapse_toc: true
---

## On `ubuntu-client1`

```console
# ssh -N root@127.0.0.1 \
    -L 10.245.48.66:29418:review.openstack.org:29418 \
    -L 10.245.48.66:4 43:review.openstack.org:443
```