# OpenStack Attach Volume


## Sample of REST Request and Response

```bash
stack@rocky:~/cinder$ openstack --debug server add volume vm-ml-1 liangr
```
```python
START with options: [u'--debug', u'server', u'add', u'volume', u'vm-ml-1', u'liangr']
command: server add volume -> openstackclient.compute.v2.server.AddServerVolume (auth=True)
Using auth plugin: password
Using parameters {'username': 'admin', 'project_name': 'admin', 'user_domain_id': 'default', 'auth_url': 'http://172.16.1.12/identity', 'password': '***', 'project_domain_id': 'default'}
Get auth_ref
REQ: curl -g -i -X GET http://172.16.1.12/identity -H "Accept: application/json" -H "User-Agent: osc-lib/1.10.0 keystoneauth1/3.5.0 python-requests/2.18.4 CPython/2.7.12"
Starting new HTTP connection (1): 172.16.1.12
http://172.16.1.12:80 "GET /identity HTTP/1.1" 300 270
RESP: [300] Connection: close Content-Length: 270 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:38 GMT Server: Apache/2.4.18 (Ubuntu) Vary: X-Auth-Token
RESP BODY: {"versions": {"values": [{"status": "stable", "updated": "2018-02-28T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.10", "links": [{"href": "http://172.16.1.12/identity/v3/", "rel": "self"}]}]}}
Making authentication request to http://172.16.1.12/identity/v3/auth/tokens
Resetting dropped connection: 172.16.1.12
http://172.16.1.12:80 "POST /identity/v3/auth/tokens HTTP/1.1" 201 3388
{"token": {"is_domain": false, "methods": ["password"], "roles": [{"id": "78830fd43815419483ae36492b11dc19", "name": "admin"}], "expires_at": "2018-07-10T03:07:38.000000Z", "project": {"domain": {"id": "default", "name": "Default"}, "id": "fed60db241ca4ad5b6a9da1ba04e2cab", "name": "admin"}, "catalog": [{"endpoints": [{"url": "http://172.16.1.12/volume/v3/fed60db241ca4ad5b6a9da1ba04e2cab", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "22691d20faef4719bac109ca124a5211"}], "type": "block-storage", "id": "08bc72d593c5438682fe6be75dffad8d", "name": "cinder"}, {"endpoints": [{"url": "http://172.16.1.12/image", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "99d58834e3c34a299ad6b97ed82bea82"}], "type": "image", "id": "3327bcc2a8b64adc859aad98d19cebc1", "name": "glance"}, {"endpoints": [{"url": "http://172.16.1.12/compute/v2/fed60db241ca4ad5b6a9da1ba04e2cab", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "689819e6f9dd43a1a876ff79312f18aa"}], "type": "compute_legacy", "id": "41aa5f6cc0e74c019ff2832baefc8e23", "name": "nova_legacy"}, {"endpoints": [{"url": "http://172.16.1.12/compute/v2.1", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "bb57b79255044ef395872b0906fdcf14"}], "type": "compute", "id": "539e2a54aec1458cb919c9c59e777b26", "name": "nova"}, {"endpoints": [{"url": "http://172.16.1.12/volume/v1/fed60db241ca4ad5b6a9da1ba04e2cab", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "b8aec49fbeed4f02ad0bf1b86c088e89"}], "type": "volume", "id": "76de4e05035744cdbdb8effa594074f4", "name": "cinder"}, {"endpoints": [{"url": "http://172.16.1.12/identity", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "111d7db3bc5a474f816d7facff428279"}, {"url": "http://172.16.1.12/identity", "interface": "admin", "region": "RegionOne", "region_id": "RegionOne", "id": "425b9c52e5aa402fbcf5df757841e035"}], "type": "identity", "id": "84bcb6f941de4b0da177780ef79f63ed", "name": "keystone"}, {"endpoints": [{"url": "http://172.16.1.12/placement", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "50c1f5700494419da9a173ccc10fd4d7"}], "type": "placement", "id": "999d174020a243a88965f01d4bb9eae1", "name": "placement"}, {"endpoints": [{"url": "http://172.16.1.12:9696/", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "ebc0350068194767934b25e6e4b5201e"}], "type": "network", "id": "b4bd21f8d0e3427a91c933cdd2b7adbb", "name": "neutron"}, {"endpoints": [{"url": "http://172.16.1.12/volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "11c6287c32f3497b81312365c4e7faae"}], "type": "volumev2", "id": "b909e44ece4648879ef903971a939305", "name": "cinderv2"}, {"endpoints": [{"url": "http://172.16.1.12/volume/v3/fed60db241ca4ad5b6a9da1ba04e2cab", "interface": "public", "region": "RegionOne", "region_id": "RegionOne", "id": "30898865f9fe457ca8bb6d2e56e29546"}], "type": "volumev3", "id": "f3f5ec996efb479dafe64f83879112ae", "name": "cinderv3"}], "user": {"domain": {"id": "default", "name": "Default"}, "password_expires_at": null, "name": "admin", "id": "4c82fda464a14c838e191127f7402587"}, "audit_ids": ["LABvdevyS8e1yHfDzVdLAA"], "issued_at": "2018-07-10T02:07:38.000000Z"}}
run(Namespace(device=None, server=u'vm-ml-1', volume=u'liangr'))
REQ: curl -g -i -X GET http://172.16.1.12/compute/v2.1/servers/vm-ml-1 -H "Accept: application/json" -H "OpenStack-API-Version: compute 2.60" -H "User-Agent: python-novaclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e" -H "X-OpenStack-Nova-API-Version: 2.60"
http://172.16.1.12:80 "GET /compute/v2.1/servers/vm-ml-1 HTTP/1.1" 404 82
RESP: [404] Connection: close Content-Length: 82 Content-Type: application/json; charset=UTF-8 Date: Tue, 10 Jul 2018 02:07:41 GMT OpenStack-API-Version: compute 2.60 Server: Apache/2.4.18 (Ubuntu) Vary: OpenStack-API-Version,X-OpenStack-Nova-API-Version X-OpenStack-Nova-API-Version: 2.60 x-compute-request-id: req-becd6183-8f4b-4c11-8a76-b693e896cd2a x-openstack-request-id: req-becd6183-8f4b-4c11-8a76-b693e896cd2a
RESP BODY: {"itemNotFound": {"message": "Instance vm-ml-1 could not be found.", "code": 404}}
GET call to compute for http://172.16.1.12/compute/v2.1/servers/vm-ml-1 used request id req-becd6183-8f4b-4c11-8a76-b693e896cd2a
REQ: curl -g -i -X GET http://172.16.1.12/compute/v2.1/servers?name=vm-ml-1 -H "Accept: application/json" -H "OpenStack-API-Version: compute 2.60" -H "User-Agent: python-novaclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e" -H "X-OpenStack-Nova-API-Version: 2.60"
http://172.16.1.12:80 "GET /compute/v2.1/servers?name=vm-ml-1 HTTP/1.1" 200 300
RESP: [200] Connection: close Content-Length: 300 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:42 GMT OpenStack-API-Version: compute 2.60 Server: Apache/2.4.18 (Ubuntu) Vary: OpenStack-API-Version,X-OpenStack-Nova-API-Version X-OpenStack-Nova-API-Version: 2.60 x-compute-request-id: req-857055b3-f73f-4f7a-8c67-1ceef6811566 x-openstack-request-id: req-857055b3-f73f-4f7a-8c67-1ceef6811566
RESP BODY: {"servers": [{"id": "83486e6e-2600-4149-a578-dbc67b506004", "links": [{"href": "http://172.16.1.12/compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004", "rel": "self"}, {"href": "http://172.16.1.12/compute/servers/83486e6e-2600-4149-a578-dbc67b506004", "rel": "bookmark"}], "name": "vm-ml-1"}]}
GET call to compute for http://172.16.1.12/compute/v2.1/servers?name=vm-ml-1 used request id req-857055b3-f73f-4f7a-8c67-1ceef6811566
REQ: curl -g -i -X GET http://172.16.1.12/compute/v2.1/servers?marker=83486e6e-2600-4149-a578-dbc67b506004&name=vm-ml-1 -H "Accept: application/json" -H "OpenStack-API-Version: compute 2.60" -H "User-Agent: python-novaclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e" -H "X-OpenStack-Nova-API-Version: 2.60"
Resetting dropped connection: 172.16.1.12
http://172.16.1.12:80 "GET /compute/v2.1/servers?marker=83486e6e-2600-4149-a578-dbc67b506004&name=vm-ml-1 HTTP/1.1" 200 15
RESP: [200] Connection: close Content-Length: 15 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:42 GMT OpenStack-API-Version: compute 2.60 Server: Apache/2.4.18 (Ubuntu) Vary: OpenStack-API-Version,X-OpenStack-Nova-API-Version X-OpenStack-Nova-API-Version: 2.60 x-compute-request-id: req-975f02d6-b82c-42ca-b3c8-6659c5b759f5 x-openstack-request-id: req-975f02d6-b82c-42ca-b3c8-6659c5b759f5
RESP BODY: {"servers": []}
GET call to compute for http://172.16.1.12/compute/v2.1/servers?marker=83486e6e-2600-4149-a578-dbc67b506004&name=vm-ml-1 used request id req-975f02d6-b82c-42ca-b3c8-6659c5b759f5
REQ: curl -g -i -X GET http://172.16.1.12/compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004 -H "Accept: application/json" -H "OpenStack-API-Version: compute 2.60" -H "User-Agent: python-novaclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e" -H "X-OpenStack-Nova-API-Version: 2.60"
Resetting dropped connection: 172.16.1.12
http://172.16.1.12:80 "GET /compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004 HTTP/1.1" 200 2028
RESP: [200] Connection: close Content-Length: 2028 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:42 GMT OpenStack-API-Version: compute 2.60 Server: Apache/2.4.18 (Ubuntu) Vary: OpenStack-API-Version,X-OpenStack-Nova-API-Version X-OpenStack-Nova-API-Version: 2.60 x-compute-request-id: req-4a383ea6-629b-4b4b-b691-4894d8119f7d x-openstack-request-id: req-4a383ea6-629b-4b4b-b691-4894d8119f7d
RESP BODY: {"server": {"OS-EXT-STS:task_state": null, "addresses": {"public": [{"OS-EXT-IPS-MAC:mac_addr": "fa:16:3e:8d:f6:78", "version": 6, "addr": "2001:db8::c", "OS-EXT-IPS:type": "fixed"}, {"OS-EXT-IPS-MAC:mac_addr": "fa:16:3e:8d:f6:78", "version": 4, "addr": "172.24.4.4", "OS-EXT-IPS:type": "fixed"}]}, "links": [{"href": "http://172.16.1.12/compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004", "rel": "self"}, {"href": "http://172.16.1.12/compute/servers/83486e6e-2600-4149-a578-dbc67b506004", "rel": "bookmark"}], "image": {"id": "2ef5bb36-4bcc-44ec-bad6-2ce95229fbbf", "links": [{"href": "http://172.16.1.12/compute/images/2ef5bb36-4bcc-44ec-bad6-2ce95229fbbf", "rel": "bookmark"}]}, "OS-EXT-SRV-ATTR:user_data": null, "OS-EXT-STS:vm_state": "active", "OS-EXT-SRV-ATTR:instance_name": "instance-00000001", "OS-EXT-SRV-ATTR:root_device_name": "/dev/vda", "OS-SRV-USG:launched_at": "2018-05-02T05:10:33.000000", "flavor": {"ephemeral": 0, "ram": 512, "original_name": "m1.tiny", "vcpus": 1, "extra_specs": {}, "swap": 0, "disk": 1}, "id": "83486e6e-2600-4149-a578-dbc67b506004", "security_groups": [{"name": "default"}], "OS-SRV-USG:terminated_at": null, "os-extended-volumes:volumes_attached": [], "user_id": "4c82fda464a14c838e191127f7402587", "OS-EXT-SRV-ATTR:hostname": "vm-ml-1", "OS-DCF:diskConfig": "MANUAL", "accessIPv4": "", "accessIPv6": "", "OS-EXT-SRV-ATTR:reservation_id": "r-7onfm7f2", "progress": 0, "OS-EXT-STS:power_state": 1, "OS-EXT-AZ:availability_zone": "nova", "config_drive": "", "status": "ACTIVE", "OS-EXT-SRV-ATTR:ramdisk_id": "", "updated": "2018-05-02T05:10:34Z", "hostId": "6d6dc27e5ae9532a23231ad5142ee1e86ed351c8354b47c3a40b23ba", "OS-EXT-SRV-ATTR:host": "rocky", "description": "vm-ml-1", "tags": [], "key_name": null, "OS-EXT-SRV-ATTR:kernel_id": "", "OS-EXT-SRV-ATTR:hypervisor_hostname": "rocky", "locked": false, "name": "vm-ml-1", "OS-EXT-SRV-ATTR:launch_index": 0, "created": "2018-05-02T05:10:22Z", "tenant_id": "fed60db241ca4ad5b6a9da1ba04e2cab", "host_status": "UP", "metadata": {}}}
GET call to compute for http://172.16.1.12/compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004 used request id req-4a383ea6-629b-4b4b-b691-4894d8119f7d
REQ: curl -g -i -X GET http://172.16.1.12/volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/liangr -H "Accept: application/json" -H "User-Agent: python-cinderclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e"
Resetting dropped connection: 172.16.1.12
http://172.16.1.12:80 "GET /volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/liangr HTTP/1.1" 404 79
RESP: [404] Connection: close Content-Length: 79 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:43 GMT Server: Apache/2.4.18 (Ubuntu) x-compute-request-id: req-43f5112e-2707-4348-8868-bc294d475cc4 x-openstack-request-id: req-43f5112e-2707-4348-8868-bc294d475cc4
RESP BODY: {"itemNotFound": {"message": "Volume liangr could not be found.", "code": 404}}
GET call to volumev2 for http://172.16.1.12/volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/liangr used request id req-43f5112e-2707-4348-8868-bc294d475cc4
REQ: curl -g -i -X GET http://172.16.1.12/volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/detail?all_tenants=1&name=liangr -H "Accept: application/json" -H "User-Agent: python-cinderclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e"
Resetting dropped connection: 172.16.1.12
http://172.16.1.12:80 "GET /volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/detail?all_tenants=1&name=liangr HTTP/1.1" 200 1039
RESP: [200] Connection: close Content-Length: 1039 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:43 GMT Server: Apache/2.4.18 (Ubuntu) x-compute-request-id: req-7b77e263-520e-4e12-8eb9-8ceee3c7e680 x-openstack-request-id: req-7b77e263-520e-4e12-8eb9-8ceee3c7e680
RESP BODY: {"volumes": [{"migration_status": null, "attachments": [], "links": [{"href": "http://172.16.1.12/volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/7dce5d8b-7974-432a-93e5-315532d1dad4", "rel": "self"}, {"href": "http://172.16.1.12/volume/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/7dce5d8b-7974-432a-93e5-315532d1dad4", "rel": "bookmark"}], "availability_zone": "nova", "os-vol-host-attr:host": "rocky@unity_backend_1#puppet_pool", "encrypted": false, "updated_at": "2018-07-09T07:36:42.000000", "replication_status": null, "snapshot_id": null, "id": "7dce5d8b-7974-432a-93e5-315532d1dad4", "size": 2, "user_id": "4c82fda464a14c838e191127f7402587", "os-vol-tenant-attr:tenant_id": "fed60db241ca4ad5b6a9da1ba04e2cab", "os-vol-mig-status-attr:migstat" : null, "metadata": {}, "status": "available", "description": null, "multiattach": false, "source_volid": null, "consistencygroup_id": null, "os-vol-mig-status-attr:name_id": null, "name": "liangr", "bootable": "false", "created_at": "2018-07-04T06:26:33.000000", "volume_type": "liangr"}]}
GET call to volumev2 for http://172.16.1.12/volume/v2/fed60db241ca4ad5b6a9da1ba04e2cab/volumes/detail?all_tenants=1&name=liangr used request id req-7b77e263-520e-4e12-8eb9-8ceee3c7e680
REQ: curl -g -i -X POST http://172.16.1.12/compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004/os-volume_attachments -H "Accept: application/json" -H "Content-Type: application/json" -H "OpenStack-API-Version: compute 2.60" -H "User-Agent: python-novaclient" -H "X-Auth-Token: {SHA1}d3951552a72aff9cefcf94d78553e5039b70a39e" -H "X-OpenStack-Nova-API-Version: 2.60" -d '{"volumeAttachment": {"volumeId": "7dce5d8b-7974-432a-93e5-315532d1dad4"}}'
Resetting dropped connection: 172.16.1.12
http://172.16.1.12:80 "POST /compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004/os-volume_attachments HTTP/1.1" 200 194
RESP: [200] Connection: close Content-Length: 194 Content-Type: application/json Date: Tue, 10 Jul 2018 02:07:43 GMT OpenStack-API-Version: compute 2.60 Server: Apache/2.4.18 (Ubuntu) Vary: OpenStack-API-Version,X-OpenStack-Nova-API-Version X-OpenStack-Nova-API-Version: 2.60 x-compute-request-id: req-31c17aa4-13bb-46ba-b8da-73a075a4ef38 x-openstack-request-id: req-31c17aa4-13bb-46ba-b8da-73a075a4ef38
RESP BODY: {"volumeAttachment": {"device": "/dev/vdb", "serverId": "83486e6e-2600-4149-a578-dbc67b506004", "id": "7dce5d8b-7974-432a-93e5-315532d1dad4", "volumeId": "7dce5d8b-7974-432a-93e5-315532d1dad4"}}
POST call to compute for http://172.16.1.12/compute/v2.1/servers/83486e6e-2600-4149-a578-dbc67b506004/os-volume_attachments used request id req-31c17aa4-13bb-46ba-b8da-73a075a4ef38
clean_up AddServerVolume:
END return value: 0
```

