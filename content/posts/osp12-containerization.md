---
title: "OSP12 Containerization Tips"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "osp",
    "osp12",
    "container",
    "containerization",
    "tips",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Install the undercloud according to [official doc](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/13/html/director_installation_and_usage/installing-the-undercloud)

**NOTE: The files/folders used below are under ~/templates**

## Edit the `openstack-tripleo-heat-templates/environments/docker.yaml` to enable the docker image for cinder/manila
```diff
git diff openstack-tripleo-heat-templates/environments/
diff --git a/openstack-tripleo-heat-templates/environments/docker.yaml b/openstack-tripleo-heat-templates/environments/docker.yaml
index 223a179..faef160 100644
--- a/openstack-tripleo-heat-templates/environments/docker.yaml
+++ b/openstack-tripleo-heat-templates/environments/docker.yaml
@@ -51,14 +51,18 @@ resource_registry:
    OS::TripleO::Services::Horizon: ../docker/services/horizon.yaml
    # Because Cinder services currently run on bare metal (see FIXME), Iscsid
    # and Multipathd must run there as well.
-  # OS::TripleO::Services::Iscsid: ../docker/services/iscsid.yaml
-  # OS::TripleO::Services::Multipathd: ../docker/services/multipathd.yaml
+  OS::TripleO::Services::Iscsid: ../docker/services/iscsid.yaml
+  OS::TripleO::Services::Multipathd: ../docker/services/multipathd.yaml
    OS::TripleO::Services::ContainersLogrotateCrond: ../docker/services/logrotate-crond.yaml
    # FIXME: Had to remove these to unblock containers CI. They should be put back when fixed.
-  # OS::TripleO::Services::CinderApi: ../docker/services/cinder-api.yaml
-  # OS::TripleO::Services::CinderScheduler: ../docker/services/cinder-scheduler.yaml
-  # OS::TripleO::Services::CinderBackup: ../docker/services/cinder-backup.yaml
-  # OS::TripleO::Services::CinderVolume: ../docker/services/cinder-volume.yaml
+  OS::TripleO::Services::CinderApi: ../docker/services/cinder-api.yaml
+  OS::TripleO::Services::CinderScheduler: ../docker/services/cinder-scheduler.yaml
+  OS::TripleO::Services::CinderBackup: ../docker/services/cinder-backup.yaml
+  OS::TripleO::Services::CinderVolume: ../docker/services/cinder-volume.yaml
+  # FIXME(peter): Added manila docker services
+  OS::TripleO::Services::ManilaApi: ../docker/services/manila-api.yaml
+  OS::TripleO::Services::ManilaScheduler: ../docker/services/manila-scheduler.yaml
+  OS::TripleO::Services::ManilaShare: ../docker/services/manila-share.yaml
    #
    OS::TripleO::Services::SwiftDispersion: OS::Heat::None
```

## Use command to enable the cinder/manila container image
```bash
openstack --debug overcloud container image prepare \
    --namespace=registry.access.redhat.com/rhosp12 \
    --prefix=openstack- --tag=12.0-20180124.1 \
    --environment-directory /home/stack/templates \
    -e openstack-tripleo-heat-templates/environments/docker.yaml \
    --output-env-file=/home/stack/templates/overcloud_images.yaml
```

## Create a template environment file for vnx cinder

```bash
$ cat /home/stack/templates/emc_customized/custom-vnx-cinder.yaml
volumerameters: # 1
    CinderEnableIscsiBackend: false
    CinderEnableRbdBackend: false
    CinderEnableNfsBackend: false
    NovaEnableRbdBackend: false
    GlanceBackend: file # 2

parameter_defaults:
    ControllerExtraConfig: # 3
    cinder::config::cinder_config:
        vnx1/volume_driver: # 4
            value: cinder.volume.drivers.dell_emc.vnx.driver.VNXDriver
        vnx1/san_ip: # 4
            value: 192.168.1.50
        vnx1/san_login: # 4
            value: sysadmin
        vnx1/san_password: # 4
            value: sysadmin
        vnx1/naviseccli_path: # 4
            value: /opt/Navisphere/bin/naviseccli
        vnx1/initiator_auto_registration: # 4
            value: True
        vnx1/storage_protocol: # 4
            value: iscsi
        vnx2/volume_driver: # 4
            value: cinder.volume.drivers.dell_emc.vnx.driver.VNXDriver
        vnx2/san_ip: # 4
            value: 192.168.1.50
        vnx2/san_login: # 4
            value: sysadmin
        vnx2/san_password: # 4
            value: sysadmin
        vnx2/naviseccli_path: # 4
            value: /opt/Navisphere/bin/naviseccli
        vnx2/initiator_auto_registration: # 4
            value: True
        vnx2/storage_protocol: # 4
            value: iscsi
    cinder_user_enabled_backends: ['vnx1','vnx2'] # 7
```

## Node introspection using [fake_pxe](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/12/html/director_installation_and_usage/appe-Power_Management_Drivers)

