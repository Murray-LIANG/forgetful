# Kubernetes Networking


## Different Networking Model

- Container-to-container
- Pod-to-pod
- Pod-to-service
- Internet-to-service

```console
+-------------------------------------------------------------------+         +-------------------------------------------------------------------+
|                                                                   |         |                                                                   |
|   +-----------------------------------+                           |         |   +-----------------------------------+                           |
|   | +-------------+   +-------------+ |                           |         |   | +-------------+   +-------------+ |                           |
|   | | Container.A |   | Container.B | |                           |         |   | | Container.A |   | Container.B | |                           |
|   | +-------------+   +-------------+ |                           |         |   | +-------------+   +-------------+ |                           |
|   |                                   |                           |         |   |                                   |                           |
|   |       +------------------+        |                           |         |   |       +------------------+        |                           |
|   |       | Parent Container |        |                           |         |   |       | Parent Container |        |                           |
|   |       | (i.e. pause)     |        |                           |         |   |       | (i.e. pause)     |        |                           |
|   |       +------------------+        |                           |         |   |       +------------------+        |                           |
|   |                                   |   +-------------------+   |         |   |                                   |   +-------------------+   |
|   |   +-----------------+             |   |                   |   |         |   |   +-----------------+             |   |                   |   |
|   |   | eth0 172.17.0.2 |       Pod.1 |   | 172.17.0.3  Pod.2 |   |         |   |   | eth0 172.18.0.2 |       Pod.1 |   | 172.18.0.3  Pod.2 |   |
|   +---+--+--------------+-------------+   +----------+--------+   |         |   +---+--+--------------+-------------+   +----------+--------+   |
|          |                                           |            |         |          |                                           |            |
|          |                                           |            |         |          |                                           |            |
|      +---+----+                                 +----+---+        |         |      +---+----+                                 +----+---+        |
|      | veth.x |                                 | veth.y |        |         |      | veth.x |                                 | veth.y |        |
|   +--+--------+---------------------------------+--------+--+     |         |   +--+--------+---------------------------------+--------+--+     |
|   |                                                         |     |         |   |                                                         |     |
|   |                         docker0                         |     |         |   |                         docker0                         |     |
|   |                        172.17.0.1                Bridge |     |         |   |                        172.18.0.1                Bridge |     |
|   +--------+------------------------------------------------+     |         |   +--------+------------------------------------------------+     |
|            |                                                      |         |            |                                                      |
|            |                                                      |         |            |                                                      |
|   +--------+--------+                                             |         |   +--------+--------+                                             |
|   |       eth0      |                                             |         |   |       eth0      |                                             |
|   | 192.168.180.167 |                                      Node.1 |         |   | 192.168.180.168 |                                      Node.2 |
+---+--------+--------+---------------------------------------------+         +---+--------+--------+---------------------------------------------+
             |                                                                             |
             |                                         +------------------+                |
             |                                         |                  |                |
             +-----------------------------------------+ Switch or Router +----------------+
                                                       +------------------+

```

### Container to Container

#### Containers in the same pod
Containers in the same pod share the same network namespace. The services in these containers use different ports. These containers can communicate easily via `localhost`. In Kubernetes, a parent container mostly `pause` container sets up the network for all containers in the same pod.

#### Containers across different pods
When a container needs to visit another one in different pod, it accesses the service via the pod/service IP and port. It relates to the pod-pod networking below.

### Pod to Pod

#### Pods in the same node

One implementation is using Linux Bridge. For example, create a bridge named `docker0` in the root network namespace, then connect the pod's `eth0` to the bridge `docker0` via veth pair. The bridge connects the node's `eth0` and pods' `eth0` in a huge network like switch. Then the pod `172.17.0.2` can communicate with `172.17.0.3` with the support of ARP.

#### Pods across different nodes

Pods across different nodes can communicate depending on:
1. the route table configurations of these nodes
2. the networking between these nodes.

For example, the route table configured on `Node.1` routes the packet to its `eth0`. Then networking between nodes routes the packet to `Node.2`. Finally the route table configured on `Node.2` routes the packet to the bridge then to the correct pod in `Node.2`.

**NOTE: the route table configuration can also be isolated via network namespaces.**

### Pod to Service

In Kubernetes, `service` is a deployment to handle the issue of pod un-durable IP address. When node reboots, application crashes and scaling would cause pod's IP address changing. The `Cluster IP` of `service` works as VIP. Traffic addressed to this IP will be load-balanced to the set of backing pods associated with the `service`.

The route from `service` to pod and the load-balanced of pods are controlled by `iptables`. The packet flows like:
1. The source pod's `eth0` port. i.e. src in the packet: `pod-1`, dst: `service-1`.
2. `veth0` of the Linux bridge. src and dst in the packet are same as in step #1.
3. The `iptables` configured in root network namespace changes the **destination** of the packet from `service` IP to `pod` IP. **src in the packet: `pod-1`, dst: `pod-5`.**
4. The updated packet is sent out through node's `eth0`.
5. Now the packet is outside of the node of source pod. Its source is the source pod IP and destination is the destination pod IP, instead of service IP. And it works like the pod-to-pod routing above.

**NOTE: The `iptables` is same on all nodes. And the rules related to service-pod routing are installed by `kube-proxy`.**

The responding packet is with original destination pod IP as source until it reaches the node of original source pod. With the iptables on the node, the packet's source now changes to original destination service IP.

### Internet to Service

#### Route traffic from the service out to the internet

`SNAT` is used here.
There are two `SNAT` rewriting happening. One happens in the root network namespace which rewrites the source of packet from `pod-1 IP` to `node IP`. The other happens on the gateway of nodes' network which rewrites the source of packet from `node IP` to `gateway IP`.

The responding packet will be routed to the correct pod under the help of `SNAT`.

#### Route traffic from the internet into the service

Three solutions:
- Use `NodePort` type service
- Use `LoadBalancer` type service
- Use `Ingress` controller

**NodePort service**

Each node exports the same port for the service. The request going to the node port will re-direct to the `service`, then `service` routes the packet to pods as described above.

For example, with below service's definition, the packet will be routed in path: `<node ip>:30123` -> `<cluster ip>:80` -> `<pod ip>:8080`

```yaml
# service.yaml
...
spec:
  type: NodePort
  ports:
  - port: 80            # ClusterIP port
    targetPort: 8080    # pod port
    nodePort: 30123     # node port
```

![](/images/kubernetes-networking-nodeport.png)

**LoadBalancer service**

The `NodePort` could be a single point of failure cause your client could point to a node for a service. If this node failed, the service cannot access.

You can create a `service` of `LoadBalancer` type. It's like putting a load balancer between the client and nodes. This load balancer is outside of the Kubernetes cluster.

The implementation of the LoadBalancer is provided by a `cloud provider` that knows how to create a load balancer for your service.

The request reaches the LoadBalancer. The LoadBalancer picks a Node at random. The Node routes the request to the service. Then `iptables` routes the packet to the pod.

![](/images/kubernetes-networking-loadbalancer.png)

**Ingress controller**

`Ingress` controller is a layer 7 load balancer, because it is HTTP aware, knows about URLs and paths. This allows you to segment your service traffic by URL path.

Why we need `Ingress` controllers? Because there is a load balancer for each `LoadBalancer` service, these load balancers have public IP address each. And `Ingress` only needs one public IP address to provide access for multiple services.

`Ingress` controller doesn't forward the request to related service, but pick a pod endpoint based the definition of the service.

![](/images/kubernetes-networking-ingress.png)
