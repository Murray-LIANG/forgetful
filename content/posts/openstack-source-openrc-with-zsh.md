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
# Single quotes of 'EOF' disable the expansion of $ in the function sourceopenrc.
cat > ~/devstack/openrc_zsh <<'EOF'
#!/bin/env bash
function sourceopenrc {
    pushd ~/devstack >/dev/null
    eval $(bash -c ". openrc $1 $2 &>/dev/null;env|sed -n '/OS_/ { s/^/export /;p}'")
    popd >/dev/null
}

sourceopenrc $1 $2
EOF
```

2. Run `source openrc_zsh admin admin`.