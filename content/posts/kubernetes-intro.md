---
title: "Kubernetes Introduction"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "kubernetes",
    "k8s",
]

comment: true
toc: true
auto_collapse_toc: true
---

## 基本概念

### Cluster

各种资源，计算、存储、网络的集合。

### Master

调度应用放在哪里运行。为了保证高可用，可以部署多个Master。

### Node

由Master管理，负责监控并汇报容器的状态。根据Master的要求，对容器进行Life Cycle Management。

### Pod

K8S中最小的工作单元。每一个Pod可以包含一个或者多个容器。Pod中的容器作为一个整体被Master调度到一个Node上运行。

为什么需要提出Pod这一层抽象呢？因为
1. 存在多个容器天生需要紧密联系，一起工作，Pod比容器抽象更高，将多个容器封装在一个部署单元中，易于管理，比如以Pod为最小单位进行调度、扩展、资源共享、LCM。

2. Pod中的所有容器使用同一个网络的namespace，即相同的IP地址和Port空间。除了网络，还有共享的存储资源。

### Controller
K8S不会直接创建Pod，而是通过Controller来管理Pod的。Controller中定义了Pod的部署特性，比如有多少个副本，在什么样的Node上面运行。K8S提供了多种Controller,包括Deployment、ReplicaSet、DaemonSet、StatefulSet、Job等。

#### Deployment
可以通过它来部署应用，Deployment来管理Pod的多个副本，并保证Pod安装预期的状态运行。

#### ReplicaSet
Pod的多副本管理，使用Deployment会间接自动创建ReplicaSet。

#### DaemonSet
用于每个Node只运行一个Pod副本的情形。

#### StatefulSet
能保证Pod的每个副本在整个生命周期中名称是不变的。使用其他Controll当某个Pod发生故障需要删除并重新启动，Pod的名称会变化。除此之外，StatefulSet会保证副本按照固定的顺序启动，更新或者删除。

#### Job
运行结束就删除。其他方式部署的Pod会一直存在。

### Service
Deployment部署多个Pod，每个Pod有自己的IP，外界不能通过Pod的IP来访问应用，因为Pod的IP会随着Pod的重启而变化。因此，用Service定义外界访问一组特定Pod的方式。Service有自己的IP和Port，而且还为Pod提供Load Balance。

Controller和Service分别控制着Pod的运行和访问。

### Namespace
如果多个用户或者项目组共享使用一个物理的Cluster，如何隔离他们创建的Controller和Pod呢？

使用Namespace可以将一个物理的Cluster划分成多个逻辑的Cluster，每个Cluster在单独的Namespace中，他们的资源完全隔离。

K8S创建了两个默认的Namespace，default和kube-system，后者用于放置K8S自己创建的系统资源。

### kubelet, kubeadm, kubectl
- kubelet - 运行在所有节点上，负责启动Pod和容器。
- kubeadm - 用于初始化Cluster。
- kubectl - K8S命令行工具，用于部署管理应用，查看管理各种资源。

## Installation

### Create three VMs.

### Create the user `k8s` with password `welcome`, add it to `sudo` group.

### Add corp CA certs on all nodes.

https://eos2git.cec.lab.emc.com/liangr/corp-notes/blob/master/fix-pip-emc-cert.md

### Install `docker.io` on all nodes.

### Install `kubelet`, `kubeadm`, `kubectl` on all nodes. https://kubernetes.io/docs/setup/independent/install-kubeadm/

### Use Flannel network.

### Set auto bash completion for `kubectl`.

```console
$ kubectl completion bash | sudo tee /etc/bash_completion.d/k8s \
  && source /etc/bash_completion.d/k8s
``` 

## 架构

K8S Cluster由Master Node和Worker Node组成，Node上运行着若干服务。

## Master Node
运行着如下Daemon：
- kube-apiserver
- kube-scheduler
- kube-controller-manager
- etcd
- Pod Network, like flannel