**For customer, they need to update the generated `overcloud_images.yaml` to use DELLEMC customized image.**

## Use following command to generate `overcloud_images.yaml`

```bash
# first use below command to get the latest version of OpenStack
$ openstack overcloud container image tag discover \
    --image registry.access.redhat.com/rhosp12/openstack-base:latest \
    --tag-from-label version-release
$ 12.0-20180319.1
# Copy the returned string and set to --tag=<TAG>
# then pull the docker image from the remote to local, to avoid some
# occasional network failure on the corp network

$ openstack overcloud container image prepare \
    --namespace=registry.access.redhat.com/rhosp12 \
    --prefix=openstack- \
    --environment-directory /home/stack/templates \
    -e openstack-tripleo-heat-templates/environments/docker.yaml \
    --tag=12.0-20180319.1 \
    --output-images-file /home/stack/local_registry_images.yaml

$ openstack overcloud container image upload \
    --config-file /home/stack/local_registry_images.yaml --verbose

$ openstack --debug overcloud container image prepare \
    --namespace=192.168.139.1:8787/rhosp12 \
    --prefix=openstack- \
    --tag=12.0-20180319.1 \
    --environment-directory /home/stack/templates \
    -e openstack-tripleo-heat-templates/environments/docker.yaml \
    --output-env-file=/home/stack/templates/overcloud_images.yaml

```

## Import the node yaml

```bash
$ openstack overcloud node import ~/templates/instackenv.json

$ openstack baremetal node list

# before introspect, suggest to create new VMs for discovery
# NOTE, the DHCP/BOOTP only works when issuing following introspect command
$ openstack overcloud node introspect --all-manageable --provide
# Debug the dhcp/bootp process
$ sudo journalctl -l -u openstack-ironic-inspector -u openstack-ironic-inspector-dnsmasq -uopenstack-ironic-conductor -f

$ openstack baremetal introspection interface list [NODE UUID]
$ openstack baremetal introspection interface show [NODE UUID] 

$ openstack baremetal introspection data save _UUID_ | jq .numa_topology
```

## Define the profile/flavor

```bash
# After introspection completes, check the nodes and their assigned profiles:

$ openstack overcloud profiles list
# If you made a mistake in introspection rules, you can delete them all:
$ openstack baremetal introspection rule purge

# update the flavor to fit the compute/control capabilities
(undercloud) [stack@undercloud ~]$ openstack flavor delete control
(undercloud) [stack@undercloud ~]$ openstack flavor delete compute
(undercloud) [stack@undercloud ~]$ openstack flavor create --id auto --ram 8192 --disk 80 --vcpus 8 control

(undercloud) [stack@undercloud ~]$ openstack flavor create --id auto --ram 8192 --disk 80 --vcpus 8 compute

(undercloud) [stack@undercloud ~]$ openstack flavor list
+--------------------------------------+---------------+------+------+-----------+-------+-----------+
| ID                                   | Name          |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+---------------+------+------+-----------+-------+-----------+
| 14772f1a-78de-435e-8e41-ba68a90f06e9 | block-storage | 4096 |   40 |         0 |     1 | True      |
| 3897718f-1ebc-4d57-80a3-41442bbb8c8b | swift-storage | 4096 |   40 |         0 |     1 | True      |
| 68f4e6dc-376b-4350-90d5-e65dd04682b8 | compute       | 8192 |   80 |         0 |     8 | True      |
| 9b823562-7857-41ae-9c1f-b4434d06526f | baremetal     | 4096 |   40 |         0 |     1 | True      |
| b602bb59-862d-4d01-8455-4f7ac1e94388 | networker     | 4096 |   40 |         0 |     1 | True      |
| e473b971-5ccb-421c-8526-c42eabe5f0bd | control       | 8192 |   80 |         0 |     8 | True      |
| ec2b9ea9-c327-49a7-a564-fafe9b6b9984 | ceph-storage  | 4096 |   40 |         0 |     1 | True      |
+--------------------------------------+---------------+------+------+-----------+-------+-----------+

(undercloud) $ openstack baremetal node set --property \
    capabilities='profile:control,boot_option:local' \
    ade5c771-dfc9-4216-b7fa-3f278248c88a
(undercloud) $ openstack baremetal node set --property \
    capabilities='profile:compute,boot_option:local' \
    ec149c12-c9fd-4891-8026-2e5a4dc0875c

```

## Deploy the overcloud by `openstack overcloud deploy`   ​
- Create the `node-info.yaml` for overcloud

```bash
[stack@undercloud templates]$ cat node-info.yaml
parameter_defaults:
    OvercloudControllerFlavor: control
    OvercloudComputeFlavor: compute
    ControllerCount: 1
    ComputeCount: 1
```

- Create the EMC VNX backend for Cinder

