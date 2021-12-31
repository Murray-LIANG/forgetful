
```console
$ sudo useradd -m -s /bin/bash -G sudo stack
$ sudo passwd stack
$ # use welcome as the password.
$ sudo vi /etc/ssh/sshd_config
$ # modify `PasswordAuthentication no` to `PasswordAuthentication yes`.
$ sudo systemctl daemon-reload && sudo systemctl restart sshd

$ sudo vi /etc/sudoers
$ # add `stack   ALL=(ALL) NOPASSWD:ALL`

$ ubuntu@manila-group-replica:~$ sudo bash -c 'cat > /usr/local/share/ca-certificates/emc-1.crt <<EOF
-----BEGIN CERTIFICATE-----
MIIDajCCAlKgAwIBAgIQDnpJf/sai2ikg8QrEDRcejANBgkqhkiG9w0BAQUFADA9
MQswCQYDVQQGEwJVUzEYMBYGA1UEChMPRU1DIENvcnBvcmF0aW9uMRQwEgYDVQQD
EwtFTUMgUm9vdCBDQTAeFw0xMTAzMDgwMjM1MThaFw0yNjAzMDgwMjM1MThaMD0x
CzAJBgNVBAYTAlVTMRgwFgYDVQQKEw9FTUMgQ29ycG9yYXRpb24xFDASBgNVBAMT
C0VNQyBSb290IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwEV0
QaykbhIOVKj1BunB8pXsISlXgiv10QSGSxG2Dnbwoli0WSgPpLqPD8bsQuwjReg0
ERGXTXpxDEpb4Kya+YcIr4KGMd+EIdLjogXnrKv1/EWa54UNNjNLU6tkwEnVQ79p
Sbx2weCxEi+VG755+Bbb5AJKDcgk4ss5hXjI8tOzAgHe+tReNQamMSOgCO+4bZJ1
RBalcYHmGxVz2TbK0qrKKC7Um4ALQfRQejB+TuvYMoTZHD8Wm/e3Hdq7wwTOmQUL
/hG4+J+k4fl8WUtf4M6CzmeYVnEpZ34wk4H/1bRmFI9jvEQlmu/uKmFZ8DPOvK8j
YJCPft/fWOLkCZOSPwIDAQABo2YwZDAOBgNVHQ8BAf8EBAMCAYYwEgYDVR0TAQH/
BAgwBgEB/wIBAzAdBgNVHQ4EFgQUjyKad6YrWTr8z+fAlE5VRpSg/zQwHwYDVR0j
BBgwFoAUjyKad6YrWTr8z+fAlE5VRpSg/zQwDQYJKoZIhvcNAQEFBQADggEBALaL
B5rAo9GLri9vvYMIkMwtI4SFYeftNrY47YA4o49sbCVlgdmzUXWk48aevoUZRl6/
rEPFbTxaZUbmjOv+XO+bGFA3T57RS6rAFeGBai/UirrckJhGgusAVU5lFtO31Mgm
W3cPXqV+PXwwHKbgLRCeTJFK3Rw68TxBqazMjNp4WufdnPC379Fg/zeKrCLwgsa4
AVFHmeIadvijSQBpY0bFzsSZGF/PmAh+NiYJpWRdDXfeeQStdZWxPESbWoXPu/Qg
0dIifLaHr2Nugkg8eTcp+F2rl2YIjnQcEFqOUNhyI8kPzzsWinYel47tC9kDL7qR
s34MLubs2L1iMIk7fJ4=
-----END CERTIFICATE-----
EOF'

$ ubuntu@manila-group-replica:~$ sudo bash -c 'cat > /usr/local/share/ca-certificates/emc-2.crt <<EOF
-----BEGIN CERTIFICATE-----
MIIFaDCCBFCgAwIBAgIQI3k+tfxRq38cNrKi5cYX0TANBgkqhkiG9w0BAQsFADB4
MQswCQYDVQQGEwJVUzEYMBYGA1UEChMPRU1DIENvcnBvcmF0aW9uMSUwIwYDVQQL
ExxHbG9iYWwgU2VjdXJpdHkgT3JnYW5pemF0aW9uMSgwJgYDVQQDEx9FTUMgU1NM
IERlY3J5cHRpb24gQXV0aG9yaXR5IHYyMB4XDTE2MTEzMDIxNDU0NloXDTIwMDIy
ODIxNDU0N1owgZgxITAfBgkqhkiG9w0BCQEWEkdTT19XZWJTZWNAZW1jLmNvbTEL
MAkGA1UEBhMCVVMxGDAWBgNVBAoTD0VNQyBDb3Jwb3JhdGlvbjElMCMGA1UECxMc
R2xvYmFsIFNlY3VyaXR5IE9yZ2FuaXphdGlvbjElMCMGA1UEAxMcRU1DIFNTTCBE
ZWNyeXB0aW9uIEF1dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBAOvkLxl1yJ5lgU1JOMB7I4Ckar4N4IR5dHR0psFfDJR39jpnS+aHnsrbwfte
isOnmdH5EONl+DdhbsO4CyWvoPzq0nOtd0DszyJRhoWhCX2NpfolG7NSMa2Me62h
tFzUKhx8YabO+jZwNGlgJYib4vVGymP+4HS+/2sZpcm5w9qSK7xr/m4v41bd/dit
aOTx8VEe4FbGWubcoPy4dwZSP4NxwBG5BXdSOVGzuPzFO9d1DT/WJt8rF8M7b9G5
aaBvGwhDYImO3Tl8PcEvZ3mbeXNcJvhw3YcaE3iNxPrPqbJMJD5WquV8u2jvQyXW
QN6S1M+EQurPHHcRKn/vU17VHl8CAwEAAaOCAcswggHHMB8GA1UdIwQYMBaAFKXI
N8PH+Ydkms8NqIKyTcTObvVDMBIGA1UdEwEB/wQIMAYBAf8CAQAwggFfBgNVHR8E
ggFWMIIBUjBBoD+gPYY7aHR0cDovL3BraS5jb3JwLmVtYy5jb20vY3JsL0VNQ1NT
TERlY3J5cHRpb25BdXRob3JpdHl2Mi5jcmwwgcSggcGggb6GgbtsZGFwOi8vL0NO
PUVNQyBTU0wgRGVjcnlwdGlvbiBBdXRob3JpdHkgdjIsQ049RU1DIFNTTCBEZWNy
eXB0aW9uIEF1dGhvcml0eSB2MixDTj1DRFAsQ049UHVibGljIEtleSBTZXJ2aWNl
cyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPWVtY3Jvb3QsREM9ZW1j
LERDPWNvbT9jZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0MEagRKBChkBodHRwOi8v
ZW50ZXJwcmlzZWNhLmNvcnAuZW1jLmNvbS9FTUNTU0xEZWNyeXB0aW9uQXV0aG9y
aXR5djIuY3JsMA4GA1UdDwEB/wQEAwIBhjAdBgNVHQ4EFgQUs5fLD4TFDvRTZux+
zJaxmlJhViUwDQYJKoZIhvcNAQELBQADggEBABTh4MbeaOk+zc8z3C2jymJDhpd2
iwTwmo+wGBfQ76f92flnpghL//2pvo++GapiGwJ+K2muqcF065f9x4de4Nj8d2Le
2mYgcQsMQkW05i9f2+73vZ1MNSpWog81rBk1e7DDP50xN/p8mME8ZLzFCtj08W8Z
DxE6NN+eoZD4sCeticJKGw88UoGRtpTAbwV5oRPk4vTV/Zb7yudb5jT9FYP/hGz7
KaqD7hIL0nGrL/9J7wwOkB7YL3w5rGLvXx+fDx8rK0rFZRb5TuimF/aDuGXjYvci
gaYV/LEaqQdcgZACYZCSYhEpR4yKbTx+tWrJnvu7UK4s8NkdAcf5+93rZc4=
-----END CERTIFICATE-----
EOF'

$ ubuntu@manila-group-replica:~$ sudo bash -c 'cat > /usr/local/share/ca-certificates/emc-3.crt <<EOF
-----BEGIN CERTIFICATE-----
MIIEzDCCA7SgAwIBAgIQfzDeHUGOuZX68dftXnbR9zANBgkqhkiG9w0BAQsFADA9
MQswCQYDVQQGEwJVUzEYMBYGA1UEChMPRU1DIENvcnBvcmF0aW9uMRQwEgYDVQQD
EwtFTUMgUm9vdCBDQTAeFw0xNTEwMTUxNzAwMzJaFw0yNjAzMDYwMjM5MzNaMHgx
CzAJBgNVBAYTAlVTMRgwFgYDVQQKEw9FTUMgQ29ycG9yYXRpb24xJTAjBgNVBAsT
HEdsb2JhbCBTZWN1cml0eSBPcmdhbml6YXRpb24xKDAmBgNVBAMTH0VNQyBTU0wg
RGVjcnlwdGlvbiBBdXRob3JpdHkgdjIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAw
ggEKAoIBAQDJvnnKsIWCtCMlm1P53fHpAnCmPfTgtGICGj1dgM2osz5Y0TZRqf57
OefJaE9uZCFr2DMPR90s1mI3ybdpgA/EA6bGDdepW3jLCn7y8uVFZ94xLr3Hv5Xe
fzIUnUXakmIbGmKfBfhVLHQfY22RDks5RNCj/Y+Q0xGbJvjKzes+FnGkMy5WdJ5P
kz8Awlbz26HVX1lh4+7KEcfjV1lyNtMhlSk7KJmVChlRvoF4u40AI7AwHamm7D4R
3BhiMzHpj1NO5tb4exd1Y6Y38pDFaIJGCDe4irdaiYg3dUSYv7oPazFCv7ng4aNR
hwPyTjAhWSXWC4kkZqgECEmeRdgX5CXxAgMBAAGjggGLMIIBhzCCAR8GA1UdHwSC
ARYwggESMDWgM6Axhi9odHRwOi8vcGtpLmNvcnAuZW1jLmNvbS9jcmwvRU1DJTIw
Um9vdCUyMENBLmNybDA6oDigNoY0aHR0cDovL2VudGVycHJpc2VjYS5jb3JwLmVt
Yy5jb20vRU1DJTIwUm9vdCUyMENBLmNybDCBnKCBmaCBloaBk2xkYXA6Ly8vQ049
RU1DIFJvb3QgQ0EsQ049RU1DIFJvb3QgQ0EsQ049Q0RQLENOPVB1YmxpYyBLZXkg
U2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1lbWNyb290
LERDPWVtYyxEQz1jb20/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdDAOBgNVHQ8B
Af8EBAMCAYYwHwYDVR0jBBgwFoAUjyKad6YrWTr8z+fAlE5VRpSg/zQwEgYDVR0T
AQH/BAgwBgEB/wIBAjAdBgNVHQ4EFgQUpcg3w8f5h2Sazw2ogrJNxM5u9UMwDQYJ
KoZIhvcNAQELBQADggEBAKDr+Kz19+Dw7bN+qm4+TZuR0g30pEiGov7i30D6hNL7
XSPzGRmQYXEmucEEsoMY6iBMAPmLqdWFfBDh2vSmnOGk0IL+q3WzLq6IGPpXI4Wf
GGAnjyujnPsk6YP1OyrqYlVN0BUPQ3Jz8l3OI1Ga0/2RM5jogkCqszSoaHdNrouk
mA5Rz0cgETyU5TXC/+a6CEqDbFviqaiiHHZvjCVlgKmdQXirEPm6b2vp2B/DBiVW
6eTyDrIGle10RuPZTKSmlEWSgTshyMCNdOFLm7TsSkbgLgWWwFyDOeCHhvZdEuqP
dApqEUwXnO0ppEtljo5zvCLsYmkri4bxanMGHBcgG9o=
-----END CERTIFICATE-----
EOF'
$ sudo update-ca-certificates
$ # test with `wget https://tarballs.opendev.org/openstack/manila-image-elements/images/manila-service-image-master.qcow2`