## Invocation Flow from Nova to Cinder

There is a route list in `nova/api/openstack/compute/routes.py`

```python
('/servers/{server_id}/os-volume_attachments', {
    'GET': [server_volume_attachments_controller, 'index'],
    'POST': [server_volume_attachments_controller, 'create'],
}),
('/servers/{server_id}/os-volume_attachments/{id}', {
    'GET': [server_volume_attachments_controller, 'show'],
    'PUT': [server_volume_attachments_controller, 'update'],
    'DELETE': [server_volume_attachments_controller, 'delete']
}),
```

The `server_volume_attachments_controller` is `VolumeAttachmentController` of `nova/api/openstack/compute/volumes.py`


## Attach
The `create` of `VolumeAttachmentController` actually invokes `attach_volume` of `nova/compute/api.py`

The `attach_volume` first does some pre-check and then attaches volume based on which style of attaching flow it uses:
- Old style (cinder API before 3.44):
    1. raise exception if the volume is already attached.
    2. reserve volume of `volume_api` (`nova/volume/cinder.py`)
        
        `cinderclient.volumes.reverse` >>> python-cinderclient REST POST `os-reserve` >>> `_reserve` of `cinder/api/contrib/volume_actions.py` >>> `reserve_volume` of `cinder/volume/api.py`

        volume status should be under `in-use` (for multiattach) or `available` (for otherwise) before reserving.
        
        Then the status will be changed to **`attaching`** if reserving successfully.

    3. `attach_volume` of `nova/compute/rpcapi.py` >>> **`cast`** rpc **async** >>> `attach_volume` of `nova/compute/manager.py` >>> `_volume_attach` (for multiattach) or `_legacy_volume_attach` (for otherwise) of `nova/virt/block_device.py`

        - `_legacy_volume_attach`

        1. `_legacy_volume_attach` >>> `initialize_connection` of `volume_api` (`nova/volume/cinder.py`) >>> `cinderclient.volumes.initialize_connection` >>> python-cinderclient REST POST `os-initialize_connection` >>> `_initialize_connection` of `cinder/api/contrib/volume_actions.py` >>> `initialize_connection` of `cinder/volume/api.py` >>> `initialize_connection` of `cinder/volume/rpcapi.py` >>> `initialize_connection` of `cinder/volume/manager.py` >>> `initialize_connection` of driver.
        2. if volume not attached to instance at boot time (ie a data volume), `_legacy_volume_attach` >>> `attach_volume` of i.e. `nova/virt/libvirt/driver.py` >>> `connect_volume` of i.e.  `nova/virt/libvirt/volume/iscsi.py`
        3. `_legacy_volume_attach` >>> `attach` of `volume_api` (`nova/volume/cinder.py`) >>> `cinderclient.volumes.reverse` >>> python-cinderclient REST POST `os-attach` >>> `_attach` of `cinder/api/contrib/volume_actions.py` >>> `attach` of `cinder/volume/api.py` >>> `attach_volume` of `cinder/volume/rpcapi.py` >>> `attach_volume` of `cinder/volume/manager.py` >>> `attach_volume` of driver (not implemented by Unity/VNX driver)

        - `_volume_attach`
        1. `_volume_attach` >>> `attachment_update` of `nova/volume/cinder.py` >>> `cinderclient.attachments.update` >>> python-cinderclient REST POST `attachments update` >>> `AttachmentsController.update` of `cinder/api/v3/attachments.py` >>> `attachment_update` of `volume_api` (`cinder/volume/api.py`) >>> `attachment_update` of `cinder/volume/rpcapi.py` >>> `attachment_update` of `cinder/volume/manager.py`
            1. \>>> `_connection_create` >>> driver `create_export`
            2. \>>> `_connection_create` >>> driver `initialize_connection`
            3. \>>> driver `attach_volume`
        2. if volume not attached to instance at boot time (ie a data volume), `_volume_attach` >>> `attach_volume` of i.e. `nova/virt/libvirt/driver.py` >>> `connect_volume` of i.e.  `nova/virt/libvirt/volume/iscsi.py`
        3. `_volume_attach` >>> `attachment_complete` of `nova/volume/cinder.py` >>> `cinderclient.attachments.complete` >>> python-cinderclient REST POST `attachments complete` >>> `AttachmentsController.complete` of `cinder/api/v3/attachments.py` >>> mark the volume status to `in-use`