### API Server（kube-apiserver）

提供HTTP/HTTPS RESTful API，即Kubernetes API。API Server是
Kubernetes Cluster的前端接口，各种客户端工具（CLI 或 UI）以及
Kubernetes其他组件可以通过它管理Cluster的各种资源。

### Scheduler（kube-scheduler）

Scheduler负责决定将Pod放在哪个Node上运行。Scheduler在调度时会充分考
虑Cluster的拓扑结构，当前各个节点的负载，以及应用对高可用、性能、数据亲
和性的需求。

### Controller Manager（kube-controller-manager）

负责管理Cluster各种资源，保证资源处于预期的状态。
Controller Manager由多种controller组成，包括replication 
controller、endpoints controller、namespace controller、
serviceaccounts controller等。

不同的controller管理不同的资源。例如replication controller管理
Deployment、StatefulSet、DaemonSet的生命周期，namespace 
controller管理Namespace资源。

### etcd

负责保存Kubernetes Cluster的配置信息和各种资源的状态信息。当数据发生
变化时，etcd会快速地通知Kubernetes相关组件。

### Pod Network

Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络，flannel是其中
一个可选方案。

## Work Node （也有直接叫Node节点的）
Pod运行的地方。K8S支持多种Container Runtime，比如Docker，rkt。
Node一般运行：
- kubelet
- kube-proxy
- Pod Network

### kubelet

是Node的agent，当Scheduler确定在某个Node上运行Pod后，会将Pod的具体
配置信息（image、volume等）发送给该节点的kubelet，kubelet根据这些信息创建和运行容器，并向Master报告运行状态。

### kube-proxy

service在逻辑上代表了后端的多个Pod，外界通过service访问Pod。service接收到的请求是如何转发到Pod的呢？这就是kube-proxy要完成的工作。

每个Node都会运行kube-proxy服务，它负责将访问service的TCP/UPD数据流转发到后端的容器。如果有多个副本，kube-proxy会实现负载均衡。

## 总结
几乎所有的service都是在Pod中运行。在`kube-system`这个namespace中。
```console
k8s@k8s-master:~$ kubectl get pod --all-namespaces -o wide
NAMESPACE     NAME                                 READY   STATUS              RESTARTS   AGE   IP            NODE         NOMINATED NODE
kube-system   coredns-576cbf47c7-7fhcz             0/1     ContainerCreating   0          33m   <none>        k8s-master   <none>
kube-system   coredns-576cbf47c7-w79m6             0/1     ContainerCreating   0          33m   <none>        k8s-master   <none>
kube-system   etcd-k8s-master                      1/1     Running             0          32m   172.16.1.8    k8s-master   <none>
kube-system   kube-apiserver-k8s-master            1/1     Running             0          32m   172.16.1.8    k8s-master   <none>
kube-system   kube-controller-manager-k8s-master   1/1     Running             0          32m   172.16.1.8    k8s-master   <none>
kube-system   kube-proxy-9r8jm                     1/1     Running             0          32m   172.16.1.13   k8s-node2    <none>
kube-system   kube-proxy-l7xcw                     1/1     Running             0          32m   172.16.1.14   k8s-node1    <none>
kube-system   kube-proxy-n8k9g                     1/1     Running             0          33m   172.16.1.8    k8s-master   <none>
kube-system   kube-scheduler-k8s-master            1/1     Running             0          32m   172.16.1.8    k8s-master   <none>
```

`kubelet`除外，他是被`systemd`管理着。
```console
k8s@k8s-master:~$ sudo systemctl status kubelet.service
[sudo] password for k8s:
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Sat 2018-10-20 10:08:27 UTC; 38min ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 19933 (kubelet)
    Tasks: 22
   Memory: 47.1M
      CPU: 1min 29.551s
   CGroup: /system.slice/kubelet.service
           └─19933 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/

```

### 应用启动流程

