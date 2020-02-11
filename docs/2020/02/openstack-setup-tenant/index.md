# OpenStack Setup Tenant


## Prerequisites
1. Have the account to the OpenStack env.


## Steps

### Login to the OpenStack GUI.
Open `192.168.1.254` in Chrome or other web browser.

### [Do for the first time] Setup the Networks

1. Navigate `Project -> Network -> Networks`, click `Create Network`.
2. Input the network name and create a `Subnet`. Suggest using private subnets like: `172.16.*.*` to `172.30.*.*`.
3. Input the DNS servers.
![](/forgetful/images/openstack-setup-tenant-network-1.png)
![](/forgetful/images/openstack-setup-tenant-network-2.png)
![](/forgetful/images/openstack-setup-tenant-network-3.png)
4. Navigate `Project -> Network -> Routers`, click `Create Router`. This router makes sure your VMs can access to external networks.
5. Input the router name like `router-net-ext`. Select the `net-ext` as the external network.
![](/forgetful/images/openstack-setup-tenant-router-1.png)
6. Navigate the `Interfaces` tab of `router-net-ext`, click `Add Interface`. Select the network created in #1.
![](/forgetful/images/openstack-setup-tenant-router-2.png)
![](/forgetful/images/openstack-setup-tenant-router-3.png)
7. Navigate `Project -> Network -> Security Groups`, click `Manage Rules` of `default` group. To make it easy, I add several rules to allow most of ports.
![](/forgetful/images/openstack-setup-tenant-security-group-1.png)

### Boot a VM
1. Navigate `Project -> Compute -> Key Pairs`, click `Create Key Pair`. Store the private key to local path like `private.pem`. You will need it to ssh to the created VM without password.
2. Copy the private key `private.pem` to `10.245.48.66`.
3. Navigate `Project -> Compute -> Instances`, click `Launch Instance`. Select `No` to `Create New Volume`.
![](/forgetful/images/openstack-setup-tenant-instance-1.png)

### Access the VM
1. Navigate `Project -> Compute -> Instances`, click `Associate Floating IP` to the created VM. This makes you can access the VM easily from outside.
2. You can see the IP like `172.30.1.50` in the `Floating IPs` of the VM.
3. On `10.245.48.66`, run:
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/private.pem
   ```
4. From `10.245.48.66`, `ssh ubuntu@172.30.1.50`.
