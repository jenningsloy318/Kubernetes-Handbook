Lab setup
---

1. our environment will build on the openvswith and kvm/libvirt VMs, the host IP is  192.168.3.35/24, host gw 192.168.3.1, all the kubernetes node will reside in 192.168.4.0/24

- Create ovs bridge
```
ovs-vsctl add-br kube-br
```
- add port 
```
ovs-vsctl add-port kube-br enp24s0
ovs-vsctl add-port kube-br vm-net  -- set Interface vm-net  type=internal
```
- Set the IP address for kube-br and vm-net
```
ip addr add 192.168.3.35/24 dev kube-br
ip addr add 192.168.3.4.1/24 dev vm-net 
ip link set vm-net up
```

- Del improper route and add new route the the lan
```
ip route del 192.168.4.0/24  dev vm-net 
ip route add 192.168.4.0/24 via 192.168.4.1
```