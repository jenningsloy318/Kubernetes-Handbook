Currently many storage drivers are in-tree code within the core kubernetes, to decouple different storage with kubernetes, future kubernetes will gradually use CSI for the storage.

## CSI 
1. CSI volume driver: it will be running on the nodes, and create is a unix domian socket for communicating with kubelet;  
    - running as container within same pod with `quay.io/k8scsi/driver-registrar:xxx` of deemonset 
2. kubelet will communicate with `CSI volume driver` with this socket;kubelet must call the CSI method NodeGetInfo to get the mapping from Kubernetes Node names to CSI driver NodeID and the associated accessible_topology
    - Create/update a CSINodeInfo object instance for the node with the NodeID and topology keys from accessible_topology
    - Update Node API object with the CSI driver NodeID as the csi.volume.kubernetes.io/nodeid annotation. The value of the annotation is a JSON blob, containing key/value pairs for each CSI driver
    - Create/update Node API object with accessible_topology as labels
    - one container `quay.io/k8scsi/driver-registrar:xxx` to register the `CSI volume driver` in the same pod of the daemonset to kubernetes
3. Provisioner:  watch the Kubernetes API on behalf of the external CSI volume driver to handle Provisioning and deletion requests. 
4. Attacher: watch the Kubernetes API on behalf of the external CSI volume driver to handle attach/detach requests.
    > provisioner and attacher must be running in a statfulset with only 1 repica; to commumicate with `CSI volume driver`, just mount the `CSI volume driver` socket driver directory into them
5. In-Tree CSI Volume Plugin: Contain all the logic required for Kubernetes to communicate with an arbitrary, out-of-tree, third-party CSI compatible volume driver.
    
[Recommended Mechanism for Deploying CSI Drivers on Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md#recommended-mechanism-for-deploying-csi-drivers-on-kubernetes)


