---
title: "Package Storops RPM on RHEL"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "storops",
    "package",
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
```console
$ sudo  stack
$ sudo passwd stack
$ sudo visudo
```

## Register subscription

```console
$ sudo subscription-manager register
$ sudo subscription-manager attach --auto
```

## Install the basic packages like `rpm-build`

```console
$ sudo yum install -y rpm-build git wget python-pip
```

**NOTE: need to upgrade the `setuptools` using pip. Otherwise, you'll meet the error like `error in storops setup command: 'install_requires' must be a string or list of strings containing valid project/version requirement specifiers`**
```console
$ sudo pip install setuptools -U
```

## Install the dependent Python packages of storops 

```console

# # Configure the yum repo
$ sudo su -
$ cat > /etc/yum.repos.d/common-candidate.repo << EOF
[common-candidate]
name=common-candidate
baseurl=http://cbs.centos.org/repos/cloud7-openstack-common-candidate/x86_64/os
enabled=1
gpgcheck=0
priority=2
EOF

# # Install the packages
sudo yum install -y python2-devel python-bitmath python-cachez \ 
    python-dateutil python-ddt python-enum34 python-fasteners \
    python-hamcrest python-mock python-persist-queue python-pytest \ 
    python-pytest-xdist python-requests python-retryz python-xmltodict
```

## Build the storops rpms

Refer to https://github.com/emc-openstack/storops/tree/develop/utility/rpm_publish