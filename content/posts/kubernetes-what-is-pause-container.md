---
title: "Kubernetes What is Pause Container"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "kubernetes",
    "k8s",
    "pause container",
]

comment: true
toc: true
auto_collapse_toc: true
---

## Why We Need Pods

Docker supports containers, which are great for deploying single units of software. However, this model can become a bit cumbersome when you want to **run multiple pieces of software together**.
You often see this when developers create Docker images that use supervisord as an entrypoint to start and manage multiple processes. For production systems, many have found that it is instead more useful to deploy those applications **in groups of containers** that are partially isolated and partially share on environment.

Kubernetes provides a clean abstraction called **pods** for just this use case. It hides the complexity of Docker flags and the need to babysit the containers, shared volumes, and the like. It also hides differences between container runtimes.

In principle, anyone can configure Docker to control the level of sharing between groups of containers -- you just have to **create a parent container**, know exactly the right flag setting to use to **create new containers that share the same environment**, and then **manage the lifetime of those containers**.

In Kubernetes, the `pause` container serves as the `parent container` for all of the containers in your pod. The `pause` container has **two core responsibilities**.
- First it serves as the basis of **Linux namespace sharing** in the pod. 
- And second, with PID namespace sharing enabled, it serves as **PID 1** for each pod and **reaps zombie processes**. NOTE: PID namespace sharing can be disabled (which is default value since Kubernetes 1.8).

## Basic of Namespaces Sharing

In Linux, when you run a new process, **the process inherits its namespaces from the parent process**. The way that you run a process in a new namespace is by `unsharing` the namespace with the parent process thus creating a new namespace.

```console
$ # run a shell in new PID, UTS, IPC, and mount namespaces.
$ sudo unshare --pid --uts --ipc --mount -f chroot rootfs /bin/sh
```

Once the process is running, you can add other processes to the process's namespace to form a pod. New processes can be added to an existing namespace using the `setns` system call.

Containers in a pod share namespaces among them. Docker lets you automate the process a bit so let's look at an example of how to create a pod from scratch by using the `pause` container and sharing namespaces.

### First start the `pause` container with Docker so that we can add our containers to the pod.

```console
$ docker run -d --name pause -p 8080:80 gcr.io/google_containers/pause-amd64:3.0
```

### Run `nginx` container and add it to our pod. 

```console
$ cat <<EOF >> nginx.conf
> error_log stderr;
> events { worker_connections  1024; }
> http {
>     access_log /dev/stdout combined;
>     server {
>         listen 80 default_server;
>         server_name example.com www.example.com;
>         location / {
>             proxy_pass http://127.0.0.1:2368;
>         }
>     }
> }
> EOF
$ docker run -d --name nginx \
    -v `pwd`/nginx.conf:/etc/nginx/nginx.conf \
    --net=container:pause \
    --ipc=container:pause \
    --pid=container:pause nginx
```

**NOTE**
- `nginx` container uses the `net`, `ipc`, `pid` namespaces of `pause` container.
- The port mapping `8080`(host) to `80`(container) is set on the parent container - `pause`, because other containers share the network namespace with `pause`.

### Run `ghost` container as our application server.

```console
$ docker run -d --name ghost \
    --net=container:pause \
    --ipc=container:pause \
    --pid=container:pause ghost
```

### If you access `http://localhost:8080`, you should be able to see `ghost` running through an `nginx` proxy because the network namespace is shared among these three containers - `pause`, `nginx` and `ghost`.

## How `pause` Container Reaps Zombies

In Linux, processes in a PID namespace form a tree with each process having a parent process. Only one process at the root of the tree doesn't really have a parent. This is the `init` process, which has `PID 1`.

Processes can start other processes using the `fork` and `exec` syscalls. When they do this, the new process's parent is the process that called the `fork` syscall. `fork` is used to start another copy of the running process and `exec` is used to replace the current process with a new one, keeping the same PID.

In order to run a totally separate application you need to run the `fork` and `exec` syscalls.
1. A process creates a new copy of itself as a child process with a new PID using `fork`.
2. When the child process runs it checks if it's the child process and runs `exec` to replace itself with the one you actually want to run.

Each process has an entry in the OS process table. This records info about the process's state and exit code. **When a child process has finished running, its process table entry remains until the parent process has retrieved its exit code using the `wait` syscall. This is called `reaping` zombie processes**.

Zombie processes are processes that have stopped running but **their process table entry still exists because the parent process hasn't retrieved it via the `wait` syscall**. Technically each process that terminates is a zombie for a very short period of time but they could live for longer.

Longer lived zombie processes occur when parent processes don't call the `wait` syscall after the child process has finished. One situation where this occurs is **when the parent process is poorly written and simply omits the `wait` call** or **when the parent process dies before the child and the new parent process does not call `wait` syscall on it**. When a process' parent dies before the child, the OS assigns the child process to the `init` process or **PID 1** (may be another process instead of `init`). i.e. The `init` process `adopts` the child process and becomes its parent. This means that now when the child process exits then new parent (`init`) must call `wait` to get its exit code or its process table entry remains forever and it becomes a zombie.

In containers, one process must be the init process for each PID namespace. With Docker, each container usually has its own PID namespace and **the `ENTRYPOINT` process is the `init` process**. For Kubernetes pods, a container can be made to run in another container's namespace. In this case, one container must assume the role of the `init` process, while others are added to the namespace as children of the `init` process.

For example,

```console
$ docker run -d --name nginx -v `pwd`/nginx.conf:/etc/nginx/nginx.conf -p 8080:80 nginx
$ docker run -d --name ghost --net=container:nginx --ipc=container:nginx --pid=container:nginx ghost
```
**`nginx` is assuming the role of `PID 1` and `ghost` is added as a child process of `nginx`**. This is mostly fine, but technically `nginx` is now responsible for any children that `ghost` orphans. If, for example, `ghost` forks itself or runs child processes using `exec`, and crashes before the child finishes, then those children will be adopted by `nginx`. However, **`nginx` is not designed to be able to run as an `init` process and reap zombies. That means we could potentially have lots of them and they will last for the life of that container**.

In Kubernetes pods, containers are run in much the same way as above, but there is **a special `pause` container that is created for each pod**. This `pause` container runs a very simple process that performs no function but essentially sleeps forever (calling `pause()`). It also performs one other important function. It **assumes the role of PID 1 and will reap any zombies by calling `wait` on them when they are orphaned by their parent process**.