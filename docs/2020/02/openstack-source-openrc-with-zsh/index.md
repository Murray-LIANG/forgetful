# How to Source Openrc with Zsh


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
