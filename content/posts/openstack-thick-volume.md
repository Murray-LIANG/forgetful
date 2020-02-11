---
title: "OpenStack Thick Volume"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "cinder",
    "thick volume",
]

comment: true
toc: true
auto_collapse_toc: true
---

```bash
openstack volume type create thick --property provisioning:type='thick' \
    --property thick_provisioning_support='<is> True'
```