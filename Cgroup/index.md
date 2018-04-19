# 1. Cgroup Basis


Cgroup which is `control group`  is a Linux kernel feature, which is used to restrict the resource consuming for process and tasks.


Common controllers:

- blkio — this subsystem sets limits on input/output access to and - from block devices such as physical drives (disk, solid state, or - USB).
- cpu — this subsystem uses the scheduler to provide cgroup tasks - access to the CPU.
- cpuacct — this subsystem generates automatic reports on CPU - resources used by tasks in a cgroup.
- cpuset — this subsystem assigns individual CPUs (on a multicore - system) and memory nodes to tasks in a cgroup.
- devices — this subsystem allows or denies access to devices by tasks - in a cgroup.
- freezer — this subsystem suspends or resumes tasks in a cgroup.
- memory — this subsystem sets limits on memory use by tasks in a - cgroup and generates automatic reports on memory resources used by - those tasks.
- net_cls — this subsystem tags network packets with a class - identifier (classid) that allows the Linux traffic controller (tc) - to identify packets originating from a particular cgroup task.
- net_prio — this subsystem provides a way to dynamically set the - priority of network traffic per network interface.
- ns — the namespace subsystem.
- perf_event — this subsystem identifies cgroup membership of tasks and can be used for performance analysis. 
- hugetlb — allows to use virtual memory pages of large sizes and to enforce resource limits on these pages. 


## Cgroup V1

In cgroup v1, each  controller has its hierarchy, if we restrict a process to muliple resources, this process will be in different resource hierarchy   

## Cgroup V2
Unlike v1, cgroup v2 has only single hierarchy, we can find details of cgroup v2 on page [Cgroup V2](https://www.kernel.org/doc/Documentation/cgroup-v2.txt).



# Content

This chapter will discuss how to configure cgroup and how to use them in kubernetes components
  - [1.1 Cgroup configuration](Config.md)
  - [1.2 Cgroups/Slices for kuberners](Cgroups.md)