$ su - stack
$ git clone https://github.com/openstack/devstack.git

$ # change the IP address accordingly.
$ cd devstack && vi local.conf
$ # put below content to local.conf
[[local|localrc]]
HOST_IP=172.16.1.7
SERVICE_HOST=172.16.1.7

ADMIN_PASSWORD=welcome
MYSQL_PASSWORD=welcome
RABBIT_PASSWORD=welcome
SERVICE_PASSWORD=welcome
SERVICE_TOKEN=welcome
SWIFT_HASH=welcome

disable_service h-eng
disable_service h-api
disable_service h-api-cfn
disable_service h-api-cw
disable_service tempest
disable_service dstat

# For backup and swift.
enable_service s-proxy s-object s-container s-account
enable_service c-bak

# Enable Manila
# enable_plugin manila https://github.com/openstack/manila stable/queens
enable_plugin manila https://github.com/openstack/manila

LOGFILE=$DEST/logs/stack.sh.log
SCREEN_LOGDIR=$DEST/logs/screen
LOG_COLOR=False

RECLONE=True
OFFLINE=False

PHYSICAL_NETWORK=public_eth1
# Devstack will add PUBLIC_INTERFACE to OVS_PHYSICAL_BRIDGE.
PUBLIC_PHYSICAL_NETWORK=public_eth0
PUBLIC_INTERFACE=eth0
OVS_PHYSICAL_BRIDGE=br-ex

