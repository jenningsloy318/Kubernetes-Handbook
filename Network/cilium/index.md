    
- install cilium via helm
    ```
    helm repo add cilium https://helm.cilium.io/

    helm install cilium cilium/cilium --version 1.7.0 \
        --namespace kube-system \
        --set global.kubeProxyReplacement=strict \
        --set global.k8sServiceHost=192.168.3.101\
        --set global.k8sServicePort=6443 \
        --set global.etcd.enabled=true \
        --set global.etcd.endpoints[0]=http://192.168.3.101:2379 \
        --set global.cni.chainingMode=portmap \
        --set global.prometheus.enabled=true 

    # patch cilium operator tolerations to make it run on master node
    kubectl -n kube-system patch deployment cilium-operator --patch "$(cat patch-cilium-tolerations.yaml)"
    ```