---
title: "Fix the git error on CentOS"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "git",
    "centos",
    "certificate",
]

comment: true
toc: true
auto_collapse_toc: true
---

The error is related to corp certificate.

1. `sudo yum install -y ca-certificates`
1. download the emc certificates from: https://eos2git.cec.lab.emc.com/liangr/corp-notes/blob/master/emc_cert/emc_cacert.crt
1. `cp emc_cacert.crt /etc/pki/ca-trust/source/anchors/`
1. run command: `update-ca-trust extract`