CINDER_ENABLED_BACKENDS=vmdk:none


[[post-config|$NOVA_CONF]]
[DEFAULT]
rpc_response_timeout=300
service_down_time=300
live_migration_uri=qemu+tcp://%s/system
force_config_drive = False

[libvirt]
iscsi_use_multipath = True

[database]
max_pool_size=40
max_overflow=60


[[post-config|$CINDER_CONF]]
[DEFAULT]
rpc_response_timeout=300
service_down_time=300
volume_name_template = volume-%s
#enabled_backends = vnx_backend_rep, vnx_backend_1, vnx_backend_2,storagepool_0, storagepool_fc, storagepool_scsi, storagepool_fc_2
enabled_backends = unity_backend_1
default_volume_type = default_type
nova_catalog_info=compute:nova:publicURL
backup_driver = cinder.backup.drivers.swift.SwiftBackupDriver

[vnx_backend_rep]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
initiator_auto_registration = True
volume_backend_name = vnx_backend_rep
volume_driver=cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver
#destroy_empty_storage_group = True
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
storage_vnx_pool_names = Pool_0
check_max_pool_luns_threshold = True
ignore_pool_full_threshold = True
replication_device = backend_id:APM00152904560,san_ip:192.168.1.94,san_login:sysadmin,san_password:sysadmin,naviseccli_path:/opt/Navisphere/bin/naviseccli,storage_vnx_authentication_type:global,storage_vnx_security_file_dir:

