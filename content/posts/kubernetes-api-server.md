---
title: "Kubernetes API Server"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "kubernetes",
    "k8s",
    "api server",
]

comment: true
toc: true
auto_collapse_toc: true
---

Although kubelet could watch a directory and run pods found configured under that directory, but in a Kubernetes cluster, **kubelet will get most its pods to run from the Kubernetes API server**.

Kubernetes stores all its cluster state in `etcd`, a distributed data store with a strong consistency model. This state includes what nodes exist in the cluster, what pods should be running, which nodes they are running on, and a whole lot more. **The API server is the only Kubernetes component that connects to `etcd`**; all the other components must go through the API server to work with cluster state.

### Start an `etcd`

It's easy to run `etcd` in a container.

```console
$ # example of starting etcd
$ mkdir etcd-data
$ docker run --volume=$PWD/etcd-data:/default.etcd \
    --detach --net=host quay.io/coreos/etcd > etcd-container-id
```
**NOTE: Use host networking makes the API server can talk to it at 127.0.0.1**

### Start the API server

```console
$ # download the apiserver of version 1.14.4
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.14.4/bin/linux/amd64/kube-apiserver

$ chmod +x kube-apiserver

$ # --disable-admission-plugins=ServiceAccount disables the authentication
$ sudo ./kube-apiserver --etcd-servers=http://127.0.0.1:2379 \
    --service-cluster-ip-range=10.0.0.0/16 \
    --disable-admission-plugins=ServiceAccount
```

Check what nodes in our cluster:
```console
$ curl --stderr /dev/null http://localhost:8080/api/v1/nodes | jq '.items'
[]
```

Check what pods our cluster is running:
```console
$ curl --stderr /dev/null http://localhost:8080/api/v1/pods | jq '.items'
[]
```

### Add a node

```console
$ # cluster server is the API server
$ cat kubelet.conf
apiVersion: v1
clusters:
- cluster:
    server: http://127.0.0.1:8080
  name: default-cluster
contexts:
- context:
    cluster: default-cluster
  name: default-context
current-context: default-context
kind: Config
preferences: {}

$ sudo ./kubelet \
    --kubeconfig=/home/ryan/git/try/kube-apiserver/kubelet.conf \
    --fail-swap-on=false
```

Check that we have a node in our cluster:
```console
$ curl --stderr /dev/null http://localhost:8080/api/v1/nodes | jq '.items'                       
[
  {
    "metadata": {
      "name": "mint-dev-vm",
      "selfLink": "/api/v1/nodes/mint-dev-vm",
      "uid": "88944812-a21f-11e9-9723-000c2905a808",
      "resourceVersion": "5833",
      "creationTimestamp": "2019-07-09T07:59:53Z",
      "labels": {
......
```

### Run a pod via the API server

**NOTE: pay attention to `nodeName` below the `spec:` line, it schedules the pods to the node where our `kubelet` is running manually. Normally, Kubernetes `kube-scheduler` does the job.**
```console
$ cat nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: mint-dev-vm
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

$ # --data-binary is needed for inputting data in yaml file
$ curl --stderr /dev/null \
    --request POST http://localhost:8080/api/v1/namespaces/default/pods \
    -H "Content-Type: application/yaml" --data-binary @nginx.yaml
```

Check what pods our cluster is running:
```console
$ curl --stderr /dev/null http://localhost:8080/api/v1/namespaces/default/pods | jq '.items'
[
  {
    "metadata": {
      "name": "nginx",
      "namespace": "default",
      "selfLink": "/api/v1/namespaces/default/pods/nginx",
      "uid": "a8d6c4f5-a2ba-11e9-8c9c-000c2905a808",
      "resourceVersion": "5739",
      "creationTimestamp": "2019-07-10T02:30:19Z"
    },
    ......
      "podIP": "172.17.0.2",
......
```

And `kubelet` will poll the info of pods to create from the API server and bring the containers for pods.
```console
$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
cb57ac67ed38        busybox                "/bin/sh -c 'while t…"   15 minutes ago      Up 15 minutes                           k8s_log-truncator_nginx_default_a8d6c4f5-a2ba-11e9-8c9c-000c2905a808_0
e2f9ec6c9e7b        nginx                  "nginx -g 'daemon of…"   15 minutes ago      Up 15 minutes                           k8s_nginx_nginx_default_a8d6c4f5-a2ba-11e9-8c9c-000c2905a808_0
b3f2388ff746        k8s.gcr.io/pause:3.1   "/pause"                 15 minutes ago      Up 15 minutes                           k8s_POD_nginx_default_a8d6c4f5-a2ba-11e9-8c9c-000c2905a808_0
faab0d697e32        quay.io/coreos/etcd    "/usr/local/bin/etcd"    19 hours ago        Up 19 hours                             mystifying_morse
```

We could use `podIP` to access the running nginx.
```console
$ curl --stderr /dev/null http://172.17.0.2                                                           
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

## `kubectl` instead of `curl` and `jq`

Talking to the API server's REST directly with `curl` and `jq` isn't the best user experience. Then `kubectl` is introduced which acts as a command line client.

```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.14.4/bin/linux/amd64/kubectl

$ chmod +x kubectl

$ ./kubectl get nodes           
NAME          STATUS   ROLES    AGE   VERSION
mint-dev-vm   Ready    <none>   19h   v1.14.4

$ ./kubectl get pods 
NAME    READY   STATUS    RESTARTS   AGE
nginx   2/2     Running   0          50m
```
