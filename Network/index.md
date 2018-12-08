# Networking

All network plugins are list on https://kubernetes.io/docs/concepts/cluster-administration/networking.

When we are about to create a kubernetes cluster, one thing need to consider is if we need to integrated with existing infrastructure in the environment, that is if pods can be accessible by existing network directly.

## Network plugins flavors
1. [CNI](https://github.com/containernetworking/cni) adhere to the appc/CNI specification, designed for interoperability.
- Each cni network plugin will support following actions:
    - ADD: Add container to network
    - DEL: Delete container from network
    - CHECK: Check container's networking is as expected
    - VERSION: Report version
- CNI support following two extra features:
    - support `hostPort` via official `portmap` plugin offered by the CNI plugin team;
    - supports pod ingress and egress traffic shaping via official `bandwidth` plugin offered by the CNI plugin team;

2. kubenet: implements basic cbr0 using the bridge and host-local CNI plugins
Kubenet is a very basic, simple network plugin, on Linux only. It does not, of itself, implement more advanced features like cross-node networking or network policy. It is typically used together with a cloud provider that sets up routing rules for communication between nodes, or in single-node environments
## Network Type

1. layer 2 (switching)
    - vlan

2. layer 3 (routing) solution
    - BGP (Calico,Contiv)

3. Overlay solutions
    - VXLAN (flannel,cilium,ovs)

4. Underlay solutions:
    - MACVLAN
    - IPVLAN


## Network Deployment

Three network plugins are chosed in this deployment:
1. [cilium](./cilium/index.md)
2. [calico](./calico/index.md)
3. [kube-router](./kube-router/index.md)

## Network policy

Network policy is to enforce security to the cluster, so we can add it to the cluster, previous listed plugin also support to enforce the network policy to different kinds of objects in kuberntes cluster.