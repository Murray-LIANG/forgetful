---
title: "OpenStack Keystone"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "keystone",
]

comment: true
toc: true
auto_collapse_toc: true
---

- Manage the Users and Roles.
- Manage the Endpoints of services.
- Authentication and authorization.

## Key points

### User

### Credentials

### Authentication

### Token

### Project, Tenant
OpenStack resources belong to the Project/Tenant, that is, the resouces
are project based not user based.
Every user including admin must be a member of the project before accessing
the resources.

### Service

### Endpoint
Each service exposes several endpoints (URL). Other service sends commands
to the endpoint.

```
# list endpoints
$ openstack catalog list
```

### Role
Keystone stores a list of roles.
Other service checks the `policy.json` to see whether the role has right
to perform the action.

# General flow

1. User (i.e admin) uses credential to login. Keystone returns the Token if succeed.
2. Keystone returns the projects the User belongs to (i.e. admin).
3. User gets endpoint of Glance if he wants to list the images.
4. User requests Glance to list the image of project (i.e. admin).
5. Glance checks the User in Keystone (whether it exists and in project admin).
6. Glance checks the role of User has the right to list image, based
    on the setting in policy.json.

# Misc
Two processes run for Keystone:
- key: screen log keystone.log
- key-access: screen log keystone_access.log