1. `kubectl run <name> --image <image_name> --replicas=<num>`
2. 命令发送到`API Server`.
3. `Controller Manager`创建`Deployment`。
4. `Scheduler`调度把`num`份Pod副本发往不同的Node上面创建，通过`kubelet`通信。
5. 每个接收到请求的Node通过`kubectl`来创建Pod。
6. 在没有创建`Service`时，`kube-proxy`不参与。

#### 举个例子
1. 创建一个Deployment
```console
k8s@k8s-master:~$ kubectl run nginx-deploy --image=nginx --replicas=2
```

2. Deployment创建成功
```console
k8s@k8s-master:~$ kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   2         2         2            0           18s
```

3. Deployment Controller还为这个deployment创建了一个Replication Set。
```console
k8s@k8s-master:~$ kubectl describe deploy nginx-deploy
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Sat, 20 Oct 2018 11:39:22 +0000
Labels:                 run=nginx-deploy
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=nginx-deploy
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=nginx-deploy
  Containers:
   nginx-deploy:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deploy-74f8cd9b44 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  39s   deployment-controller  Scaled up replica set nginx-deploy-74f8cd9b44 to 2
```
4. Check the replication set.
```console
k8s@k8s-master:~$ kubectl get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-74f8cd9b44   2         2         2       4m13s
```

5. 通过Replication Controller在这个Replication Set中创建了两个Pod。
```console
k8s@k8s-master:~$ kubectl describe replicaset nginx-deploy-74f8cd9b44
Name:           nginx-deploy-74f8cd9b44
Namespace:      default
Selector:       pod-template-hash=74f8cd9b44,run=nginx-deploy
Labels:         pod-template-hash=74f8cd9b44
                run=nginx-deploy
Annotations:    deployment.kubernetes.io/desired-replicas: 2
                deployment.kubernetes.io/max-replicas: 3
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/nginx-deploy
Replicas:       2 current / 2 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  pod-template-hash=74f8cd9b44
           run=nginx-deploy
  Containers:
   nginx-deploy:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  4m36s  replicaset-controller  Created pod: nginx-deploy-74f8cd9b44-c89gz
  Normal  SuccessfulCreate  4m36s  replicaset-controller  Created pod: nginx-deploy-74f8cd9b44-59cl6
```

6. 两个Pod。
```console
k8s@k8s-master:~$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-74f8cd9b44-59cl6   1/1     Running   0          5m1s
nginx-deploy-74f8cd9b44-c89gz   1/1     Running   0          5m1s
```

7. 每个Pod记录了创建Container的过程。
```console
k8s@k8s-master:~$ kubectl describe pods nginx-deploy-74f8cd9b44-59cl6                                                                                 [9/9105]
Name:               nginx-deploy-74f8cd9b44-59cl6
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               k8s-node2/172.16.1.13
Start Time:         Sat, 20 Oct 2018 11:39:22 +0000
Labels:             pod-template-hash=74f8cd9b44
                    run=nginx-deploy
Annotations:        <none>
Status:             Running
IP:                 10.244.2.2
Controlled By:      ReplicaSet/nginx-deploy-74f8cd9b44
Containers:
  nginx-deploy:
    Container ID:   docker://282e75e7b0f8237d86e92677a9272e190fc8b870924814622756b4482912e460
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:b73f527d86e3461fd652f62cf47e7b375196063bbbd503e853af5be16597cb2e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 20 Oct 2018 11:39:42 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8c8lf (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-8c8lf:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8c8lf
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                Message
  ----    ------     ----   ----                -------
  Normal  Scheduled  5m21s  default-scheduler   Successfully assigned default/nginx-deploy-74f8cd9b44-59cl6 to k8s-node2
  Normal  Pulling    5m10s  kubelet, k8s-node2  pulling image "nginx"
  Normal  Pulled     5m1s   kubelet, k8s-node2  Successfully pulled image "nginx"
  Normal  Created    5m1s   kubelet, k8s-node2  Created container
  Normal  Started    5m1s   kubelet, k8s-node2  Started container
```

