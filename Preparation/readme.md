Lab setup
---

1. our environment will build on the openvswith and kvm/libvirt VMs, create a ovs bridge, then attach the host ethernet interface, create a mgmt interface on the bridge, and assigh ip address to the mgmt interface or using dhcp; 
- /etc/sysconfig/network-scripts/ifcfg-enp24s0
```shell
cat /etc/sysconfig/network-scripts/ifcfg-enp24s0
DEVICE=enp24s0
ONBOOT=yes
DEVICETYPE=ovs
TYPE=OVSPort
OVS_BRIDGE=br-k8s
BOOTPROTO=none
HOTPLUG=no
```

- /etc/sysconfig/network-scripts/ifcfg-br-k8s
```shell
cat /etc/sysconfig/network-scripts/ifcfg-br-k8s
DEVICE=br-k8s
ONBOOT=yes 
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=static
IPADDR=192.168.3.10
NETMASK=255.255.255.0 
GATEWAY=192.168.3.1
DNS1=192.168.3.1
HOTPLUG=no
```


and then check ovs 
```shell
# ovs-vsctl add-br br-k8s
# ovs-vsctl add-port br-k8s enp24s0
# ovs-vsctl show 
6f640d61-54ac-4644-b6b8-3b27db02294b
    Bridge "br-k8s"
        Port "enp24s0"
            Interface "enp24s0"
        Port "br-k8s"
            Interface "br-k8s"
                type: internal
    ovs_version: "2.10.0"
```
- or we can use nmcli to create them
```shell
#!/bin/sh

# create ovs bridge named KubeBridge0
nmcli conn add type ovs-bridge conn.interface br-k8s autoconnect yes

# add add a port to the bridge to encapsulate internal ovs interface (br-k8s)
nmcli conn add type ovs-port conn.interface br-k8s master br-k8s autoconnect yes

# add internal ovs interface to the create br-k8s port
nmcli conn add type ovs-interface conn.interface br-k8s master br-k8s autoconnect yes ipv4.method auto

# add another port to the bridge  to encapsulate host ethernet interface (enp24s0)
nmcli conn add type ovs-port conn.interface enp24s0 master br-k8s autoconnect yes

# attach host ethernet interface to the port
nmcli conn add type ethernet conn.interface enp24s0 master enp24s0 autoconnect yes
```

2. then add more internal interfaces which can be used more VMs
```shell
ovs-vsctl add-port br-k8s vnet0  -- set interface vnet0 type=internal
ovs-vsctl add-port br-k8s vnet1  -- set interface vnet1 type=internal
ovs-vsctl add-port br-k8s vnet2  -- set interface vnet2 type=internal
ovs-vsctl add-port br-k8s vnet3  -- set interface vnet3 type=internal
ovs-vsctl add-port br-k8s vnet4  -- set interface vnet4 type=internal
ovs-vsctl add-port br-k8s vnet5  -- set interface vnet5 type=internal
ovs-vsctl add-port br-k8s vnet6  -- set interface vnet6 type=internal
```

then check the port 
```shell
#ovs-vsctl show
6f640d61-54ac-4644-b6b8-3b27db02294b
    Bridge "br-k8s"
        Port "vnet1"
            Interface "vnet1"
                type: internal
        Port "vnet3"
            Interface "vnet3"
                type: internal
        Port "enp24s0"
            Interface "enp24s0"
        Port "vnet2"
            Interface "vnet2"
                type: internal
        Port "br-k8s"
            Interface "br-k8s"
                type: internal
        Port "vnet6"
            Interface "vnet6"
                type: internal
        Port "vnet0"
            Interface "vnet0"
                type: internal
        Port "vnet4"
            Interface "vnet4"
                type: internal
        Port "vnet5"
            Interface "vnet5"
                type: internal
    ovs_version: "2.10.0"
```
3. Create VMs
```shell 
#virt-install --name=kube-router --disk path=/data/libvirt/vms/kube-router.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet0,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

#virt-install --name=kube-master1 --disk path=/data/libvirt/vms/kube-master1.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet1,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

#virt-install --name=kube-master2 --disk path=/data/libvirt/vms/kube-master2.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet2,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

#virt-install --name=kube-master3 --disk path=/data/libvirt/vms/kube-master3.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet3,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

#virt-install --name=kube-worker1 --disk path=/data/libvirt/vms/kube-worker1.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet4,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

#virt-install --name=kube-worker2 --disk path=/data/libvirt/vms/kube-worker2.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet5,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import

#virt-install --name=kube-worker3 --disk path=/data/libvirt/vms/kube-worker3.qcow2,size=20,format=qcow2 --vcpus 2  --memory=4096 --os-type=linux --os-variant=ubuntu18.04 --network type=direct,source=vnet6,source_mode=bridge --graphics spice  --console pty,target_type=serial   --noautoconsole --import
```

4. configure kube-router
4.1 install quagga
```shell
# apt  install quagga
```
4.2 configure quagga 

```shell
cat /etc/quagga/zebra.conf
! -*- zebra -*-
!
! zebra sample configuration file
!
! $Id: zebra.conf.sample,v 1.1 2002/12/13 20:15:30 paul Exp $
!
hostname kube-router
password zebra
enable password zebra
!
! Interface's description. 
!
!interface lo
! description test of desc.
!
!interface sit0
! multicast

!
! Static default route sample.
!
!ip route 0.0.0.0/0 203.181.89.241
!

!log file zebra.log
```

