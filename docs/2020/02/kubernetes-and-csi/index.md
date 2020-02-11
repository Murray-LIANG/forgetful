# Kubernetes and CSI


## 基本概念
### Plugin
In the CSI world, it means a service that exposes gRPC endpoints. There are two plugins:
- Node plugin - a gRPC server that needs to run on the Node where the volume will be provisioned.
- Controller plugin - a gRPC server that can run anywhere (even on the master).

implementing three interfaces:
- Node interface
- Controller interface
- Identity interface

### Plugin Deployment
The `Node` and `Controller` plugin could be released as one binary or in two binaries.

Having a single binary, that contains a single gRPC server is much easier to maintain and still has all the benefits of the individual plugins.

Depending on where you deploy, it can act as a `Node` plugin or as a `Controller` plugin. More on this later.

### Sidecar Containers
Sidecar containers extend and enhance the `main` container. They exist in the same pod as the `main` container sharing storage and network with it.

### Kubernetes Core, Kubernetes External Component, 3rd-party External Component
The vendor CSI plugin works with Kubernetes external components which is also a component split from Kubernetes core components.

Some benefits of splitting vendor CSI plugins:
- Vendor CSI plugins don't coupled and depend on Kubernetes releases.
- Kubernetes developers are no longer responsible for testing and maintaining all vendor plugins, instead of just testing and maintaining a stable plugin API.
- Bugs in vendor plugins will not crash critical Kubernetes components, instead of the plugin itself.
- Privileges of vendor plugins are controlled.
- Vendor plugins could be released just as a binary instead of source codes.

## Kubernetes External Component
This is completely implemented and maintained by the Kubernetes team. These extend kubernetes actions outside of kubernetes.

- Driver registrar
  
  a sidecar container that registers the CSI driver with kubelet, and adds the drivers custom NodeId to a label on the Kubernetes Node API Object. It does this by communicating with the Identity service on the CSI driver and also calling the the CSI GetNodeId operation.

- External provisioner 
  
  a sidecar container that watches Kubernetes PersistentVolumeClaim objects and triggers CSI CreateVolume and DeleteVolume operations against a driver endpoint.

- External attacher 

  a sidecar container that watches Kubernetes VolumeAttachment objects and triggers CSI ControllerPublish and ControllerUnpublish operations against a driver endpoint.

## Storage Vendor / 3rd-party External Component
Each vendor should implement their respective APIs into gRPC service functions, including:
- CSI Identity interface
- CSI Controller interface
- CSI Node interface

### CSI Identity Interface
- GetPluginInfo
- GetPluginCapabilities
- Probe - called by the CO just to check whether the plugin is running or not.

### CSI Controller Interface
- CreateVolume
- DeleteVolume
- ControllerPublishVolume - initialize connection of OpenStack
- ControllerUnpublishVolume - terminate connection of OpenStack
- ValidateVolumeCapabilities
- ListVolumes
- GetCapacity
- ControllerGetCapabilities

### CSI Node Interface
- NodeStageVolume

    This method is called by the CO to temporarily mount the volume to a staging path. Usually this staging path is a global directory on the node. In Kubernetes, after it’s mounted to the global directory, you mount it into the pod directory (via `NodePublishVolume`). The reason that mounting is a two step operation is because Kubernetes allows you to use a single volume by multiple pods. This is allowed when the storage system supports it (say NFS) or if all pods run on the same node. One thing to note is that you also need to format the volume if it’s not formatted already. Keep that in mind.

- NodeUnstageVolume - the reverse of `NodeStageVolume`
- NodePublishVolume

    This method is called to mount the volume from staging to target path. Usually what you do here is a bind mount. A bind mount allows you to mount a path to a different path (instead of mounting a device to a path). In Kubernetes, this allows us for example to use the mounted volume from the staging path (i.e global directory) to the target path (pod directory). Here, formatting is not needed because we already did it in `NodeStageVolume`.

- NodeUnpublishVolume - the reverse of `NodeUnpublishVolume`
- NodeGetId
- NodeGetCapabilities

### Topology
![](/images/kubernetes-and-csi.png)

### Implementation Tips

#### Idempotent
All functions have to be idempotent.

#### Internal API
For long time consuming actions like attaching a volume, need to implement functions such as `waitForAttach`.