## Node Selector

可以通过Label来控制Pod会被schedule到哪个Node上。
```console
## Add label to node
$ kubectl label node k8s-node1 disktype=ssd

## Remove label
$ kubectl label node k8s-node1 disktype-
```

```yaml
spec:
  ...
  template:
    ...
    spec:
      containers:
      - name: xxx
        image: xxx
      nodeSelector:
        disktype: ssd
```

## DaemonSet
与Deployment不一样，只允许一个Node上运行一个Pod副本。yaml配置文件中`kind`为`DaemonSet`。

## Service
Pod随时会死掉，而要保证应用的健壮，使用Service。Service逻辑上代表一组Pod。为了保证应用的健壮，Service的IP不变，即使下面的Pod的IP改变了。K8S维护着Service和Pod的映射。

K8S提供多种类型的Service：
- ClusterIP （默认）
- NodePort
- LoadBalancer

### ClusterIP
默认情况下，没有指定type，会创建ClusterIP类型的Service，只有Cluster内的节点和Pod可以访问。
```console
# Create the deployment
k8s@k8s-master:~$ kubectl apply -f httpd.yaml
deployment.apps/httpd created
k8s@k8s-master:~$ cat httpd.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  template:
    metadata:
      labels:
        run: httpd  # Service uses this label to group Pods
    spec:
      containers:
      - name: httpd
        image: httpd
        ports:
        - containerPort: 80  # Container uses port 80

# Check the created Pods.
k8s@k8s-master:~$ kubectl get pod -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NOD
E        NOMINATED NODE
httpd-79c4f99955-6zswp   1/1     Running   0          2m11s   10.244.2.3   k8s
-node2   <none>
httpd-79c4f99955-7bl5h   1/1     Running   0          2m11s   10.244.1.4   k8s
-node1   <none>
httpd-79c4f99955-wvfcd   1/1     Running   0          2m11s   10.244.1.3   k8s
-node1   <none>

# Check the containers in Pod.
k8s@k8s-master:~$ kubectl describe pod httpd-79c4f99955-6zswp
Name:               httpd-79c4f99955-6zswp
...
Node:               k8s-node2/172.16.1.13
Labels:             pod-template-hash=79c4f99955
                    run=httpd
...
IP:                 10.244.2.3  # The Pod IP
...
Containers:
  httpd:
    Container ID:   docker://a97c4b1fba73326f83457fc2ba4a7b11e751d31f22a9a3624
6ac20c6700cc703
    ...
    Port:           80/TCP  # Port 80 is set
    Host Port:      0/TCP
    ...
...


# Check the httpd service is up, visit using Pod IP
k8s@k8s-master:~$ curl 10.244.2.3
<html><body><h1>It works!</h1></body></html>

# Create Service
k8s@k8s-master:~$ kubectl apply -f httpd-svc.yaml
service/httpd-svc created
k8s@k8s-master:~$ cat httpd-svc.yaml
apiVersion: v1
kind: Service  # Note this line
metadata:
  name: httpd-svc
spec:
  selector:
    run: httpd
  ports:
  - protocol: TCP
    port: 8080  # Use this port to expose service
    targetPort: 80  # The port of pods use

# A cluster IP is created for the service, which is stable and
# will not change even when the Pods IP change.
k8s@k8s-master:~$ kubectl get service
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
httpd-svc    ClusterIP   10.110.229.127   <none>        8080/TCP   8s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    153m

# Pay attention to the `Endpoints` which includes all three 
# Pods IP.
k8s@k8s-master:~$ kubectl describe service httpd-svc
Name:              httpd-svc
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"httpd-svc","namespace":"default"},"spec":{"ports":[{"port":8080,"..."
Selector:          run=httpd
Type:              ClusterIP
IP:                10.110.229.127
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.3:80,10.244.1.4:80,10.244.2.3:80
Session Affinity:  None
Events:            <none>

# Works on cluster IP
k8s@k8s-master:~$ curl 10.110.229.127:8080
<html><body><h1>It works!</h1></body></html>

```

