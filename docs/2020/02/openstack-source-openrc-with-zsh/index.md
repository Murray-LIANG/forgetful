# How to Source Openrc with Zsh


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
