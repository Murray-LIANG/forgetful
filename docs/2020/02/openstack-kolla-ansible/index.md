# OpenStack Deployment using Kolla Ansible

## Network layout
### API network
- eth3 of ubuntu-serverX: 192.168.4.X/24
- Internal VIP: 192.168.4.254

### External network
- eth0 of ubuntu-serverX.
    
    **NOTE: 172.30.X.X/16 subnet is used as the floating IP of OpenStack VMs. Do NOT use 192.168.1.X/24 subnet. Refer to `Network settings`**
- External VIP: 192.168.1.254, used to access Horizon from `Windows Client`. Because the `eth0` of Neutron node (aka. `ubuntu-server3`) is bound to `br-ex`, IP 192.168.1.254 need to be set to `br-ex` manually after deploy, so as IP 192.168.1.3.

### Tanent network
- eth2 - VXLAN (prefered)
- eht1 - VLAN

## Network settings on host ubuntu-serverX
### Gateway ubuntu-server1
1. Add IP 172.30.0.1: `sudo ip addr add 172.30.0.1/16 brd 172.30.255.255 dev eth`
1. Add MASQUERADE rules:
    ```bash
    $ sudo iptables -t nat -A POSTROUTING -s 192.168.4.0/255.255.255.0 -o eth0 -j MASQUERADE
    $ sudo iptables -t nat -A POSTROUTING -s 172.30.0.0/16 -o eth0 -j MASQUERADE
    
    # check the rules via: iptables -L -t nat 
    ```

### Neutron node ubuntu-server3
1. Edit `/etc/network/interface` to remove 192.168.1.3 IP from eth0.

### Ubuntu client and Windows client (Re-configure manually after reboot)
1. Add IP 172.30.0.32 for ubuntu-client1: `sudo ip addr add 172.30.0.32/16 brd 172.30.255.255 dev enp1s0f1`
1. Add IP 172.30.0.33 for Windows client.

## Ubuntu-serverX settings before deploy
**Use `ubuntu-server8` to deploy OpenStack**

### ubuntu-server8 configuration
- Add `ubuntu-serverX` to `/etc/hosts`. Use **192.168.4.X**.
- Gen ssh key and configure ssh via the key (passwordless)
    ```bash
    $ ssh-keygen -t rsa -b 4096 -C "ubuntu-server8"
    $ eval $(ssh-agent -s)
    $ ssh-add ~/.ssh/id_rsa
    # add the public key to /home/stack/.ssh/authorized_keys of 
    # other ubuntu-serverX
    ```

### Setup NTP on all ubuntu-serverX
Add below lines to `/etc/ntp.conf`
```
pool 10.244.255.118
pool 10.228.254.10
```

```bash
$ sudo service ntp stop && sudo ntpd -gq && sudo service ntp start
```

## Customize kolla for Dell

The official repo was forked to https://github.com/Murray-LIANG/kolla.

The branch is `dell-customize`. Need to rebase with the latest `master` of official kolla before do `kolla-build`.


## Start local docker registry

### Set up the registry
Use the script: `https://github.com/Murray-LIANG/kolla/blob/master/tools/start-registry`

Or the command line:

**NOTE: Use port 5050 on host to avoid conflict with the port used by keystone.**
``` bash
$ docker run -d \
    --name registry \
    --restart=always \
    -p 5050:5000 \
    -v registry:/var/lib/registry \
  registry:2
```

### Set up the GUI to view the images in local registry
``` bash
$ docker run -d -p 8080:80 \
    --restart=always \
    -e ENV_DOCKER_REGISTRY_HOST=192.168.1.8 \
    -e ENV_DOCKER_REGISTRY_PORT=5050 \
  konradkleine/docker-registry-frontend:v2
```

## Install virtualenv on ubuntu-server8
Install to folder `~/pyvenv`, then run commands:
```bash
$ cd ~/git
$ git clone https://murray-liang@github.com/Murray-LIANG/kolla.git
$ git clone https://github.com/openstack/kolla-ansible
$ source ~/pyvenv/bin/activate
# Use the latest master version of kolla and kolla-ansible
$ cd ~/git/kolla && pip install .
$ cd ~/git/kolla-ansible && pip install .
```

## Build docker images via kolla-build
TODO

## Customize the OpenStack service configuration
Refer to https://docs.openstack.org/kolla-ansible/latest/admin/advanced-configuration.html#openstack-service-configuration-in-kolla

