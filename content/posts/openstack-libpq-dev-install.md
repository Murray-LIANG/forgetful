---
title: "OpenStack Install libpq-dev on Ubuntu18/Mint19"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "dev",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Meet errors when running `tox -e py27,pep8` of Cinder

If meet below error, need to install `libpq-dev` with version 9.5.14.

**Error: could not determine PostgreSQL version from '10.6'**

