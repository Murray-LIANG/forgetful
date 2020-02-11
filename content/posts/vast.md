---
title: "VAST"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "vast",
    "nvme",
]

comment: true
toc: true
auto_collapse_toc: true
---

VAST Data has found a way to collapse all storage tiers onto one with decoupled **compute nodes** using **NVMeoF** to access 2U **databoxes** filled with **QLC flash** data drives and **Optane XPoint** metadata and write-staging drives, with up to 2PB or more of capacity after data reduction.

**3D XPoint** memory is non-volatile, faster than NAND but slower than DRAM.

There is a **shared-everything** architecture embodied in separate and independently **scalable** **compute nodes** in a loosely-coupled cluster running Universal Filesystem logic. These link across an NVMe link to **databoxes (DBOX3 storage nodes)** which are dumb boxes filled with **flash drives** for data and **Optane XPoint** for metadata.

In VAST's scheme there are **compute nodes** and **databoxes** - high-availability DF-5615 NVMe JBOFs, connected by NVMe-oF running across Ethernet or InfiniBand. The x86 servers run VAST Universal File System (UFS) software packaged in Docker containers. The 2U databoxes are dumb, and contain 18 * 960GB of Optane 3D XPoint memory and 583TB of NVMe QLC (4bits/cell) SSD (38 * 15.36TB). **Data is striped across the databoxes and their drives with global erasure coding**.

![](/forgetful/images/vast-compute-nodes-storage-nodes.jpg)

**The XPoint is not a tier. It is used for storing metadata and as an incoming write buffer.**

All writes are **atomic and persisted in XPoint**; there is **no DRAM cache**. **Compute nodes are stateless** and see all the storage in the databoxes. If one compute node fails another can pick up its work with no data loss.

https://www.vastdata.com/app/pdf/datasheet.pdf

## DASE
Disaggregated shared-everything architecture

DASE storage architecture breaks from the idea that scalable storage needs to be built from shared-nothing clusters. When servers can be disaggregated from storage, everything is better:
- Servers can be **stateless**, and failure of any server never require data reconstruction across a network.
- Servers can be **loosely coupled** in a scalable cluster - each operating independently while all accessing a shared, global namespace, thereby eliminating cluster cross-talk and enabling virtually limitless scale.
- New, global algorithms can be implemented to achieve game-changing levels of efficiency and resilience.

## Three Options to Deploy
- Full appliance: VAST Quad Server Chassis + Active/Active NVMe Enclosure.
- Software defined: Container software + Active/Active NVMe Enclosure.
- Software only: Run VAST software on Certified QLC hardware.

## Conceptual Diagram

https://glennklockwood.blogspot.com/2019/02/vast-datas-storage-system-architecture.html

![](/forgetful/images/vast-conceptual-diagram.png)

### JBOFs (Just-Bunch-Of-Flash, data boxes or HA enclosures).

These things are what contain the storage media itself.
    
JBOFs are dead simple and **their only job is to expose each NVMe device attached to them as an NVMeoF target**.
- 2x embedded active/active servers, each with two Intel CPUs and the necessary hardware to support failover.
- 4x 100 gigabit NICs, either operating using RoCE or InfiniBand.
- 38x 15.36 TB U.2 SSD carriers.
- 18x 960 GB U.2 Intel Optane SSDs.

**They are not RAID controllers nor do they do any data motion between the SSDs they host. They literally serve each device out to the network and that's it**.

### I/O servers (compute nodes or cnodes) These things are what HPC cluster compute nodes talk to to perform I/O via NFS or S3.

Here are where the magic happens, and they are physically discrete servers that
1. share the same SAN fabric as the JBOFs and speak NVMeoF on one side, and
2. share a network with client nodes and talk NFS on the other side.

These I/O servers are completely **stateless**; all the data is stored in the JBOFs. The I/O servers have **no cache**; their job is to run NFS requests from HPC compute nodes into NVMeoF transfers to JBOFs. Specifically, they perform the following functions:
1. Determine which NVMeoF device to talk to to serve an incoming I/O request from NFS client. This is done **using a hashing function**.
2. Enforce file permissions, ACLs, and everything else that an NFS client would expect.
3. Transfer data to/from SSDs, and transfer data to/from 3D XPoint drives.
4. Transfer data between SSDs and 3D XPoint drives.
5. Perform "global compression", rebuilds from parity and other maintenance tasks.

## System Composition

Because I/O servers are stateless and operate independently, they can be **dynamically added and removed** from the system at any time to **increase or decrease the I/O processing power** available to clients. VAST's position is that the peak I/O performance to the JBOFs is virtually always **CPU limited** since the data path between CPUs (in the I/O servers) and the storage devices (in JBOFs) uses **NVMeoF**. This is a reasonable assertion since NVMeoF is extremely efficient at moving data as a result of its use of **RDMA** ans simple block-level access semantics.

At the same time, this design requires that every I/O server be able to communicate with every SSD in the entire VAST system via NVMeoF. This means that each I/O server **mounts every SSD** at the same time; in a relatively small two-JBOF system, this results in 112x ( (38+18)*2 ) NVMe targets on every I/O server. This poses two distinct challenges:
1. From a implementation standpoint, this is pushing **the limits of how many NVMeoF targets a single Linux host can effectively manage** in practice. For example, a 10 PB VAST system will have over 900 NVMeoF targets mounted on every single I/O server. This scale will exercise pieces of Linux kernel in ways it was never designed to be used.
2. From a fundamental standpoint, this puts tremendous pressure on the storage network.

## Shared-everything Consistency

**NOT FULLY UNDERSTAND THIS SECTION**

Mounting every block device on every server may also sound like anathema to anyone familiar with block-based SANs, and generally speaking, it is. NVMeoF (and every other block-level protocol) does not really have locking, so **if a single device is mounted by two servers, it is up to those servers to communicate with each other to ensure they aren't attempting to modify the same blocks at the same time**. **Typical shared-block** configurations manage this by simply **assigning exclusive ownership** of each drive to a single server and **relying on heartbeating or quorum** (e.g., in HA enclosures or GPFS) **to decide when to change a drive's owner**.

VAST can avoid a lot of these problems by **simply not caching any I/Os on the I/O servers** and instead passing NFS requests through as NVMeoF requests. This is not unlike how parallel file systems like PVFS (now OrangeFS) avoided the lock contention problem; **not using caches dramatically reduces the window of time during which two conflicting I/Os can collide**. VAST also **claws back some of the latency penalties of doing this sort of direct I/O by issuing all writes to nonvolatile memory instead of flash**; this will be discussed later.

For the rare cases where two I/O servers are asked to change the same piece of data **at the same time** though, there is a mechanism by which **an extent of a file (which is on the order of 4KiB) can be locked**. I/O servers will flip a lock bit for that extent in the JBOF's memory using an atomic RDMA operation before issuing an update to serialize overlapping I/Os to the same byte range.

VAST also uses **redirect-on-write** to ensure that writes are always consistent. If a JBOF fails before an I/O is complete, presumably any outstanding locks evaporate since they are resident only in RAM. Any changes that were in flight simply get lost **because the metadata structure that describes the affected file's layout only points to updated extents after they have been successfully written**. Again, the redirect-on-write is achieved using an atomic RDMA operation, so data is always consistent. VAST does not need to maintain a write journal as a result.

### Summary
- No cache
- Issue writes to nonvolatile memory
- Redirect-on-write


## Write Path Flow 

https://glennklockwood.blogspot.com/2019/02/vast-datas-storage-system-architecture.html
