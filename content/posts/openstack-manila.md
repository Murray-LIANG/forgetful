---
title: "OpenStack Manila"
date: 2020-05-28T16:43:24+08:00
lastmod: 2020-05-28T16:43:33+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "manila",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Below commands are used by elab.
```console
$ # remove admin_subnet and admin_net created by devstack
$ openstack subnet delete admin_subnet
$ openstack network delete admin_net

$ VLAN_ID=50
$ NETWORK_NAME=unity-net
$ SUBNET_NAME=unity-subnet
$ SUBNET_RANGE=$(grep '^FIXED_RANGE' /tmp/dg-local.conf|head -1|cut -f2 -d=)
$ 
$ # create datamover network
$ NETWORK_ID=$(openstack network create --share -f value -c id --provider-network-type vlan  --provider-physical-network public_eth1 --provider-segment $VLAN_ID $NETWORK_NAME)
$ SUBNET_ID=$(openstack subnet create -f value -c id --ip-version 4 --network $NETWORK_NAME --subnet-range $SUBNET_RANGE $SUBNET_NAME)
$
$ # create security service with Domain Controller information, make sure this DC is also in the VLAN
$ SECURITY_NAME=unity-ad
$ AD_IP=12.0.0.254
$ AD_DOMAIN=openstackci.elab
$ manila security-service-create --dns-ip $AD_IP --domain $AD_DOMAIN --server vnxadsvr --user Administrator --password '#1Danger0us' --name $SECURITY_NAME active_directory 
$ #SECURITY_ID=$(manila security-service-list|grep adserver|awk '{print $2}')
$
$ # create share network and associate this share network with previous security service.
$ SHARENET_NAME=unity-sharenet
$ manila share-network-create --neutron-net-id $NETWORK_ID --neutron-subnet-id $SUBNET_ID --name $SHARENET_NAME
$ SHARENET_ID=$(manila share-network-list|awk -v name=$SHARENET_NAME '{if($4==name){print $2}}')
$ manila share-network-security-service-add $SHARENET_NAME $SECURITY_NAME

$ #create a share to keep share server from being removed. 
$ manila create --share-network $SHARENET_ID --name ws1keep NFS 10
    
$ #2. Append [share] section for tempest.conf
$ sudo sed -i "s/%SHARE_NETWORK_ID%/$SHARENET_ID/g" /opt/stack/new/tempest/etc/tempest.conf
```

## Misc

### Enable manila client bash completion

```console
$ sudo cp /opt/stack/python-manilaclient/tools/manila.bash_completion /etc/bash_completion.d
```