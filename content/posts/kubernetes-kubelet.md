---
title: "Kubernetes Kubelet"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "kubernetes",
    "k8s",
    "kubelet",
]

comment: true
toc: true
auto_collapse_toc: true
---

**kubelet** is the lowest level component in Kubernetes. It's responsible for what's running on an individual machine. You can think of it **as a process watcher like `supervisord`, but focused on running containers**. It has one job: given a set of containers to run, make sure they are all running.

There are a few ways the kubelet finds pods to run:
- a directory it polls for new pod manifests to run
- a URL it polls and downloads pod manifests from
- from the Kubernetes API server

The first one above is the simplest: to run a pod, we just put a manifest file in the watched directory. Every 20 seconds, the kubelet checks for changes in the directory, and adjusts what it's running based on what it finds. This means both launching pods that are added, as well as killing ones that are removed.

```console
$ # download the kubelet of version 1.14.4
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.14.4/bin/linux/amd64/kubelet

$ sudo ./kubelet --pod-manifest-path=/home/ryan/git/try/kubelet/manifests --fail-swap-on=false
```

The kubelet will watch the directory `/home/ryan/git/try/kubelet/manifests` for pod manifests to run.

Then put a pod yaml under `manifests` folder.
```yaml
# /home/ryan/git/try/kubelet/manifests/nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /var/log/nginx
      name: nginx-logs
  - name: log-truncator
    image: busybox
    command:
    - /bin/sh
    args: [-c, 'while true; do cat /dev/null > /logdir/access.log; sleep 10; done']
    volumeMounts:
    - mountPath: /logdir
      name: nginx-logs
  volumes:
  - name: nginx-logs
    emptyDir: {}
```

Wait for several minutes (need to pull images), three containers will be brought up. Two are defined in the yaml file. And the `pause` container works as the parent container of the other two.

```console
$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS               NAMES
b4bdd1f525e7        busybox                "/bin/sh -c 'while t…"   22 seconds ago       Up 21 seconds                           k8s_log-truncator_nginx-mint-dev-vm_default_d44d63ad82a229b51355cefe1e2e8321_0
9bb7c1339420        nginx                  "nginx -g 'daemon of…"   33 seconds ago       Up 32 seconds                           k8s_nginx_nginx-mint-dev-vm_default_d44d63ad82a229b51355cefe1e2e8321_0
ef71cebec1f8        k8s.gcr.io/pause:3.1   "/pause"                 About a minute ago   Up About a minute                       k8s_POD_nginx-mint-dev-vm_default_d44d63ad82a229b51355cefe1e2e8321_0
```

You can check the `pause` container sets up the network which is shared by other two containers.
```console
$ docker inspect --format '{{ .NetworkSettings.IPAddress  }}' b4bdd1f525e7                               

$ docker inspect --format '{{ .NetworkSettings.IPAddress  }}' 9bb7c1339420            

$ docker inspect --format '{{ .NetworkSettings.IPAddress  }}' ef71cebec1f8            
172.17.0.2

$ docker inspect --format '{{ .HostConfig.NetworkMode }}' b4bdd1f525e7            
container:ef71cebec1f8e7b034317dd76cbf0c9e318973f797d2dee02e036abbfa9f2a81

$ docker inspect --format '{{ .HostConfig.NetworkMode }}' 9bb7c1339420            
container:ef71cebec1f8e7b034317dd76cbf0c9e318973f797d2dee02e036abbfa9f2a81

$ docker inspect --format '{{ .HostConfig.NetworkMode }}' ef71cebec1f8            
default
```