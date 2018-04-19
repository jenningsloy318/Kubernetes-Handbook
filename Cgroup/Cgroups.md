# 1.2 Cgroups for different components

On systems that ship with systemd, we can create `.slice` file and bond to corresponding `.service` file.

Restrict the resorce consuming of kubernetes components make it not eat all the resources, hence  the   basic services will still be running, thus the system don't crash due OOMKill or CPU exhausted.

We will create at least 2 slices:
- runtime.slice: used for all kubelet and docker engine service 
- container.slice: used for all pods/containers

## Create runtime.slice

/lib/systemd/system/runtime.slice
```
[Unit]
Description=Resource limits for kubelet and docker engine
DefaultDependencies=no
Before=slices.target
Requires=-.slice
After=-.slice
[Slice]
MemoryLimit=8G
```

## Create container.slice

/lib/systemd/system/runtime.slice
```
[Unit]
Description=Resource limits for kubelet and docker engine
DefaultDependencies=no
Before=slices.target
Requires=-.slice
After=-.slice
[Slice]
MemoryLimit=60G
```

> Here we only set the Memory limit, we can add any kind of limits here

> Here we create slice with cgroup v1, the parameter is different if we use cgroup v2 

> details of cgroup v1/v2 configuraton in systemd, we can find it on page [systemd.resource-control](https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html).