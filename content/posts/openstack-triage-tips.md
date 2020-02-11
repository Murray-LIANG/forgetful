---
title: "OpenStack Triage Tips"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "openstack",
    "triage",
    "tips",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Get the versions of OpenStack and Drivers

```bash
$ find . -name "*" -type f | xargs grep -n -P -i "starting.*" | grep -v "eventlet"

/var/log/cinder/scheduler.log:4490:2018-03-03 00:13:06.689 3581 INFO cinder.service [-] Starting cinder-scheduler node (version 9.1.4)
/var/log/cinder/scheduler.log:8000:2018-09-06 21:04:58.062 974844 INFO cinder.service [-] Starting cinder-scheduler node (version 9.1.4)

/var/log/cinder/volume.log:25398:2018-09-06 21:03:36.372 969733 INFO cinder.service [-] Starting cinder-volume node (version 9.1.4)
/var/log/cinder/volume.log:25434:2018-09-06 21:03:36.406 969733 INFO cinder.volume.manager [req-2122ed0d-c21c-46f1-b104-db9ea0c7934e - - - - -] Starting volume driver EMCVNXDriver (08.00.00)
```

## Get the attach duration

```bash

day='1109'

find . -name nova-compute.log-2018${day}* | xargs zgrep "initialize_connection" | grep -P "req-[\w-]+" -o > ~/openstack/980969/nova-attach-req-id-${day}.txt

mkdir ~/openstack/980969/nova-attach-req-${day}

for id in $(cat ~/openstack/980969/nova-attach-req-id-${day}.txt); do find . -name nova-compute.log-2018${day}* | xargs zgrep $id > ~/openstack/980969/nova-attach-req-${day}/$id.txt; done

for id in $(cat ~/openstack/980969/nova-attach-req-id-${day}.txt); do end=$(tail -1 ~/openstack/980969/nova-attach-req-${day}/$id.txt | cut -d " " -f1,2);end=${end#*:};start=$(head -1 ~/openstack/980969/nova-attach-req-${day}/$id.txt | cut -d " " -f1,2);start=${start#*:};echo $id $(( $(date -d "$end" +%s) - $(date -d "$start" +%s) )) >> ~/openstack/980969/nova-attach-req-${day}/duration.txt; done
```

```bash
find . -name nova-compute.log > nova-compute-log.files

cat nova-compute-log.files | xargs grep "initialize_connection" | grep -P "req-[\w-]+" -o > ./nova-attach-req-id.txt

mkdir nova-attach-req

for id in $(cat nova-attach-req-id.txt); do cat nova-compute-log.files | xargs grep $id > nova-attach-req/$id.txt; done

for id in $(cat nova-attach-req-id.txt); do end=$(tail -1 nova-attach-req/$id.txt | cut -d " " -f1,2);end=${end#*:};start=$(head -1 nova-attach-req/$id.txt | cut -d " " -f1,2);start=${start#*:};echo $id $(( $(date -d "$end" +%s) - $(date -d "$start" +%s) )) >> nova-attach-req/duration.txt; done

sort -rn -k2 -t" " nova-attach-req/duration.txt
```