```yaml
[stack@undercloud emc_customized]$ cat custom-vnx-cinder.yaml
parameters: # 1
    CinderEnableIscsiBackend: false
    CinderEnableRbdBackend: false
    CinderEnableNfsBackend: false
    NovaEnableRbdBackend: false
#  GlanceBackend: file # 2

parameter_defaults:
    ControllerExtraConfig: # 3
    cinder::config::cinder_config:
        vnx1/volume_driver: # 4
            value: cinder.volume.drivers.dell_emc.vnx.driver.VNXDriver
        vnx1/san_ip: # 4
            value: 192.168.1.50
        vnx1/san_login: # 4
            value: sysadmin
        vnx1/san_password: # 4
            value: sysadmin
        vnx1/naviseccli_path: # 4
            value: /opt/Navisphere/bin/naviseccli
        vnx1/initiator_auto_registration: # 4
            value: True
        vnx1/storage_protocol: # 4
            value: iscsi
        vnx2/volume_driver: # 4
            value: cinder.volume.drivers.dell_emc.vnx.driver.VNXDriver
        vnx2/san_ip: # 4
            value: 192.168.1.50
        vnx2/san_login: # 4
            value: sysadmin
        vnx2/san_password: # 4
            value: sysadmin
        vnx2/naviseccli_path: # 4
            value: /opt/Navisphere/bin/naviseccli
        vnx2/initiator_auto_registration: # 4
            value: True
        vnx2/storage_protocol: # 4
            value: iscsi
    cinder_user_enabled_backends: ['vnx1','vnx2'] # 7

```

- Enable VNX/Cinder service in `vim overcloud-resource-registry-puppet.j2.yaml`


- Deploy the templates

**NOTE: After running below `deploy` command, you will need to power on
the overcloud VMs twice. When to power on? The practice is that run
command `watch -n 5 openstack baremetal node list` in another terminal,
the first time to power on is when you see status `deploy 
wait-callback`. The second time is after the status changes to 
`Active` but the VMs is off. For more detail, refer to [this wiki](
https://oglok.github.io/2016-11-16-Faxe_pxe-Driver-Workflow-on-TripleO/).**

```bash
$ openstack overcloud deploy --templates /usr/share/openstack-tripleo-heat-templates \
    -e /home/stack/templates/node-info.yaml \
    -e /home/stack/templates/overcloud_images.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml \
    -e emc_customized/custom-vnx-cinder.yaml \
    -e /home/stack/templates/openstack-tripleo-heat-templates/environments/docker.yaml \
    -e openstack-tripleo-heat-templates/environments/docker-ha.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/disable-telemetry.yaml \
    -e /home/stack/templates/network-environment.yaml \
    -e emc_customized/custom-containers.yaml \
    --ntp-server 10.244.255.118
```

- During deployment, check the status

```bash
$ openstack stack list --nested

$ openstack stack resource list overcloud

$ openstack stack resource list overcloud --filter status=FAILED 
$ openstack stack resource show <resource>
# from sam's tips
$ openstack stack failures list --long overcloud  
```

**Tips**(https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_openstack_platform/7/html/director_installation_and_usage/sect-fake_driver)

```markdown
Additional Notes

- This driver does not use any authentication details because it does not control power management.
- Edit the /etc/ironic/ironic.conf file and add fake_pxe to the enabled_drivers option to enable this driver.
- When performing introspection on nodes, manually power the nodes after running the openstack baremetal introspection bulk start command.
- When performing Overcloud deployment, check the node status with the ironic node-list command. Wait until the node status changes from deploying to deploy **wait call-back** and then manually power(or reset) the nodes.
- After the Overcloud provisioning process completes, reboot the nodes. To check the completion of provisioning, check the node status with the ironic node-list command, wait until the node status changes to active, then manually reboot all Overcloud nodes.
```

- Manually change the configs of the containers

```bash
sudo vi /var/lib/config-data/puppet-generated/nova_libvirt/etc/nova/nova.conf
```

## If deploy failed, clean the stack 

```bash
# undeloy
(undercloud) [stack@undercloud templates]$ openstack baremetal node undeploy  665c5b4a-8496-4929-80ed-f0859a225775
(undercloud) [stack@undercloud templates]$ openstack baremetal node undeploy  3d29830f-712f-4a3e-a87b-4b64eb9ac3b8
# delete stack
$ openstack  stack  delete overcloud

# Delete plan
$ openstack overcloud plan delete overcloud

# Delete the baremetal nodes
$ openstack baremetal node delete  b9756047-e342-4f16-bc99-df2f503b02ba fe2aa09c-7ee5-4f99-966a-8e7dbec835b5
```

## Troubleshooting
- Failed to create instance
    
    The error message is like:
    ```bash
    2018-04-10 23:13:14.223 1 ERROR nova.compute.manager [instance: 2a7c6019-62d5-4ce6-a753-1781d15aedf2] libvirtError: internal error: process exited while connecting to monitor: libvirt:  error : cannot execute binary /usr/libexec/qemu-kvm: Permission denied
    ```

    The solution: https://www.thegeekdiary.com/how-to-disable-or-set-selinux-to-permissive-mode/