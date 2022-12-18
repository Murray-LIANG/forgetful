# OpenStack Dev Tips


## Add release notes

```yaml
---
fixes:
  - |
    Dell EMC Unity Driver: Fixes `bug 1798529
    <https://bugs.launchpad.net/cinder/+bug/1798529>`_ to add an option for
    force deleting the snapshot even if it is attached to hosts.
```

## Check storops version

```python
import pkg_resources
pkg_resources.get_distribution("storops").version
```

Or
```console
$ python -c "import pkg_resources; print(pkg_resources.get_distribution('storops').version)"
```
