---
title: "Install Storops RPM on RHEL7"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "storops",
    "rpm",
    "redhat",
    "rhel",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Install a RHEL VM.

1. If using `qcow2` image, the default user is `cloud-user`.
Use the OpenStack ssh key to login.

2. Create a new user `stack` with password `welcome`.
```bash
$ sudo useradd stack
$ sudo passwd stack
$ sudo visudo
```

3. Modify `/etc/ssh/sshd_config` to allow login via password

Entry name is `PasswordAuthentication`.

## Register subscription

```bash
$ sudo subscription-manager register
$ sudo subscription-manager attach --auto
$ sudo subscription-manager repos \
    --enable=rhel-7-server-rpms --enable=rhel-7-server-extras-rpms \
    --enable=rhel-7-server-rh-common-rpms \
    --enable=rhel-ha-for-rhel-7-server-rpms
$ sudo yum update
```

## Download `storops` rpm from github
```bash
curl -OJL https://github.com/emc-openstack/storops/releases/download/r1.0.0/python2-storops-1.0.0-1.el7.noarch.rpm
```

## Check its dependencies
```bash
rpm -qp python2-storops-1.0.0-1.el7.noarch.rpm --requires
```

## Install the dependencies
```bash
# Configure the yum repo
$ sudo su -
$ cat > /etc/yum.repos.d/common-candidate.repo << EOF
[common-candidate]
name=common-candidate
baseurl=http://cbs.centos.org/repos/cloud7-openstack-common-candidate/x86_64/os
enabled=1
gpgcheck=0
priority=2
EOF

# Install the packages
sudo yum install -y python-bitmath python-cachez \ 
    python-dateutil python-enum34 python-persist-queue \ 
    python-requests python-retryz
```