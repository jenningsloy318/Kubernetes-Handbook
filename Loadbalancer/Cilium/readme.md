## Use cilium externalIPs as VIP to give the applications inside kubernetes external access

### Description: 
|name|value|
|----|-----|
|externalIP CIDR| 10.200.0.0/16|
|LB Node Ip | 10.180.1.16|

To access applications deployed in kubrenetes, we create a  service  with external IP by utilizing cilium `externalIPs` feature, cilium will pick up an IP address from `externalIP CIDR` and  assign it to this service; on the other hand, on LB node, there is a `dummy interface cilium_ext` assigned an IP address from `externalIP CIDR`, we can access the serice external Ip from this interface. 


This LB node must be part of the kubernetes cluster(before cilium can run on pure virtual machine), and if possible add taint to not schedule workload on this node 
### Prepare the LB node

1. install required package on centos 8, newer kernel for cilium to run kube-free mode 

    ```
    dnf install -y conntrack-tools ebtables socat  tc  vim https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
    dnf install -y  --disablerepo="*" --enablerepo="elrepo-kernel" kernel-ml.x86_64 bpftool.x86_64
    dnf install -y  https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm  docker-ce kubeadm
    ```

2. install cilium, enable `externalIPs` feature
    ```
    helm upgrade --install cilium cilium/cilium --version 1.7.2 \
    --namespace kube-system \
    --set global.kubeProxyReplacement=strict \
    --set global.k8sServiceHost=10.180.1.5 \
    --set global.k8sServicePort=6443 \
    --set global.etcd.enabled=false \
    --set externalIPs.enabled=true
    ```
3. create test Nginx 
    - create clusterIP type service with externa IP
    ```
    kubectl apply -f nginx.yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx
    labels:
        app: nginx
    spec:
    externalIPs:
    - 10.200.0.5
    ports:
    - port: 80
        name: cilium-smoke
    selector:
        app: nginx
    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
    name: cilium-smoke
    spec:
    serviceName: "nginx"
    replicas: 1
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx
            ports:
            - containerPort: 80
            name: cilium-smoke
    ```
    - create `LoadBalancer` type service
    ```
    apiVersion: v1
    kind: Service
    metadata:
    name: nginx2
    labels:
        app: nginx2
    spec:
    type: LoadBalancer
    externalIPs:
    - 10.200.0.6
    ports:
    - port: 80
        name: http
    selector:
        app: nginx2
    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
    name: nginx2
    spec:
    serviceName: nginx2
    replicas: 1
    selector:
        matchLabels:
        app: nginx2
    template:
        metadata:
        labels:
            app: nginx2
        spec:
        containers:
        - name: nginx2
            image: nginx
            ports:
            - containerPort: 80
            name: http
    ```
4. check the service
```
# kubectl get svc nginx nginx2
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.96.225.241   10.200.0.5    80/TCP    17h
nginx2  LoadBalancer   10.96.115.232   10.200.0.6    80:31158/TCP   13m

# kubectl exec -it cilium-bz4d8 -- cilium service list 
ID   Frontend             Service Type   Backend
6    10.200.0.5:80        ExternalIPs    1 => 10.100.1.41:80
8    10.200.0.6:80        ExternalIPs    1 => 10.100.1.232:80
```



5. add a route on an test node which can access the LB node

    ```
    ip route add 10.200.0.0/16 via  10.180.1.16
    ```

    then test 
    ```
    # curl  10.200.0.5:80
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```
    
    Note:
    - *The externalIP **can't** be accessed from the kubernetes nodes, as it is intentionally designed to avoid uses to send wrong requests to malicious pods*
