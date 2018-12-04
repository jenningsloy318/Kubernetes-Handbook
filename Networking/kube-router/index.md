Kube-router Usage cases
---
- we don't use kube-router to build pod-to-pod network or service porxy(equivilent of kube-proxy)
- kube-router can peer with existing BGP routers outside the cluster so that pod IP will be routable from outside the cluster without creating corresponding service
- Kube-router can advertise cluster IP(service IP) and service's external IP’s to configured BGP peers, so that services can be exposed easily outside the cluster.
