# OpenStack Multi-attach

## References

https://docs.openstack.org/nova/latest/admin/manage-volumes.html#volume-multi-attach
https://docs.openstack.org/cinder/latest/admin/blockstorage-volume-multiattach.html
http://specs.openstack.org/openstack/cinder-specs/specs/ocata/add-new-attach-apis.html

## Versions Needed

- The minimum required compute API microversion for attaching a multiattach-capable volume to more than one server is **2.60**.
- Cinder **12.0.0 (Queens)** or newer is required.
- The ``nova-compute`` service must be running at least **Queens** release level code (17.0.0) and the hypervisor driver must support attaching block storage devices to more than one guest. Refer to [Feature Support Matrix](https://docs.openstack.org/nova/latest/user/support-matrix.html) for details on which compute drivers support volume multiattach.
- When using the libvirt compute driver, the following native package versions determine multiattach support:
    - ``libvirt`` must be greater than or equal to **3.10**, or
    - ``qemu`` must be less than **2.10**
- Swapping an in-use multiattach volume is not supported (this is actually controlled via the block storage volume retype API).

## Commands

```bash
export OS_COMPUTE_API_VERSION=2.60

# Create volume type supporting multiattach
openstack volume type create --property multiattach="<is> True" multiattach

# Create multiattach volume
openstack volume create --size 1 --type multiattach vol-multiattach

# Create a vm with two volumes, the second one is the multiattach volume
VOL_ID=$(openstack volume show vol-multiattach -c id -f value)
CIRROS_IMG=$(openstack image show cirros-0.3.5-x86_64-disk -c id -f value)
NET_ID=$(openstack network show net-private -c id -f value)
nova boot --nic net-id=$NET_ID --flavor 1 \
    --block-device id=$CIRROS_IMG,source=image,dest=volume,size=1,bootindex=0,shutdown=remove,tag=cirros \
    --block-device id=$VOL_ID,source=volume,dest=volume,bootindex=-1,tag=multiattach \
    --config-drive True --poll vm1

# Check the second volume is multiattach volume
sudo virsh list
sudo virsh dumpxml 4
openstack console log show vm1

# Allocate a floating ip to the vm
VM1=$(openstack server show vm1 -c id -f value)
PORT1=$(openstack port list --device-id $VM1 -c id -f value)
FLOATINGIP1=$(openstack floating ip create --port $PORT1 pulic -c floating_ip_address -f value)

# SSH to the vm to check the second volume
ssh cirros@FLOATINGIP1
# Or
sudo ip netns exec qrouter-02c168e8-ec5f-4b64-b575-7f6e70f19c45 ssh cirros@10.10.1.3

    blkid

    # Mount the multiattach volume
    sudo mkdir -p /mnt/volume
    sudo mkfs -t ext4 /dev/vdb
    sudo mount /dev/vdb /mnt/volume

    # Write some file into the volume
    sudo sh -c "echo 'hello from $(hostname)' > /mnt/volume/hello.txt; sync"
    sudo cat /mnt/volume/hello.txt
    sudo umount /mnt/volume

# Create a second vm to attach the multiattach volume
openstack server create --flavor 1 --image $CIRROS_IMG --nic net-id=$NET_ID --wait vm2
VM2=$(openstack server show vm2 -c id -f value)

openstack server show -c status -c volumes_attached vm2
openstack server add volume vm2 vol-multiattach
openstack server show -c status -c volumes_attached vm2

stack@rocky:~/cinder$ openstack volume show $VOL_ID -c attachments -f json
{
  "attachments": [
    {
      "server_id": "873411ed-1033-40a6-aff6-cb67988ed6f9",
      "attachment_id": "242037e3-cff0-4694-8d19-e5559e091677",
      "attached_at": "2018-07-13T07:45:12.000000",
      "host_name": "rocky",
      "volume_id": "224cc34a-3c07-4644-9698-dd1d3f40142f",
      "device": "/dev/vdb",
      "id": "224cc34a-3c07-4644-9698-dd1d3f40142f"
    },
    {
      "server_id": "729aec10-2dc3-4635-bbfb-0be16b69d7bf",
      "attachment_id": "caaa9204-f4b8-453f-b7b1-22af76899f20",
      "attached_at": "2018-07-13T06:57:18.000000",
      "host_name": "rocky",
      "volume_id": "224cc34a-3c07-4644-9698-dd1d3f40142f",
      "device": "/dev/vdb",
      "id": "224cc34a-3c07-4644-9698-dd1d3f40142f"
    }
  ]
}

# SSH to the second vm to check the second volume
ssh cirros@FLOATINGIP2
# Or
sudo ip netns exec qrouter-02c168e8-ec5f-4b64-b575-7f6e70f19c45 ssh cirros@10.10.1.10

  sudo mkdir /mnt/volume
  sudo mount /dev/vdb /mnt/volume

  sudo sh -c "echo 'hello from $(hostname)' >> /mnt/volume/hello.txt; sync"

```

## Troubleshooting

### Fix the volume attached to a deleted VM from mysql
```sql
mysql> update volumes set attach_status='detached',status='available' where id = '304d4c71-9817-4b83-8677-82f411cb5f22';
```
