# 1.1 Configure Cgroup

## Enable conrollers 

Edit `/etc/systemd/system.conf` to enable necessary controllers, add or modify `JoinControllers` parameter. 

```
JoinControllers=cpu,cpuacct,cpuset,net_cls,net_prio,hugetlb,memory,systemd
```

> All controllers can be enabled, but at least controolers that in kubernet source code  [cgroup_manager_linux.go](ttps://github.com/kubernetes/kubernetes/blob/release-1.10/pkg/kubelet/cm/cgroup_manager_linux.go) must be enabled. We can get cotrollers from this file, jist find this line`whitelistControllers := sets.NewString("cpu", "cpuacct", "cpuset", "memory", "systemd")`

## Enable Accounting

Another  parameters in `/etc/systemd/system.conf` that should be set:
 - DefaultCPUAccounting=true 
 - DefaultMemoryAccounting=true

This enables the CPU/MEM limits work when starting a container/pod