- New style (cinder API from 3.44):
    1. `attachment_create` of `volume_api` (`nova/volume/cinder.py`) >>>  `cinderclient.attachments.create` >>> python-cinderclient REST POST `attachments create` >>> `AttachmentsController.create` of `cinder/api/v3/attachments.py` >>> `attachment_create` of `volume_api` (`cinder/volume/api.py`)
    2. `attachment_create` of `volume_api` (`cinder/volume/api.py`) >>> `_attachment_reserve` >>> the volume status should be under `available`, `in-use` (for multiattach), or `downloading` before reserving. It will be changed to `reserved` if successfully reserving.
    3. if not boot-from-volume case, `attachment_create` of `volume_api` (`cinder/volume/api.py`) >>> `attachment_update` of `cinder/volume/rpcapi.py` >>> `attachment_update` of `cinder/volume/manager.py`
        1. \>>> `_connection_create` >>> driver `create_export`
        2. \>>> `_connection_create` >>> driver `initialize_connection`
        3. \>>> driver `attach_volume`

    4. `attach_volume` of `nova/compute/rpcapi.py` (same of step iii of old style)


## Detach

The `delete` of `VolumeAttachmentController` actually invokes `detach_volume` of `nova/compute/api.py`

The `detach_volume` has different flow for `SHELVED_OFFLOADED` status instance. >>>
`_detach_volume_shelved_offloaded` (for `SHELVED_OFFLOADED` instances), `_detach_volume` (for other instances)