#### Logging System
- logrus
- [humanlog](https://github.com/aybabtme/humanlog)

#### Logging using gRPC Interceptor
You need to log the errors, or you won't know what happens from the plugin side (unless checking the events in Kubernetes using `kubectl describe`).

And if you use normal `log.Error` to log, you'll need add one line of code before the `return` statement of every function.

A better way is to log the errors via a gRPC interceptor, which is like a Go http/handler middleware.

Only need to add it in when creating a server.
```go
errInterceptor := func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
	resp, err := handler(ctx, req)
	if err != nil {
		log.WithError(err).WithField("method", info.FullMethod).Error("method failed")
	}
	return resp, err
}

srv = grpc.NewServer(grpc.UnaryInterceptor(errInterceptor))
srv.Serve() 
// ...
```

## Deploy CSI Plugin to Kubernetes

`Node plugin` needs to be on all nodes because it needs to be able to `format` and `mount` the volume that is attached to a given volume.

`Controller plugin` needs to run as single copy.

You could start the plugins as a system service like using `systemd`. If you're already running a Kubernetes cluster, its best to deploy them as Kubernetes primitives.

The way to deploy is as: deploy the `Node` plugin as a `DaemonSet` and the `Controller` plugin as a `StatefulSet`.

### `Node` Plugin as a `DaemonSet`
This ensures that a copy of a Pod runs on all nodes.

### `Controller` Plugin as a `StatefulSet`
`SstatefulSet` with `1` replicas could ensure a stable scaling, that means that if the pod dies or we do an update, it never creates a second pod before the first one is fully shutdown and terminated. This is very important, because this guarantees that only a single copy of the `Controller` plugin will run.

### Release Plugins in Two Separated Binaries
One binary satisfies the `Node` plugin and another one satisfies `Controller` plugin.

Need two docker images.

### Release Plugins in a Unified Binary
Specify the same docker image for both `DaemonSet` (`Node` plugin) and `StatefulSet` (`Controller` plugin).

For detail implementation, register all you implementations of interfaces within a single gRPC server inside a Go `main` function.
```go
// returns a struct instance that satisfies Node, Controller and 
// Identity interfaces
d := NewDriver()

srv = grpc.NewServer(grpc.UnaryInterceptor(errInterceptory))
csi.RegisterIdentityServer(srv, d)
csi.RegisterControllerServer(srv, d)
csi.RegisterNodeServer(srv, d)

listener, err := net.Listen("unix://", "/var/lib/kubelet/plugins/com.digitalocean.csi.dobs/csi.sock")
if err != nil {
	return fmt.Errorf("failed to listen: %v", err)
}

return srv.Serve(listener)
```

### How Kubernetes (or Other CO) Know Which Pod is `Node` or `Controller`?

The answer is the sidecar containers, the Kubernetes external components metioned above.
- driver-registrar - register the plugin
- external-provisioner - provisioning volumes
- external-attacher - attaching/detaching volumes (not including mounting or connecting)

Deploy `Node` plugin with the `driver-registrar`. `driver-registrar` only registers the plugin. It doesn't make any calls to the `Node` interface methods. All calls to `Node` plugin are made via `kubelet` itself.

Deploy `Controller` plugin with the `external-provisioner` and `external-attacher`. 

[DigitalOcean example](https://github.com/digitalocean/csi-digitalocean/blob/master/deploy/kubernetes/releases/csi-digitalocean-v0.1.1.yaml).

## How CSI Drivers Register to Kubelet

1. kubelet has a `pluginwatcher` which keeps scanning the folder: `/var/lib/kubelet/plugins_registry/` to find new registered plugins.

    https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/pluginmanager/pluginwatcher/plugin_watcher.go#L71

2. Unity CSI plugin registers itself to kubelet by creating a sock file `/registration/unity-csi.dell.com-reg.sock` while `/registration/` is mounted from host volume: `/var/lib/kubelet/plugins_registry/`. This part is done by `node-driver-registrar` running in a sidecar container.

3. kubelet's `pluginwatcher` will call `GetInfo` via socket `/var/lib/kubelet/plugins_registry/unity-csi.dell.com-reg.sock`.

4. kubelet directly issues  CSI calls (like `NodeStageVolume`, `NodePublishVolume`, etc.) to Unity CSI drivers via socket `/var/lib/kubelet/plugins/unity-csi.dell.com/csi.sock` (this host volume is mounted to container as `/csi/csi.sock`) to mount and unmount volumes.

5. Other Kubernetes API (like volume create, volume attach, volume snapshot, etc.) will be sent to Kubernetes master. Unity CSI plugin watches these API requests via `external-provisioner` and `external-attacher` sidecar containers. These sidecar containers will send requests to Unity CSI plugin via a socket (it is a Empty volume inside the pod).

