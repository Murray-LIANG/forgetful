---
title: "OpenStack Devstack Tips"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "devstack",
    "tips",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Clean Up
1. Unstack and clean up the databases, .etc.

    `~/devstack/clean.sh` 

2. Uninstall all python packages via `pip`.
   
    ```bash
    sudo pip freeze | grep -v "git+https" | \
        xargs sudo pip uninstall -y
    
    for i in cinder glance keystone neutron nova;
        do bash -c "cd /opt/stack/$i && sudo python setup.py develop --uninstall";
    done
    ```

3. [Optional] Remove all installed binaries and source codes.

    `sudo rm -rf /opt/stack /usr/local/bin`

## Set Up Devstack

1. `git clone https://git.openstack.org/openstack-dev/devstack`, then checkout to the desired release (branch). 
2. Modify `GIT_BASE` in `stackrc` file. Use `http` instead of `https`.
    ```diff
     # Base GIT Repo URL
     # Another option is https://git.openstack.org
    -GIT_BASE=${GIT_BASE:-git://git.openstack.org}
    +GIT_BASE=${GIT_BASE:-http://git.openstack.org}
    ```
3. [Optional, try this when meet `git clone cannot find branch` error] Update the branches' name in `stackrc` file.
    ```diff
    # block storage service
    CINDER_REPO=${CINDER_REPO:-${GIT_BASE}/openstack/cinder.git}
    -CINDER_BRANCH=${CINDER_BRANCH:-stable/newton}
    +CINDER_BRANCH=${CINDER_BRANCH:-newton-eol}
    
    # image catalog service
    GLANCE_REPO=${GLANCE_REPO:-${GIT_BASE}/openstack/glance.git}
    -GLANCE_BRANCH=${GLANCE_BRANCH:-stable/newton}
    +GLANCE_BRANCH=${GLANCE_BRANCH:-newton-eol}
    
    # heat service
    HEAT_REPO=${HEAT_REPO:-${GIT_BASE}/openstack/heat.git}
    -HEAT_BRANCH=${HEAT_BRANCH:-stable/newton}
    +HEAT_BRANCH=${HEAT_BRANCH:-newton-eol}
    
    # django powered web control panel for openstack
    HORIZON_REPO=${HORIZON_REPO:-${GIT_BASE}/openstack/horizon.git}
    -HORIZON_BRANCH=${HORIZON_BRANCH:-stable/newton}
    +HORIZON_BRANCH=${HORIZON_BRANCH:-newton-eol}
    
    # unified auth system (manages accounts/tokens)
    KEYSTONE_REPO=${KEYSTONE_REPO:-${GIT_BASE}/openstack/keystone.git}
    -KEYSTONE_BRANCH=${KEYSTONE_BRANCH:-stable/newton}
    +KEYSTONE_BRANCH=${KEYSTONE_BRANCH:-newton-eol}
    
    # neutron service
    NEUTRON_REPO=${NEUTRON_REPO:-${GIT_BASE}/openstack/neutron.git}
    -NEUTRON_BRANCH=${NEUTRON_BRANCH:-stable/newton}
    +NEUTRON_BRANCH=${NEUTRON_BRANCH:-newton-eol}
    
    # neutron fwaas service
    NEUTRON_FWAAS_REPO=${NEUTRON_FWAAS_REPO:-${GIT_BASE}/openstack/neutron-fwaas.git}
    -NEUTRON_FWAAS_BRANCH=${NEUTRON_FWAAS_BRANCH:-stable/newton}
    +NEUTRON_FWAAS_BRANCH=${NEUTRON_FWAAS_BRANCH:-newton-eol}
    
    # compute service
    NOVA_REPO=${NOVA_REPO:-${GIT_BASE}/openstack/nova.git}
    -NOVA_BRANCH=${NOVA_BRANCH:-stable/newton}
    +NOVA_BRANCH=${NOVA_BRANCH:-newton-eol}
    
    # object storage service
    SWIFT_REPO=${SWIFT_REPO:-${GIT_BASE}/openstack/swift.git}
    -SWIFT_BRANCH=${SWIFT_BRANCH:-stable/newton}
    +SWIFT_BRANCH=${SWIFT_BRANCH:-newton-eol}
    ```

4. [Optional, try this when meet `pip installation` error] Update pip version in `tools/cap-pip.txt`
    ```diff
    -pip!=8
    +pip!=8,<10
    ```
5. Add `local.conf` file. Download https://eos2git.cec.lab.emc.com/liangr/corp-notes/blob/master/openstack-devstack-local.conf and save it to `~/devstack/local.conf`, then modify the `HOST_IP` to the IP address of VM.

6. Run `~/devstack/stack.sh` to start the build up of devstack.