---
title: "Glance"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "glance",
]

comment: true
toc: true
auto_collapse_toc: true
---

## glance-api
Dispatch the REST request.

## glance-registry
Process the request about the metadata of image.

## Storage backend
Configed in `/etc/glance/glance-api.conf`.

# Misc
Two processes run for Glance:
- g-api
- g-reg