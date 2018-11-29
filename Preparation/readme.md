Lab setup
---

1. our environment will build on the openvswith and kvm/libvirt VMs, create a ovs bridge, then attach the host ethernet interface, create a mgmt interface on the bridge, and assigh ip address to the mgmt interface or using dhcp; as new version of NetworkManger can support ovs bridge/port/interface, we can use nmcli to create these items.
```shell
#!/bin/sh

# create ovs bridge named KubeBridge0
nmcli conn add type ovs-bridge conn.interface KubeBridge0 autoconnect yes

# add add a port to the bridge to encapsulate internal ovs interface (MgmtIface0)
nmcli conn add type ovs-port conn.interface MgmtPort0 master KubeBridge0 autoconnect yes

# add internal ovs interface to the create MgmtPort0 port 
nmcli conn add type ovs-interface conn.interface MgmtIface0 master MgmtPort0 autoconnect yes ipv4.method auto

# add another port to the bridge  to encapsulate host ethernet interface (enp24s0)
nmcli conn add type ovs-port conn.interface enp24s0Port0 master KubeBridge0 autoconnect yes

# attach host ethernet interface to the port
nmcli conn add type ethernet conn.interface enp24s0 master enp24s0Port0 autoconnect yes
```


2. then add more internal interfaces which can be used more VMs
```shell
ovs-vsctl add-port KubeBridge0 vnet0  -- set Interface vnet0  type=internal
ovs-vsctl add-port KubeBridge0 vnet1  -- set Interface vnet1  type=internal
ovs-vsctl add-port KubeBridge0 vnet2  -- set Interface vnet2  type=internal
ovs-vsctl add-port KubeBridge0 vnet3  -- set Interface vnet3  type=internal
ovs-vsctl add-port KubeBridge0 vnet4  -- set Interface vnet4  type=internal
ovs-vsctl add-port KubeBridge0 vnet5  -- set Interface vnet5  type=internal
ovs-vsctl add-port KubeBridge0 vnet6  -- set Interface vnet6  type=internal

```

3. Create VMs
```shell 
virt-install --name=kube-router --disk path=/data/libvirt/vms/kube-router.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet0,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

virt-install --name=kube-master1 --disk path=/data/libvirt/vms/kube-master1.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet1,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

virt-install --name=kube-master2 --disk path=/data/libvirt/vms/kube-master2.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet2,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

virt-install --name=kube-master3 --disk path=/data/libvirt/vms/kube-master3.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet3,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

virt-install --name=kube-worker1 --disk path=/data/libvirt/vms/kube-worker1.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet4,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

virt-install --name=kube-worker2 --disk path=/data/libvirt/vms/kube-worker2.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet5,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

virt-install --name=kube-worker3 --disk path=/data/libvirt/vms/kube-worker3.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet6,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import
```

