1. adjust the component version in [istioctl-cilium-values.yaml](istioctl-cilium-values.yaml)
2. install/upgrade istioctl
```sh
#!/bin/bash

export ISTIO_VERSION=1.5.2


echo -e "\e[1;31downloading cilium modified istioctl  \e[0m\n"
curl -LS https://github.com/cilium/istio/releases/download/$ISTIO_VERSION/cilium-istioctl-$ISTIO_VERSION-linux.tar.gz | tar xz

./cilium-istioctl   manifest apply --verbose  -f istioctl-cilium-values.yaml 

or 
./cilium-istioctl   upgrade --verbose  -f istioctl-cilium-values.yaml 
```

3. as we disabled auto-inject, just label namespace which we need to inject
```
kubectl label namespace jaas istio-injection=enabled
```

4. then create/restart pod to take effect