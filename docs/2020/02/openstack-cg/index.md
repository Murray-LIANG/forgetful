# OpenStack Generic Group and Consistent Group


## References
https://docs.openstack.org/cinder/latest/admin/blockstorage-groups.html
https://docs.openstack.org/cinder/latest/contributor/groups.html

## Consistent Group

For a group to support consistent group snapshot, the group specs in the corresponding group type should have the following entry:

```python
{'consistent_group_snapshot_enabled': <is> True}
```

Similarly, for a volume to be in a group that supports consistent group snapshots, the volume type extra specs would also have the following entry:

```python
{'consistent_group_snapshot_enabled': <is> True}
```

## Cinder Cli Microversion

- The minimum microversion to support group type and group specs is 3.11.
- The minimum microversion to support groups operations is 3.13. 
- The minimum microversion to support group snapshots operations is 3.14.

```bash
alias cc="cinder --os-volume-api-version 3.14"

cc get-pools --detail

consistent_group_snapshot_enabled="<is> True"
```
