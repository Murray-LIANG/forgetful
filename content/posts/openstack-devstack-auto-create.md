---
title: "Some Auto Setup by Devstack"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "devstack",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Neutron network part

```bash
openstack --os-cloud devstack-admin --os-region RegionOne subnet pool create \
    shared-default-subnetpool-v4 --default-prefix-length 26 \
    --pool-prefix 10.0.0.0/24 --share --default -f value -c id

openstack --os-cloud devstack-admin --os-region RegionOne subnet pool create \
    shared-default-subnetpool-v6 --default-prefix-length 64 \
    --pool-prefix fd6c:b20f:1a47::/56 --share --default -f value -c id

openstack --os-cloud devstack-admin --os-region RegionOne network create \
    --project 61f69c1beb974772967eba71b6b86e60 private

openstack --os-cloud devstack-admin --os-region RegionOne subnet create \
    --project 61f69c1beb974772967eba71b6b86e60 --ip-version 4 \
    --subnet-pool ccf77a09-8652-4356-af6e-719a7a6cd29f \
    --network 2ea9a24f-b7d6-4557-9937-5032611018b5 private-subnet

openstack --os-cloud devstack-admin --os-region RegionOne subnet create \
    --project 61f69c1beb974772967eba71b6b86e60 --ip-version 6 \
    --subnet-pool 572e00ee-55c3-4a2d-ac84-b870b27f540f \
    --ipv6-ra-mode slaac --ipv6-address-mode slaac \
    --network 2ea9a24f-b7d6-4557-9937-5032611018b5 ipv6-private-subnet

openstack --os-cloud devstack-admin --os-region RegionOne router \
    create --project 61f69c1beb974772967eba71b6b86e60 router1

openstack --os-cloud devstack-admin --os-region RegionOne network \
    create public --external --default --provider-network-type flat \
    --provider-physical-network public

# Add `private-subnet` to `router1`
openstack --os-cloud devstack-admin --os-region RegionOne router add subnet \ 
    3933a2cc-16ba-43f2-9d52-8f6839e12f45 8071de26-e7cb-452c-a467-f8b1cb2727c5

openstack --os-cloud devstack-admin --os-region RegionOne subnet \
    create --ip-version 4 \
    --network 3416d82e-d55d-4725-899e-0495ebc691fb \
    --subnet-range 172.24.4.0/24 --no-dhcp public-subnet

# Set external gateway of `router1` to network `public`
openstack --os-cloud devstack-admin --os-region RegionOne router set \
    --external-gateway 3416d82e-d55d-4725-899e-0495ebc691fb \
    3933a2cc-16ba-43f2-9d52-8f6839e12f45

openstack --os-cloud devstack-admin --os-region RegionOne port list \
    -c 'Fixed IP Addresses' --device-owner network:router_gateway

# Add `ipv6-private-subnet` to `router1`
openstack --os-cloud devstack-admin --os-region RegionOne router add subnet \
    3933a2cc-16ba-43f2-9d52-8f6839e12f45 3ced074c-3df3-4f28-b553-a4805a04b0c8

openstack --os-cloud devstack-admin --os-region RegionOne subnet \
    create --ip-version 6 --gateway 2001:db8::2 \
    --network 3416d82e-d55d-4725-899e-0495ebc691fb \
    --subnet-range 2001:db8::/64 --no-dhcp ipv6-public-subnet
```