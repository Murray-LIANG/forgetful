# What Even Is A Container And A Kubernetes Pod


## Container

The word `container` doesn't mean anything super precise. Basically there are a few new Linux kernel features (`namespaces` and `cgroups`) that let you isolate processes from each other. When you use those features, you call it `containers`.

Basically these features let you pretend you have something like a virtual machine, except it's not a virtual machine at all, it's just processes running in the same Linux kernel.

### Namespaces

`namespaces` is a Linux feature which could separate your processes from the other processes on the same computer.

There are a bunch of different kinds:
- **pid** - you become PID 1 and then your children are other processes. All the other programs are gone.
- **network** - you can run programs on any port you want without it conflicting with what's already running.
- **mount** - you can mount and unmount file systems without it affecting the host filesystem.

#### `unshare` command makes it easy to run your processes inside a namespace.

```console
$ sudo unshare --fork --pid --mount-proc bash
```

List all the processes inside the namespace. You can only see two processes.
```console
root@mint-dev-vm:~/git/microservices-example# ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0  24456  5236 pts/4    S    10:48   0:00 bash
root         14  0.0  0.0  39112  3528 pts/4    R+   10:51   0:00 ps aux
```

Then we run `top` inside the namespace. We can check it's PID from the regular PID namespaces.
```console
➜  ~/git/cinder git:(850013325) ✗ 
$ sudo ps -elf | grep top 
0 S root     114177 114063  0  80   0 - 10856 poll_s 10:50 pts/4    00:00:00 top
```

The PID of `top` is 3 in the new namespace and 114177 in the regular namespace.

#### `nsenter` command allows you enter the namespace of another running program.

```console
➜  ~/git/cinder git:(850013325) ✗ 
$ sudo nsenter -t 114063 -a bash
```

### Cgroups

We now isolate our processes from our old world.

What if I want to limit how much memory or CPU one of my programs is using? `cgroups` could help.

```console
$ # Create a cgroup to just limit memory.
$ sudo cgcreate -a ryan -g memory:mymemgroup

$ ls -l /sys/fs/cgroup/memory/mymemgroup

$ # Limit the quota of memory to 10MB.
$ sudo echo 10000000 > /sys/fs/cgroup/memory/mymemgroup/memory.kmem.limit_in_bytes

$ # Run `bash` under cgroup limitation.
$ sudo cgexec -g memory:mymemgroup bash
```

## Kubernetes Pod

When you run a container with Docker normally, Docker creates namespaces and cgroups for each container so they map one to one.

However, with some extra command line arguments, you can **combine** Docker containers using a single namespace.

```console
$ docker run -d --name ghost \
    --net=container:nginx --ipc=container:nginx --pid=container:nginx ghost
```

In the above example, `ghost` container uses namespaces of `nginx`. **These namespaces allow these two Docker containers to discover and communicate with each other**.

This way you could combine namespaces and cgroups with multiple processes, we can see that's exactly what Kubernetes Pods are. Pods allow you to specify the containers that you want to run and Kubernetes automates setting up the namespaces and cgroups in the right way. So, **Pods are Containers running in same namespaces**.

Once we have our containers set up this way, each process `feels` like it's running on the same machine. They can talk to each other on localhost, they can use shared volumes. They can even use IPC or send each other signals like `HUP` or `TERM`.

With Pods, you can put different services in different containers. Kubernetes manages each container and has insight into its state. That way it can provide services like restarting it when it crashes or automated logging.

Without Pods, you can just put all services into one container to let them feel like running on the same machine and use a `supervisord` to keep all services running. Services might be crashing but Docker would have no idea.


