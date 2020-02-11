# NVMe and Related


## Storage Evolution
- SRAM
    - Latency: 1X
    - Size of Data: 1X
- DRAM
    - Latency: ~10X
    - Size of Data: ~100X
- 3D XPoint
    - Latency: ~100X
    - Size of Data: ~1000X
- NAND
    - Latency: ~100000X
    - Size of Data: ~1000X
- HDD
    - Latency: ~10 MillionX
    - Size of Data: ~10000X

## NVMe
NVM Express is a standardized high performance software interface for PCI Express Solid State Drives.

## NVMe-of
NVMe over Fabrics: NVMe commands over storage networking fabric.

Remote access to storage - iSCSI vs NVMe-oF
- iSCSI: Network -> iSCSI target -> SCSI -> Block Device Abstraction(BDEV) -> SCSI devices
- NVMe-oF: Network -> NVMe-oF target -> NVMe -> Block Device Abstraction(BDEV) -> NVMe devices

NVMe-oF supports various fabric transports: RDMA, InfiniBand, Fibre Channel, .etc.

![](/forgetful/images/nvme-of-performance.png)

The idea of NVMe-oF is to extend the efficiency of the local NVMe interface over a network fabric, Ethernet or InfiniBand. NVMe commands and data structures are transferred end to end.

NVMe-oF relies on RDMA for performance: bypassing TCP/IP.

## RDMA

Remote Direct Memory Access, is an Advance Transport Protocol (same layer as TCP and UDP)

### Main features
- Remote memory read/write semantics in addition to send/receive
- Kernel bypass / direct user space access
- Full hardware offload
- Secure, channel based IO
  
### Application advantage
- Low latency
- High bandwidth
- Low CPU consumption
