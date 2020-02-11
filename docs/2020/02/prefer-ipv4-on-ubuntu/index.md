# Prefer IPv4 on Ubuntu


Edit `/etc/gai.conf` and locate the line and uncomment it:
```
precedence ::ffff:0:0/96  100
```