[vnx_backend_1]
default_timeout = 2000
destroy_empty_storage_group=False
force_delete_lun_in_storagegroup=True
initiator_auto_registration = True
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_ip = 192.168.1.50
san_login = sysadmin
san_password = sysadmin
storage_protocol = iSCSI
storage_vnx_authentication_type=global
use_multipath_for_image_xfer = True
volume_backend_name = vnx_backend_1
volume_driver = cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver

[vnx_backend_2]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = vnx_backend_2
initiator_auto_registration = True
#destroy_empty_storage_group = True
volume_driver=cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52

[storagepool_0]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = storagepool_0
volume_driver=cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
storage_vnx_pool_name = DAILY_1, invalid
initiator_auto_registration = True
check_max_pool_luns_threshold = True
ignore_pool_full_threshold = True

[storagepool_1]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = storagepool_1
volume_driver=cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
san_secondary_ip = 192.168.1.52
storage_vnx_pool_names = DAILY_2
initiator_auto_registration = True
iscsi_initiators = {"ubuntu-server7":["192.168.1.7"]}

[storagepool_fc]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = storagepool_fc
volume_driver = cinder.volume.drivers.emc.emc_cli_fc.EMCCLIFCDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
san_secondary_ip = 192.168.1.53
storage_vnx_pool_names = DAILY_1
initiator_auto_registration = True
check_max_pool_luns_threshold = True
ignore_pool_full_threshold = True
io_port_list = a-1,b-1

[storagepool_scsi]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = storagepool_scsi
volume_driver=cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
storage_vnx_pool_name = DAILY_2
initiator_auto_registration = True
force_delete_lun_in_storagegroup = True
check_max_pool_luns_threshold = True
ignore_pool_full_threshold = True
io_port_list=a-4-0,b-4-0

[multiple_pools]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = multiple_pools
volume_driver=cinder.volume.drivers.emc.emc_cli_iscsi.EMCCLIISCSIDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
storage_vnx_pool_names = DAILY_1, DAILY_2
initiator_auto_registration = True
check_max_pool_luns_threshold = True
ignore_pool_full_threshold = True

[storagepool_fc_2]
use_multipath_for_image_xfer = True
num_volume_device_scan_tries = 5
volume_backend_name = storagepool_fc_2
volume_driver = cinder.volume.drivers.emc.emc_cli_fc.EMCCLIFCDriver
default_timeout = 20
naviseccli_path = /opt/Navisphere/bin/naviseccli
san_password = sysadmin
san_login = sysadmin
san_ip = 192.168.1.52
san_secondary_ip = 192.168.1.53
storage_vnx_pool_names = DAILY_1
initiator_auto_registration = True
check_max_pool_luns_threshold = True
ignore_pool_full_threshold = True

[unity_backend_1]
storage_protocol = iSCSI
san_ip = 10.245.101.39
san_login = admin
san_password = Password123!
volume_driver = cinder.volume.drivers.dell_emc.unity.Driver
volume_backend_name = unity_backend_1

[database]
max_pool_size=60
max_overflow=90


[[post-config|$MANILA_CONF]]

[DEFAULT]
enabled_share_backends = Unity
enabled_share_protocols = NFS,CIFS

[Unity]
emc_nas_server = 10.245.101.39
emc_nas_login = admin
emc_nas_password = Password123!
share_driver = manila.share.drivers.dell_emc.driver.EMCShareDriver
emc_share_backend = unity
unity_ethernet_ports = spa_eth2
unity_server_meta_pool = Manila_Pool
unity_share_data_pools = Manila_Pool
driver_handles_share_servers = True


[[post-config|/$Q_PLUGIN_CONF_FILE]]
[ml2]
tenant_network_types = vlan,flat
[ml2_type_flat]
flat_networks = public_eth0
[ml2_type_vlan]
network_vlan_ranges = public_eth1:115:119

[ovs]
# modify to below line after stack
# bridge_mappings = public_eth0:br-ex,public_eth1:br-eth1
bridge_mappings = public_eth0:br-ex
enable_tunneling = False

$ ./stack.sh
```