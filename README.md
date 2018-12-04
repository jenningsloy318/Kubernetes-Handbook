# Introduction
  This book is a step-by-step handbook to build a high available kubernetes cluster. Although there are many tools that can help us to create kubernetes cluster, but here I still use create this cluster manually. from this way, I will have a better understading of the cluster components and other addons and plugins. This test k8s cluster will simulate the scenario that integrate a existing network with k8s cluster, make pods are accessible to formally created servers.


# Cluster Design

## 1.  Network diagram


 
|Node|IP address(k8s) | role| version|
|------------|----------------|------|-----|
|kube-router| 192.168.3.100|Quagga| x |
|kube-master1| 192.168.3.101|master| 1.13|
|kube-master2| 192.168.3.102|master| 1.13|
|kube-master3| 192.168.3.103|master| 1.13|
|kube-worker1| 192.168.3.104|work-node| 1.13|
|kube-worker2| 192.168.3.105|work-node| 1.13|
|kube-worker3| 192.168.3.106|work-node| 1.13|



## 2. K8S POD/SVC Ntworks

- Pod: 10.30.0.0/16
- service: 10.40.0.0/16
- loadbalancer: 10.50.0.0/16
- service external: 10.50.0.0/16
- DNS Service IP: 10.40.0.10
- k8s cluster DNS domain: k8s.org
- external  DNS domain: k8s.net
- Kubernetes API VIP: 192.168.3.110


## 3. Other components

|component|version|
|---------|-------|
|Docker CE |18.06|
|etcd| 3.2.24|
|cilium|1.3.0|
|calico|3.3.1|
|kube-router|0.2.3|
|metallb|v0.7.3|
|CoreDNS|v1.2.6|

## 4. OS 

ubuntu 18.04 

## 5. BGP router
Quagga:  
- separate Linux host running as a router
- can work with metallb to assign the loadbalancer IP for services
- also can work with kube-router to expose POD/SVC IP
- also can work with kube-router to  add external IP for service

## 6. External DNS
- PowerDNS
- CoreDNS
- Use [external-dns](https://github.com/kubernetes-incubator/external-dns) to synchronize the service and ingress in k8s cluster to external PowerDNS/CoreDNS, thus external can access the serive from outside


## 7. kubelet TLS bootstrap
- https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/
- https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/

## 8. User Management 
- OpenLDAP 
- [Dex](https://github.com/dexidp/dex)
- apiserver configure odic 
- apiserver enable RBAC


# Cluster Deployment

1. All kubernetes master components can be deployed etheir in binary or pod mode
2. Master nodes disable the workload
3. Use IPVS for service proxy(kube-router or kube-proxy)
4. Use NFS as the storage backend  
