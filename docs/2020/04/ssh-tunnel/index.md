# SSH Tunnel


## On `ubuntu-client1`

```console
# ssh -N root@127.0.0.1 \
    -L 10.245.48.66:29418:review.openstack.org:29418 \
    -L 10.245.48.66:4 43:review.openstack.org:443
```
