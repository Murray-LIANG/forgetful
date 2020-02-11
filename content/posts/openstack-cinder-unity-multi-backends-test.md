---
title: "OpenStack Cinder Multiple Backends"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "cinder",
]

comment: true
toc: true
auto_collapse_toc: true
---
## Cinder configuration

Some settings in `/ect/cinder/cinder.conf`

```ini
[DEFAULT]
... ...
enabled_backends = unity_backend_1,unity_backend_2
... ...

[unity_backend_1]
storage_protocol = iSCSI
san_ip = 10.245.101.39
san_login = admin
san_password = Password123!
volume_driver = cinder.volume.drivers.dell_emc.unity.Driver
volume_backend_name = unity_backend_1
force_delete_attached_snapshots = True

[unity_backend_2]
storage_protocol = iSCSI
san_ip = 192.168.1.58
san_login = admin
san_password = Password123!
volume_driver = cinder.volume.drivers.dell_emc.unity.Driver
volume_backend_name = unity_backend_2
force_delete_attached_snapshots = True

```

```bash
stack@newton:~/storops$ openstack volume type create --property volume_backend_name=unity_backend_1 type-1
+---------------------------------+---------------------------------------+
| Field                           | Value                                 |
+---------------------------------+---------------------------------------+
| description                     | None                                  |
| id                              | 4d2758b8-a480-4ab7-a42f-51a6aab823e4  |
| is_public                       | True                                  |
| name                            | type-1                                |
| os-volume-type-access:is_public | True                                  |
| properties                      | volume_backend_name='unity_backend_1' |
+---------------------------------+---------------------------------------+
stack@newton:~/storops$ openstack volume type create --property volume_backend_name=unity_backend_2 type-2
+---------------------------------+---------------------------------------+
| Field                           | Value                                 |
+---------------------------------+---------------------------------------+
| description                     | None                                  |
| id                              | 0b965050-1025-408b-ae8b-d5236c03df22  |
| is_public                       | True                                  |
| name                            | type-2                                |
| os-volume-type-access:is_public | True                                  |
| properties                      | volume_backend_name='unity_backend_2' |
+---------------------------------+---------------------------------------+

stack@newton:~/storops$ openstack volume show vol-1
+--------------------------------+--------------------------------------+
| Field                          | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2018-11-19T01:42:12.000000           |
| description                    | None                                 |
| encrypted                      | False                                |
| id                             | e267fe89-cd76-41b8-9719-77dc29f64cec |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | vol-1                                |
| os-vol-host-attr:host          | newton@unity_backend_1#Manila_Pool   |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | ca2fbd57936b45bf9abdb536dc152a6d     |
| properties                     |                                      |
| replication_status             | disabled                             |
| size                           | 3                                    |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | available                            |
| type                           | type-1                               |
| updated_at                     | 2018-11-19T01:42:18.000000           |
| user_id                        | 3d2be13807814d80887c0ec869f6be32     |
+--------------------------------+--------------------------------------+

stack@newton:~/storops$ openstack volume show vol-2
+--------------------------------+--------------------------------------+
| Field                          | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2018-11-19T01:44:43.000000           |
| description                    | None                                 |
| encrypted                      | False                                |
| id                             | be01e4cf-9229-4b04-98e8-60a2198ee61a |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | vol-2                                |
| os-vol-host-attr:host          | newton@unity_backend_2#Manila_Pool   |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | ca2fbd57936b45bf9abdb536dc152a6d     |
| properties                     |                                      |
| replication_status             | disabled                             |
| size                           | 3                                    |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | available                            |
| type                           | type-2                               |
| updated_at                     | 2018-11-19T01:44:52.000000           |
| user_id                        | 3d2be13807814d80887c0ec869f6be32     |
+--------------------------------+--------------------------------------+
```

## Cinder version, Unity Cinder version, and Storops version

- Cinder: 9.1.5
- Unity driver: 00.05.03
- Storops: 1.0.0