#### Service的Cluster IP怎样做到的
通过`iptables`。而且所有的iptable的rule在所有同一个Cluster的节点都一样。

```console

k8s@k8s-master:~$ sudo iptables-save | grep 10.110.229.127

# 不是源地址来自10.244.0.0/16，要访问httpd-svc（IP和port），则跳至
# MARK（貌似仅仅是MARK了一下）。换句话说，只有符合上述条件才允许走到下一
# 条rule
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.110.229.127/32 -p tcp -m comment --comment "default/httpd-svc: cluster IP" -m tcp --dport 8080 -j KUBE-MARK-MASQ

# 允许转发的，跳到对应的rule
-A KUBE-SERVICES -d 10.110.229.127/32 -p tcp -m comment --comment "default/httpd-svc: cluster IP" -m tcp --dport 8080 -j KUBE-SVC-RL3JAE4GN7VOGDGP

k8s@k8s-master:~$ sudo iptables-save | grep KUBE-SVC-RL3JAE4GN7VOGDGP

# 这样就做到了负载均衡？`probability`.以不同的概率跳至对应Pod的rule
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-FLNU5BTIQMZ5GJ3J
-A KUBE-SVC-RL3JAE4GN7VOGDGP -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-5PWFTV3INZUJOOJD
-A KUBE-SVC-RL3JAE4GN7VOGDGP -j KUBE-SEP-ATZOK6UEJOCBNV4B

```

#### DNS
每当有新的Service被创建，DNS服务会为新的Service添加一条DNS记录。Cluster内的Pod能通过`<service_name>.<namespace_name>`来访问。同一个namespace的话，可以省略。

```console
k8s@k8s-master:~$ kubectl run busybox --rm -ti --image=busybox /bin/sh
/ # wget httpd-svc:8080
Connecting to httpd-svc:8080 (10.110.229.127:8080)
index.html           100% |******************************|    45  0:00:00 ETA

/ # cat index.html
<html><body><h1>It works!</h1></body></html>

/ # nslookup httpd-svc.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   httpd-svc.default.svc.cluster.local
Address: 10.110.229.127

*** Can't find httpd-svc.default.svc.cluster.local: No answer
```

### NodePort
对Cluster节点的静态端口对外暴露服务。Cluster外面可以通过`<node_ip>:<node_port>`来访问。

```console
k8s@k8s-master:~$ cat httpd-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  type: NodePort  # Notice this type changed
  selector:
    run: httpd
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80

# Redeploy the service
k8s@k8s-master:~$ kubectl apply -f httpd-svc.yaml
service/httpd-svc configured

# Pay attention to the `PORT(S)`
# `8080` is the port ClusterIP is licsening.
# while`32309` is the Node is licsening.
k8s@k8s-master:~$ kubectl get service httpd-svc
NAME        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
httpd-svc   NodePort   10.110.229.127   <none>        8080:32309/TCP   71m

k8s@k8s-master:~$ curl k8s-node1:32309
<html><body><h1>It works!</h1></body></html>
k8s@k8s-master:~$ curl k8s-node2:32309
<html><body><h1>It works!</h1></body></html>
k8s@k8s-master:~$ curl k8s-master:32309
<html><body><h1>It works!</h1></body></html>

```
同样是通过iptables做到的。

端口号是K8S自动分配的，从30000 - 32767，可以通过在yaml中指定端口，来给定端口号。

```console
k8s@k8s-master:~$ cat httpd-svc.yaml
...
spec:
  ...
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
    nodePort: 34140  # Specified port
```

### LoadBalancer
可以配置GCP，AWS等云服务提供商，来做到load balance将流量导向Service。

## Volume
TO BE CONTINUED