### `SHELVED_OFFLOADED` Instances (not support multiattach yet
In general, it calls `volume_api` `detach`, `volume_api` `terminate_connection` and deletes the bdm record. If the volume has `delete_on_termination` option set, call `volume_api` `delete` as well.

1. `_detach_volume_shelved_offloaded` of `nova/compute/api.py` >>> `begin_detaching` of `nova/volume/cinder.py` >>> `cinderclient.volumes.begin_detaching` >>> python-cinderclient REST POST `os-begin_detaching` >>> `_begin_detaching` of `cinder/api/contrib/volume_actions.py` >>> `begin_detach` of `cinder/volume/api.py`

    volume status should be under `in-use`.
        
    Then the status will be changed to **`detaching`** if succeeds.

2. `_detach_volume_shelved_offloaded` of `nova/compute/api.py` >>> `_local_cleanup_bdm_volumes`
    
    1. <a name="new_detach"></a>if `bdm` is volume and has `attachment_id` >>> `attachment_delete` of `nova/volume/cinder.py` >>> `cinderclient.attachments.delete` >>> python-cinderclient REST POST `attachments delete` >>> `AttachmentsController.delete` of `cinder/api/v3/attachments.py` >>> `attachment_delete` of `volume_api` (`cinder/volume/api.py`) >>> `attachment_delete` of `cinder/volume/rpcapi.py` >>> **`call`** rpc **`sync`** >>> `attachment_delete` of `cinder/volume/manager.py` >>> `_do_attachment_delete`
        1. `_do_attachment_delete` >>> `_connection_terminate` >>> driver `terminate_connection` >>> return whether the volume is shared by other instances
            ```python
            # snippets of cinder/volume/manager.py:_connection_terminate
            shared_connections = self.driver.terminate_connection(volume,
                                                                  connector,
                                                                  force=force)
            if not isinstance(shared_connections, bool):
                shared_connections = False
            ```
        2. `_do_attachment_delete` >>> driver `detach_volume` (not implemented in Unity/VNX driver)
        3. if the volume is not shared by other instances (return value of step `a`), `_do_attachment_delete` >>> driver `remove_export` (not implemented in Unity/VNX driver)

    2. <a name="old_detach"></a>otherwise
        1. \>>> `terminate_connection` of `nova/volume/cinder.py` >>> `cinderclient.volumes.terminate_connection` >>> python-cinderclient REST POST `volumes os-terminate_connection` >>> `VolumeActionsController._terminate_connection` of `cinder/api/contrib/volume_actions.py` >>> `terminate_connection` of `volume_api` (`cinder/volume/api.py`) >>> `terminate_connection` of `cinder/volume/rpcapi.py` >>> **`call`** rpc **`sync`** >>> `terminate_connection` of `cinder/volume/manager.py` >>> driver `terminate_connection`
        2. \>>> `detach` of `nova/volume/cinder.py` >>> `cinderclient.volumes.detach` >>> python-cinderclient REST POST `volumes os-detach` >>> `VolumeActionsController._detach` of `cinder/api/contrib/volume_actions.py` >>> `detach` of `volume_api` (`cinder/volume/api.py`) >>> `detach_volume` of `cinder/volume/rpcapi.py` >>> **`call`** rpc **`sync`** >>> `detach_volume` of `cinder/volume/manager.py` 
            1. manager `detach_volume` >>> driver `detach_volume`
            2. manager `detach_volume` >>> driver `remove_export`
            3. manager `detach_volume` >>> driver `finish_detach` of `cinder/objects/volumes.py` >>> update the status to `detached`

