---
title: "OpenStack Dev Tips"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "dev",
    "tips",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Add release notes

```yaml
---
fixes:
  - |
    Dell EMC Unity Driver: Fixes `bug 1798529
    <https://bugs.launchpad.net/cinder/+bug/1798529>`__ to add an option for
    force deleting the snapshot even if it is attached to hosts.
```