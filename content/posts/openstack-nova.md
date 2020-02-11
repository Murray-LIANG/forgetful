---
title: "OpenStack Nova"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "nova",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Services
- nova-api
- nova-scheduler
- nova-compute
- nova-conductor
- Hypervisor

## Where the services run
Generally, the api, scheduler, conductor are running on `Control Node`.
compute, Hypervisor are running on `Compute Node`.

## General flow
1. User (End user or other services) sends requests to `nova-api` to
create a VM instance.
2. `nova-api` sends message to `Message Queue` to ask `nova-scheduler` to
schedule a `Compute Node` to create the VM.
3. `nova-scheduler` gets the message and pick a `Compute Node` via a filter
algorithm, then sends message to let that `Compute Node` (A) to create VM.
4. `Compute Node` (A) gets the message, uses the `Hypervisor` to boot the VM.
5. Any update to the DB will be executed by messaging the `nova-conductor`.

## Config hypervisor driver
In `/etc/nova/nova.conf`, set `compute_driver` to `libvirt.libvirtDriver`.

## Scheduler
User specifies the `Flavor` to create the VM. `nova-scheduler` will choose
the `Compute Node`.

`Flavor` defines VCPU, RAM, Disk and Metadata.

In `/etc/nova/nova.conf`, `scheduler_driver`, `scheduler_available_filters`
and `scheduler_default_filter` settings affect the behavior of scheduler.

### Filter scheduler
The default scheduler.
Two steps:
1. Filter to select the `Compute Node` meeting the requirements.
2. Weight the `Compute Node` of step 1.

```
scheduler_driver = nova.scheduler.filter_scheduler.FilterScheduler

# Available filters
scheduler_available_filters = nova.scheduler.filters.all_filters

# Filte in below order
scheduler_default_filters = RetryFilter, AvailabilityZoneFilter,
                            RamFilter, DiskFilter, ComputeFilter,
                            ComputeCapabilitiesFilter, ImagePropertiesFilter,
                            ServerGroupAntiAffinityFilter, ServerGroupAffinityFilter
```