6. (optional) configure dummy interface(IP, route)

    *this step is optional, as without this interface, on other host can also access the externalIP via the main interface e.g `eth0`*
    
    *we can add this if we use BCP software e.g bird to anounce the routes(externalIP CIDR) to core switches*

    ```
    sed -i 's/numdummies=0/numdummies=1/g'  /lib/modprobe.d/systemd.conf
    ## udev rule to rename the dummy interface to "cilium_ext"
    echo 'SUBSYSTEM=="net", KERNEL=="dummy0", NAME="cilium_ext"' > /etc/udev/rules.d/75-persistent-dummy-net.rules
    nmcli connection add connection.id cilium_ext connection.autoconnect true connection.type dummy connection.interface-name  cilium_ext  ipv4.method manual ipv4.address 10.200.0.1/24 +ipv4.routes "10.200.0.0/24 10.200.0.1"

    ```
    create scripts to bring up  connection `cilium_ext` at boot 
    ```
    cat /etc/NetworkManager/dispatcher.d/start_cilium_ext
    #!/bin/bash
    /bin/nmcli connection up  cilium_ext
    chmod +x /etc/NetworkManager/dispatcher.d/start_cilium_ext
    ```
    show the connection 

    ```
    # nmcli conn show cilium_ext
    connection.id:                          cilium_ext
    connection.uuid:                        8ede8ce8-1206-4b71-9214-8f7b0b2c14d0
    connection.stable-id:                   --
    connection.type:                        dummy
    connection.interface-name:              cilium_ext
    connection.autoconnect:                 yes
    connection.autoconnect-priority:        0
    connection.autoconnect-retries:         -1 (default)
    connection.multi-connect:               0 (default)
    connection.auth-retries:                -1
    connection.timestamp:                   1587522924
    connection.read-only:                   no
    connection.permissions:                 --
    connection.zone:                        --
    connection.master:                      --
    connection.slave-type:                  --
    connection.autoconnect-slaves:          -1 (default)
    connection.secondaries:                 --
    connection.gateway-ping-timeout:        0
    connection.metered:                     unknown
    connection.lldp:                        default
    connection.mdns:                        -1 (default)
    connection.llmnr:                       -1 (default)
    connection.wait-device-timeout:         -1
    ipv4.method:                            manual
    ipv4.dns:                               --
    ipv4.dns-search:                        --
    ipv4.dns-options:                       --
    ipv4.dns-priority:                      0
    ipv4.addresses:                         10.200.0.1/24
    ipv4.gateway:                           --
    ipv4.routes:                            { ip = 10.200.0.0/16, nh = 10.200.0.1 }
    ipv4.route-metric:                      -1
    ipv4.route-table:                       0 (unspec)
    ipv4.routing-rules:                     --
    ipv4.ignore-auto-routes:                no
    ipv4.ignore-auto-dns:                   no
    ipv4.dhcp-client-id:                    --
    ipv4.dhcp-timeout:                      0 (default)
    ipv4.dhcp-send-hostname:                yes
    ipv4.dhcp-hostname:                     --
    ipv4.dhcp-fqdn:                         --
    ipv4.never-default:                     no
    ipv4.may-fail:                          yes
    ipv4.dad-timeout:                       -1 (default)
    ipv6.method:                            auto
    ipv6.dns:                               --
    ipv6.dns-search:                        --
    ipv6.dns-options:                       --
    ipv6.dns-priority:                      0
    ipv6.addresses:                         --
    ipv6.gateway:                           --
    ipv6.routes:                            --
    ipv6.route-metric:                      -1
    ipv6.route-table:                       0 (unspec)
    ipv6.routing-rules:                     --
    ipv6.ignore-auto-routes:                no
    ipv6.ignore-auto-dns:                   no
    ipv6.never-default:                     no
    ipv6.may-fail:                          yes
    ipv6.ip6-privacy:                       -1 (unknown)
    ipv6.addr-gen-mode:                     stable-privacy
    ipv6.dhcp-duid:                         --
    ipv6.dhcp-send-hostname:                yes
    ipv6.dhcp-hostname:                     --
    ipv6.token:                             --
    proxy.method:                           none
    proxy.browser-only:                     no
    proxy.pac-url:                          --
    proxy.pac-script:                       --
    ```

7.(optional) we can also install `bird` or other router softeware to anounce BGP routes about this `external CIDR` to the core switches or routes, so we don't need to add static routes manually


---
## cilium  1.10 has XDP-based Standalone Load Balancer(1) and with help of BGP(metalLB)(2), it provies VIP access from outside



