# Yum Localinstall

```bash
[root@4994829af1cb osp-deploy]# yum localinstall ./*.rpm
Loaded plugins: fastestmirror, ovl
Examining ./python2-bitmath-1.3.1-1.el7.noarch.rpm: python2-bitmath-1.3.1-1.el7.noarch
Marking ./python2-bitmath-1.3.1-1.el7.noarch.rpm to be installed
Examining ./python2-cachez-0.1.2-1.el7.noarch.rpm: python2-cachez-0.1.2-1.el7.noarch
Marking ./python2-cachez-0.1.2-1.el7.noarch.rpm to be installed
Examining ./python2-persist-queue-0.2.3-1.el7.noarch.rpm: python2-persist-queue-0.2.3-1.el7.noarch
Marking ./python2-persist-queue-0.2.3-1.el7.noarch.rpm to be installed
Examining ./python2-retryz-0.1.8-1.el7.noarch.rpm: python2-retryz-0.1.8-1.el7.noarch
./python2-retryz-0.1.8-1.el7.noarch.rpm: does not update installed package.
Examining ./python2-storops-0.5.5-1.el7.noarch.rpm: python2-storops-0.5.5-1.el7.noarch
Marking ./python2-storops-0.5.5-1.el7.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package python2-bitmath.noarch 0:1.3.1-1.el7 will be installed
---> Package python2-cachez.noarch 0:0.1.2-1.el7 will be installed
---> Package python2-persist-queue.noarch 0:0.2.3-1.el7 will be installed
---> Package python2-storops.noarch 0:0.5.5-1.el7 will be installed
--> Processing Dependency: PyYAML for package: python2-storops-0.5.5-1.el7.noarch
Loading mirror speeds from cached hostfile
 * base: dallas.tx.mirror.xygenhosting.com
 * extras: centos.mirror.lstn.net
 * updates: centos.mbni.med.umich.edu
--> Running transaction check
---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
--> Processing Dependency: libyaml-0.so.2()(64bit) for package: PyYAML-3.10-11.el7.x86_64
--> Running transaction check
---> Package libyaml.x86_64 0:0.1.4-11.el7_0 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================================================================
 Package                                      Arch                          Version                                Repository                                                        Size
==========================================================================================================================================================================================
Installing:
 python2-bitmath                              noarch                        1.3.1-1.el7                            /python2-bitmath-1.3.1-1.el7.noarch                              295 k
 python2-cachez                               noarch                        0.1.2-1.el7                            /python2-cachez-0.1.2-1.el7.noarch                                55 k
 python2-persist-queue                        noarch                        0.2.3-1.el7                            /python2-persist-queue-0.2.3-1.el7.noarch                         53 k
 python2-storops                              noarch                        0.5.5-1.el7                            /python2-storops-0.5.5-1.el7.noarch                              1.7 M
Installing for dependencies:
 PyYAML                                       x86_64                        3.10-11.el7                            base                                                             153 k
 libyaml                                      x86_64                        0.1.4-11.el7_0                         base                                                              55 k

Transaction Summary
==========================================================================================================================================================================================
Install  4 Packages (+2 Dependent packages)

Total size: 2.3 M
Total download size: 208 k
Installed size: 2.9 M
Is this ok [y/d/N]: y
Downloading packages:
(1/2): libyaml-0.1.4-11.el7_0.x86_64.rpm                                                                                                                           |  55 kB  00:00:01     
(2/2): PyYAML-3.10-11.el7.x86_64.rpm                                                                                                                               | 153 kB  00:00:03     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                      67 kB/s | 208 kB  00:00:03     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : python2-bitmath-1.3.1-1.el7.noarch                                                                                                                                     1/6 
  Installing : python2-persist-queue-0.2.3-1.el7.noarch                                                                                                                               2/6 
  Installing : python2-cachez-0.1.2-1.el7.noarch                                                                                                                                      3/6 
  Installing : libyaml-0.1.4-11.el7_0.x86_64                                                                                                                                          4/6 
  Installing : PyYAML-3.10-11.el7.x86_64                                                                                                                                              5/6 
  Installing : python2-storops-0.5.5-1.el7.noarch                                                                                                                                     6/6 
  Verifying  : libyaml-0.1.4-11.el7_0.x86_64                                                                                                                                          1/6 
  Verifying  : python2-cachez-0.1.2-1.el7.noarch                                                                                                                                      2/6 
  Verifying  : python2-persist-queue-0.2.3-1.el7.noarch                                                                                                                               3/6 
  Verifying  : python2-storops-0.5.5-1.el7.noarch                                                                                                                                     4/6 
  Verifying  : PyYAML-3.10-11.el7.x86_64                                                                                                                                              5/6 
  Verifying  : python2-bitmath-1.3.1-1.el7.noarch                                                                                                                                     6/6 

Installed:
  python2-bitmath.noarch 0:1.3.1-1.el7         python2-cachez.noarch 0:0.1.2-1.el7         python2-persist-queue.noarch 0:0.2.3-1.el7         python2-storops.noarch 0:0.5.5-1.el7        

Dependency Installed:
  PyYAML.x86_64 0:3.10-11.el7                                                               libyaml.x86_64 0:0.1.4-11.el7_0                                                              

Complete!
[root@4994829af1cb osp-deploy]# cd / 
```
