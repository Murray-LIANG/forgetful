---
title: "How to Source Openrc with Zsh"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "openrc",
    "zsh",
]

comment: true
toc: true
auto_collapse_toc: true
---

1. Create file `openrc_zsh` with content:
```bash
#!/bin/env bash
function sourceopenrc {
    pushd ~/devstack >/dev/null
    eval $(bash -c ". openrc $1 $2;env|sed -n '/OS_/ { s/^/export /;p}'")
    popd >/dev/null
}
sourceopenrc admin admin
```

2. Run `source openrc_zsh`.