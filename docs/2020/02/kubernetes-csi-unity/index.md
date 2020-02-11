# Kubernetes CSI Unity


## Setup K8S Nodes

### 1. Create three VMs.

### 2. Create the user `k8s` with password `welcome`, add it to `sudo` group.

### 3. Add corp CA certs on all nodes.

https://eos2git.cec.lab.emc.com/liangr/corp-notes/blob/master/fix-pip-emc-cert.md

### 4. Install `docker.io` on all nodes.

**NOTE: Do NOT use 18.09.2 currently (2019-04-25). It causes below errors:**
```console
root@k8s-master:~# docker pull k8s.gcr.io/kube-apiserver:v1.14.1
v1.14.1: Pulling from kube-apiserver
346aee5ea5bc: Pulling fs layer
7f0e834d5a94: Pulling fs layer
error pulling image configuration: Get https://storage.googleapis.com/us.artifacts.google-containers.appspot.com/containers/images/sha256:cfaa4ad74c379e428b953c9aa9962e25a6de470a38b3b62ea2fe
aef1bfb30e0d: remote error: tls: handshake failure
```

Need more configuration after installation.

```bash
# under root user
cat > /etc/docker/daemon.json <<EOF
{
    "dns": ["10.254.174.10", "10.104.128.235"],
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    },
    "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```

### 5. Install `kubelet`, `kubeadm`, `kubectl` on all nodes.

https://kubernetes.io/docs/setup/independent/install-kubeadm/

**NOTE: Do NOT forget to configure cgroup driver used by kubelet on Master Node**

```bash
$ sudo vim /etc/default/kubelet
KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
```

### 6. Initialize the master using Flannel network.

`kubeadm init --pod-network-cidr=10.244.0.0/16`

Switch to the user `k8s`.

```bash
# To start using your cluster, you need to run the following as a regular user:

$ mkdir -p $HOME/.kube && \
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && \
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Deploy a pod network to the cluster.
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

### 7. Join the worker nodes to the cluster.

`kubeadm join 172.16.1.8:6443 --token 60xmgt.n6ki3huo6bife2qb \
    --discovery-token-ca-cert-hash sha256:fd15de9d8ac029893cda34f4e356bb540477500fe243d6b65c947419c111c53b`

### 8. Set auto bash completion for `kubectl`.

- For current console:
```console
$ source <(kubectl completion bash)

$ source <(kubectl completion zsh)
```

- Auto load on startup

```console
$ kubectl completion bash | sudo tee /etc/bash_completion.d/k8s \
  && source /etc/bash_completion.d/k8s

$ kubectl completion zsh > "${fpath[1]}/_kubectl"
``` 

### 9. Tear down

https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#tear-down

On `k8s-master`:
```bash
for node in k8s-node1 k8s-node2 k8s-master; do \
    kubectl drain $node --delete-local-data --force --ignore-daemonsets && \
    kubectl delete node $node; \
done
```

On `k8s-master`, `k8s-node1`, `k8s-node2`:
```console
$ sudo su -
# kubeadm reset
```

## Tips

### How to downgrade `kubelet`, `kubeadm`, `kubectl`.

On all nodes:
```console
$ sudo su -

$ apt-cache policy kubeadm | less

$ apt install --allow-downgrades -y kubeadm=1.13.6-00 kubelet=1.13.6-00 kubectl=1.13.6-00
```

## Troubleshooting

### How to check the current configuration used by kubelet?

After deploy of k8s cluster, run:
```console
$ sudo kubectl proxy --port 8001 &
$ kubectl get nodes
$ curl -sSL "http://localhost:8001/api/v1/nodes/k8s-node1/proxy/configz"  | python3 -m json.tool | grep cgroup
        "cgroupsPerQOS": true,
        "cgroupDriver": "systemd",
```

### Different paths mounted in containers.
Application container:
```json
"Source": "/var/lib/kubelet/pods/33d9ee75-7307-11e9-9080-fa163ece4076/volumes/kubernetes.io~csi/pvc-6cccf68a-7304-11e9-9080-fa163ece4076/mount",
"Destination": "/var/lib/mysql",
```

Node Server container:
```bash
10.245.47.97:/csi-nfs-74baf2fc-7304-11e9-817f-fa163ecc7867   10G  1.8G  8.3G  18% /var/lib/kubelet/pods/33d9ee75-7307-11e9-9080-fa163ece4076/volumes/kubernetes.io~csi/pvc-6cccf68a-7304-11e9-9080-fa163ece4076/mount
```
```bash
"Source": "/var/lib/kubelet/pods",
"Destination": "/var/lib/kubelet/pods",
"Mode": "rshared",
```

```bash
Warning  FailedAttachVolume  10s (x8 over 74s)  attachdetach-controller  AttachVolume.Attach failed for volume "pvc-48f80977-7851-11e9-9080-fa163ece4076" : failed to load secret "default/csi-unity-secret": secrets "csi-unity-secret" is forbidden: User "system:serviceaccount:default:csi-attacher" cannot get resource "secrets" in API group "" in the namespace "default"
```

### The feature gates and required ones by CSI.
1. The raw block support needs the Kubernetes cluster to enable `BlockVolume` and `CSIBlockVolume`.
Quoted from the page (scroll to bottom): https://kubernetes-csi.github.io/docs/raw-block.html

        The BlockVolume and CSIBlockVolume feature gates need to be enabled on all Kubernetes masters and nodes.
    
        --feature-gates=BlockVolume=true,CSIBlockVolume=true...

2. Check the `BlockVolume` and `CSIBlockVolume` in the feature list: https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/. We could know that `BlockVolume` is enabled by default from 1.13 however `CSIBlockVolume` is enabled by default from 1.14. (Supposing we are using Kubernetes 1.13)

3. Thus, we need to enable `CSIBlockVolume` in Kubernetes 1.13. There are two places need to modify: `kubelet` and `apiserver`.

    After deployment, take below actions on all nodes:

    - kubelet: modify the `/etc/default/kubelet` file.
        ```bash
        $ cat /etc/default/kubelet
        KUBELET_EXTRA_ARGS="--cgroup-driver=systemd --feature-gates=CSIBlockVolume=true"
        ```
    - apiserver: modify the `/etc/kubernetes/manifests/kube-apiserver.yaml` (can be found in master node)
        ```bash
        spec:
            containers:
                - command:
                    - kube-apiserver
                    ...
                    - --feature-gates=CSIBlockVolume=true
                    ...
        ```
    
UGFzc3dvcmQxMjMh
