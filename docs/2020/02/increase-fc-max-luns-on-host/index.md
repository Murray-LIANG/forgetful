# Increase the Max LUNs of FC Host Adapter


In our environment, `Emulex LightPulse Fibre Channel Host Adapter (lpfc)` is used. For other
adapters like `QLogic Corp Fibre Channel to PCI Express HBA (qla driver)` and
`Cisco Systems Inc VIC FCoE Host Adapter (fnic driver)`, please refer to
https://access.redhat.com/solutions/26017 or ask for help from RedHat experts directly.

## Default value of max LUNs
For lpfc, the default value of max LUNs is 255.
```bash
stack@ubuntu-server7:~$ cat /sys/class/scsi_host/host9/lpfc_max_luns
255
```

If a lun is with HLU number greater than 255, then the host cannot see this lun.
And you could see below messages via `dmesg`:
```bash
lun11096 has a LUN larger than allowed by the host adapter
```

## Host OS is Ubuntu

1. Edit `/etc/initramfs-tools/modules`, add below lines:
```bash
scsi_mod max_luns=16384
lpfc lpfc_max_luns=16384
```
2. Update the initramfs:
```bash
sudo update-initramfs -u
```

3. Reboot the host.

4. Verify the change effective:
```bash
stack@ubuntu-server7:~$ sudo cat /sys/module/lpfc/parameters/lpfc_max_luns
16384
```

## Host OS is RedHat

Refer to https://access.redhat.com/solutions/26017.

The difference is just using different configuration files and the way of generating new initramfs and booting using new initramfs.
