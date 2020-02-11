# OSP13 Deployment TripleO Tips


## Verify Quick Fixes

### Executes below commands on undercloud
```bash
# Enter OpenStack env
source ~/stackrc

# Install the rpms which contain the latest fixes
sudo rpm -Uvh \
    openstack-tripleo-heat-templates-8.0.3-2.el7ost.noarch.rpm \
    puppet-tripleo-8.3.4-2.el7ost.noarch.rpm \
    puppet-cinder-12.4.1-0.20180628102250.641e036.el7ost.noarch.rpm \
    puppet-manila-12.4.0-2.el7ost.noarch.rpm

# Upload the installed rpms
# This step helps update the puppet modules when executing the
# overcloud deployment
cp -r /usr/share/openstack-puppet/modules/tripleo/ \
    /usr/share/openstack-puppet/modules/cinder/ \
    /usr/share/openstack-puppet/modules/manila/ ~/puppet-modules/

upload-puppet-modules --directory puppet-modules/

# Clean up the old deployment
sh /home/stack/templates/cleanup_stack.sh

# On ESXi, stop vms (overcloud-controller-0, overcloud-compute-0)

# Node registration
openstack overcloud node import ~/templates/instackenv.json

# Tagging nodes into profiles (maybe no need)
nodes=()
for i in $(openstack baremetal node list -c UUID -f value); do nodes+=($i); done
openstack baremetal node set \
    --property capabilities='profile:compute,boot_option:local' ${nodes[0]}
openstack baremetal node set \
    --property capabilities='profile:control,boot_option:local' ${nodes[1]}

# Node inspection
openstack overcloud node introspect --all-manageable --provide

# On ESXi, start vms
# Once node inspection successfully, stop vms

# Deploy overcloud
openstack overcloud deploy --templates /usr/share/openstack-tripleo-heat-templates \
    -e /home/stack/templates/node-info.yaml \
    -e /home/stack/templates/local_registry/overcloud_images.yaml \
    -e /home/stack/templates/network-environment.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/disable-telemetry.yaml \
    -e /home/stack/templates/emc_customized/emc_containerized.yaml \
    -e /home/stack/templates/emc_customized/cinder-dellemc-vnx-config.yaml \
    -e /home/stack/templates/emc_customized/manila-vnx-config.yaml \
    --timeout 600 \
    --ntp-server 10.228.254.10

# On ESXi, start vms when the node status become “waiting-callback”
# vms will be shutdown automatically
# Start vms when the node status change from “deploying” to “active”
# Tips: run below command in another terminal
watch -n 5 openstack baremetal node list
```