**TODO: push all the configuration to git.**

## Deploy

```bash
# enter virtualenv
$ source ~/pyvenv/bin/activate
$ kolla-ansible -i ./multinode bootstrap-servers
$ kolla-ansible -i ./multinode deploy
$ kolla-ansible -v -i ./multinode  post-deploy

# Sample of openstack cli
$ docker exec kolla_toolbox openstack --os-interface internal \
    --os-auth-url http://192.168.4.254:35357 \
    --os-identity-api-version 3 --os-project-domain-name default \
    --os-tenant-name admin --os-username admin \
    --os-password sn13sMTl44LoCLrR5X7aXxKzypOBDvtEc0guxpWz \
    --os-user-domain-name default compute service list -f json \
    --service nova-compute
```

## Configurations after deploy

### `openvswitch_vswitchd` containter failed to start due to no bridge `br-eth1`
**On control and compute nodes: ubuntu-server3,4,5,6,7,12,13,14**
```bash
$ docker exec -u 0 openvswitch_vswitchd ovs-vsctl add-br br-eth1
```

### Set IP to `br-ex` of ubuntu-server3 (Re-configure manually after reboot)
```bash
# remove 192.168.1.254 from eth0
$ sudo ip addr del 192.168.1.254/32 dev eth0
$ sudo ip addr add 192.168.1.254/32 dev br-ex
$ sudo ip addr add 192.168.1.3/32 dev br-ex
$ sudo ip link set br-ex up
```

### Set default gateway on Neutron node: ubuntu-server3
```bash
$ sudo route add default gw 192.168.4.1 eth3
```

### Enable nested virtualization on compute nodes

**On ubuntu-server4,5,6,7,12,13,14**
```bash
$ cat /sys/module/kvm_intel/parameters/nested
N
$ sudo rmmod kvm-intel
$ sudo sh -c "echo 'options kvm-intel nested=y' >> /etc/modprobe.d/dist.conf"
$ sudo modprobe kvm-intel
$ cat /sys/module/kvm_intel/parameters/nested
Y
$ modinfo kvm_intel | grep nested
parm:           nested:bool
$ docker restart nova_compute nova_libvirt
```

## Troubleshooting

```bash
# failed to start iscsid container with below error:
# APIError: 500 Server Error: Internal Server Error (\"mkdir /sys/kernel/config: operation not permitted\")
$ sudo systemctl status sys-kernel-config.mount ; \
    sudo modprobe configfs && sudo systemctl start sys-kernel-config.mount \
    && sudo systemctl status sys-kernel-config.mount

# failed to create OpenStack VM
# libvirt error: Could not access KVM kernel module: Permission denied
# qemu-system-x86_64: failed to initialize KVM: Permission denied
# it may be related to the permission of /dev/kvm
stack@ubuntu-server4:~$ sudo setfacl -bn /dev/kvm
(nova-libvirt)[root@ubuntu-server4 /]# getfacl /dev/kvm
getfacl: Removing leading '/' from absolute path names
# file: dev/kvm
# owner: root
# group: qemu
user::rw-
group::rw-
other::---
```


## MISC

### Clean-up before re-deploy

```bash
# pip uninstall all packages. No need when using virtualenv.
$ pip freeze | grep -v virtualenv | sudo xargs pip uninstall -y

# you may encounter issue of docker, run below command to clean.
$ cat /proc/mounts | grep docker
$ sudo apt-get remove -y docker docker-engine docker-ce docker.io \
    && sudo umount /var/lib/docker/aufs && sudo rm -rf /var/lib/docker/
```

## Tips
```bash
# ssh to OpenStack VM using tenant network
(openvswitch-vswitchd)[root@ubuntu-server3 /]# ip netns exec \
    qdhcp-fd3363af-c26e-4219-847b-968b401680e7 ssh -i key-admin.pem ubuntu@172.16.1.3

# select specified host to the new created VM
stack@ubuntu-server8:~/kolla-ansible$ openstack server create --image Ubuntu \
    --flavor flavor-4-6-100 --availability-zone nova:ubuntu-server14 \
    --nic net-id=fd3363af-c26e-4219-847b-968b401680e7 vm-1
```

## All related ubuntu-serverX configurations

These files are put under `/home/liangr/kolla-ansible` of `ubuntu-client1`.
