# Introduction
  This book is a step-by-step handbook to build a high available kubernetes cluster. Although there are many tools that can help us to create kubernetes cluster, but here I still use create this cluster manually. from this way, I will have a better understading of the cluster components and other addons and plugins. 


## Cluster design

|Node|IP address | role | version|
|----|-----------|-----------|--------|
|kube-master1| 192.168.59.130|master| 1.10|
|kube-master2| 192.168.59.130|master| 1.10|
|kube-master3| 192.168.59.130|master| 1.10|
|kube-node1| 192.168.59.130|work-node| 1.10|


>Four VMs are created to build this cluster, using vmware player 14 player to achieve this. 

Other components

|component|version|
|---------|-------|
|Docker CE |17.03.2|
