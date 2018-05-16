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

/lib/systemd/system/container.slice
```
[Unit]
Description=Resource limits for  containers
DefaultDependencies=no
Before=slices.target
Requires=-.slice
After=-.slice
[Slice]
MemoryLimit=60G
```
## Set the system.slice resource 

```
[Unit]
Description=System Slice
Documentation=man:systemd.special(7)
DefaultDependencies=no
Before=slices.target
[Slice]
MemoryLimit=4G
```

> Here we only set the Memory limit, we can add any kinds of limits here

> Here we create slice with cgroup v1, the parameter is different if we use cgroup v2 

> details of cgroup v1/v2 configuraton in systemd, we can find it on page [systemd.resource-control](https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html).

> in V1 model, resource restriction only have upper limit, for example `MemoryLimit=`.

> in V2 model, resource restriction have both upper and lower limit, for example `MemoryMax=` and `MemoryLow=`

> untill now, docker engine still don't support cgroup v2 in ubuntu 18.04 though ubuntu 18.04 can boot with only cgroup v2.