and 
```shell
cat  /etc/quagga/bgpd.conf 
! -*- bgp -*-"
hostname kube-router
password password                                                                                    
!enable password please-set-at-here                                                                  
!                                                                                                    
!bgp mulitple-instance                                                                               
!                                                                                                    
router bgp 64513                                                                                     
  maximum-paths 4 
  bgp router-id 192.168.3.100
  neighbor 192.168.3.101 remote-as 64512                                                             
  neighbor 192.168.3.102 remote-as 64512                                                             
  neighbor 192.168.3.103 remote-as 64512                                                             
  neighbor 192.168.3.104 remote-as 64512                                                             
  neighbor 192.168.3.105 remote-as 64512                                                             
  neighbor 192.168.3.106 remote-as 64512                                                             
!                                                                                                    
! access-list all permit any                                                                         
!                                                                                                    
!route-map set-nexthop permit 10                                                                     
! match ip address all                                                                               
! set ip next-hop 10.0.0.1                                                                           
!                                                                                                    
!log file /var/log/quagga/bgpd.log                                                                   
!                                                                                                    
log stdout  
```

5. configure kube-idp
- start dex with following conf
```yml
issuer: http://192.168.3.101:5556/dex
storage:
  type: sqlite3
  config:
    file: examples/dex.db
web:
  http: 0.0.0.0:5556

connectors:
- type: ldap
  name: OpenLDAP
  id: ldap
  config:
    host: localhost:389

    # No TLS for this setup.
    insecureNoSSL: true

    # This would normally be a read-only user.
    bindDN: cn=binduser,dc=k8s,dc=org
    bindPW: jennings

    usernamePrompt: Email Address

    userSearch:
      baseDN: ou=People,dc=k8s,dc=org
      filter: "(objectClass=person)"
      username: mail
      # "DN" (case sensitive) is a special attribute name. It indicates that
      # this value should be taken from the entity's DN not an attribute on
      # the entity.
      idAttr: DN
      emailAttr: mail
      nameAttr: cn

    groupSearch:
      baseDN: ou=Groups,dc=k8s,dc=org
      filter: "(objectClass=groupOfNames)"

      # A user is a member of a group when their DN matches
      # the value of a "member" attribute on the group entity.
      userAttr: DN
      groupAttr: member

      # The group name should be the "cn" value.
      nameAttr: cn

staticClients:
- id: k8s
  redirectURIs:
  - 'http://127.0.0.1:5555/callback'
  name: 'k8s'
  secret: ZXhhbXBsZS1hcHAtc2VjcmV0
```
- create slapd.conf, allow user `cn=binduser,dc=k8s,dc=org` can access all objects which can be uesed for binding
    ```conf
    database monitor
    access to *
            by dn.exact="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read
            by dn.exact="cn=Manager,dc=k8s,dc=org" read
            by dn.exact="cn=binduser,dc=k8s,dc=org" read
            by * none

    database      bdb
    suffix        "dc=k8s,dc=org"
    checkpoint    1024 15
    rootdn        "cn=Manager,dc=k8s,dc=org"
    rootpw        {MD5}+UNqsTwwk4dy7iRr1pIHNA==
    ```

- Create domain and add sub-directory, then add multiple users/groups  

    k8s-net.ldif   
    ```ldif
    dn: dc=k8s,dc=org
    objectClass: dcObject
    objectClass: organization
    o: k8s Company
    dc: k8s


    dn: cn=binduser,ou=People,dc=k8s,dc=org
    objectClass: person
    objectClass: inetOrgPerson
    sn: binduser
    cn: binduser
    mail: binduser@k8s.org
    userpassword: jennings


    dn: ou=People,dc=k8s,dc=org
    objectClass: organizationalUnit
    ou: People

    dn: cn=root,ou=People,dc=k8s,dc=org
    objectClass: person
    objectClass: inetOrgPerson
    sn: admin
    cn: root
    mail: root@k8s.org
    userpassword: jennings

    dn: cn=jenningsl,ou=People,dc=k8s,dc=org
    objectClass: person
    objectClass: inetOrgPerson
    sn: liu
    cn: jenningsl
    mail: jenningsl@k8s.org
    userpassword: jennings

    dn: cn=user1,ou=People,dc=k8s,dc=org
    objectClass: person
    objectClass: inetOrgPerson
    sn: wang
    cn: user1
    mail: user1@k8s.org
    userpassword: jennings

    # Group definitions.

    dn: ou=Groups,dc=k8s,dc=org
    objectClass: organizationalUnit
    ou: Groups

    dn: cn=admins,ou=Groups,dc=k8s,dc=org
    objectClass: groupOfNames
    cn: admins
    member: cn=jenningsl,ou=People,dc=k8s,dc=org
    member: cn=root,ou=People,dc=k8s,dc=org

    dn: cn=developers,ou=Groups,dc=k8s,dc=org
    objectClass: groupOfNames
    cn: developers
    member: cn=user1,ou=People,dc=k8s,dc=org
    ```