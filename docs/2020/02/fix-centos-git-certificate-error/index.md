# Fix the git error on CentOS


The error is related to corp certificate.

1. `sudo yum install -y ca-certificates`
1. download the emc certificates from: https://eos2git.cec.lab.emc.com/liangr/corp-notes/blob/master/emc_cert/emc_cacert.crt
1. `cp emc_cacert.crt /etc/pki/ca-trust/source/anchors/`
1. run command: `update-ca-trust extract`
