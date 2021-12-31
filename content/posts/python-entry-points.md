---
title: "Python Module Entry Points"
date: 2020-09-22T15:53:50+08:00
lastmod: 2020-09-22T15:53:59+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "python",
    "entry points",
    "pkg_resources",
    "stevedore",
    "extension manager",
    "extension",
    "plugin",
]

comment: true
toc: true
auto_collapse_toc: true
---

# Python Module Entry Points

You could use `pkg_resources` to get the full list of entry points.
```python
for ep in pkg_resources.iter_entry_points('manila.scheduler.filters'):
    print(ep.name, ep.dist.location, ep.module_name)

# Sample output
AvailabilityZoneFilter /opt/stack/manila manila.scheduler.filters.availability_zone
ShareGroupReplicationFilter /opt/stack/manila manila.scheduler.filters.share_group_filters.share_group_replication
```

`manila.scheduler.filters` here is called group or namespace. You can get it via package name as below:
```python
for e in pkg_resources.working_set:
    if e.key == 'manila':
        print(e._ep_map.keys())

# Sample output
dict_keys(['console_scripts', 'manila.scheduler.filters', 'manila.scheduler.weighers', 'manila.share.drivers.dell_emc.plugins', 'manila.tests.scheduler.fakes', 'oslo.config.opts', 'oslo.config.opts.defaults', 'oslo.policy.enforcer', 'oslo.policy.policies', 'oslo_messaging.notify.drivers', 'wsgi_scripts'])
```

# Stevedore Extension/Plugin Manager

Stevedore extension manager depends on entry points heavily.

1. For packages installed in develop mode, the entry points info is stored under like `/opt/stack/manila/manila.egg-info/entry_points.txt`.
2. For others, you could find them in `/usr/local/lib/python3.6/dist-packages/websockify-0.9.0.dist-info/entry_points.txt`.