3. `_detach_volume_shelved_offloaded` of `nova/compute/api.py` >>> `_local_cleanup_bdm_volumes` >>> `delete` of `nova/volume/cinder.py`

### Other Instances

1. `_detach_volume` of `nova/compute/api.py` >>> `begin_detaching` of `nova/volume/cinder.py` >>> `cinderclient.volumes.begin_detaching` >>> python-cinderclient REST POST `os-begin_detaching` >>> `_begin_detaching` of `cinder/api/contrib/volume_actions.py` >>> `begin_detach` of `cinder/volume/api.py`

    volume status should be under `in-use`.
        
    Then the status will be changed to **`detaching`** if succeeds.

2. `_detach_volume` of `nova/compute/api.py` >>> `detach_volume` of `nova/compute/rpcapi.py` >>> **`cast`** rpc **async** >>> `detach_volume` of `nova/compute/manager.py` >>> `_detach_volume` of `nova/compute/manager.py` >>> 
`detach` of `nova/virt/block_device.py` >>> `_do_detach`

    1. `_do_detach` >>> `driver_detach` of `nova/virt/block_device.py` >>> `detach_volume` of `nova/virt/libvirt/driver.py` >>> calling os-brick to detach iscsi volume
    2. Different styles of detaching
        - Old style (if bdm doesn't have `attachment_id`)
        1. \>>> `terminate_connection` of `nova/volume/cinder.py`, same as [2.ii.a](#old_detach) of `SHELVED_OFFLOADED` Instances
        2. \>>> `detach` of `nova/volume/cinder.py`, same as [2.ii.b](#old_detach) of `SHELVED_OFFLOADED` Instances

        - New style (otherwise)
        1. \>>> `attachment_delete` of `nova/volume/cinder.py`, same as [2.i](#new_detach) of `SHELVED_OFFLOADED` Instances

 
