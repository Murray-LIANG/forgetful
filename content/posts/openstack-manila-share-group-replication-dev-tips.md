---
title: "OpenStack Manila Share Group Replication Dev Tips"
date: 2020-06-23T15:45:28+08:00
lastmod: 2020-06-23T15:45:40+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "manila",
    "share group replication",
    "replication",
    "dev tips",
]

comment: true
toc: true
auto_collapse_toc: true
---

## When Share Group Replication Creation Succeed

| Share<br>Replication<br>Type | create_replica<br>/delete_replica<br>Implemented | Group<br>Replication<br>Type | create_share<br>_group_replica<br>Implemented | Succeed in share group replication creation                                                                                                                                                                                                                                      | Note |
|------------------------------|--------------------------------------------------|------------------------------|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------|
| write/read/dr/None           | yes/no                                           | None                         | yes or no                                     | no, raise error from share_group/api.                                                                                                                                                                                                                                            |      |
| write/read/dr/None           | yes/no                                           | write/read/dr                | yes                                           | yes                                                                                                                                                                                                                                                                              |      |
| write/read/dr                | yes                                              | write/read/dr                | no                                            | yes, the default implementation is <br>creating replicas for shares in the group.<br>But `share replication type` should be consistent<br>with `share group replication type`, otherwise,<br>an exception raised when creating <br>the share group (share_group/api.py:create()) |      |
| None                         | yes                                              | write/read/dr                | no                                            | yes, the default implementation is used. Just assume<br>the `share replication type` is the same as<br>`share group replication type`.                                                                                                                                           |      |
| write/read/dr/None           | no                                               | write/read/dr                | no                                            | no, raise error from share/driver.                                                                                                                                                                                                                                               |      |


## Manila DB Upgrade/Downgrade

```console
$ # Get the current version before upgrade.
$ manila-manage db version
e6d88547b381

$ # Update the alembic file.
$ vim manila/db/migrations/alembic/versions/d34c9e9098fa_add_share_group_instances.py
$ # Change the down_revision to e6d88547b381.

$ # Upgrade de.
$ manila-manage db sync


```