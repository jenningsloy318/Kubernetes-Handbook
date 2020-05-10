# Docker engine

Download URL: https://download.docker.com/linux/

The docker has several components integrated together to serve as an engine. 
![docker componenent](./images/docker-components.png)
#### Note: now docker engine don't support cgroup v2, so we can't disable cgroup v1.

## 1. Instllation
download the docker-ce package and install via dpkg command
```
dpkg -i <docker-ce deb package>
```

## 2. configure docker.service

Modify `ExecStart=` to `ExecStart=/usr/bin/dockerd`, since all configurations are moved to `/etc/docker/daemon.json`
and add `Slice=container.slice` in `[Service]` section.

example docker.service file can be found [here](./docker.srvice)


modify `MountFlags=shared` to adhere to the `CSI` to let kubernetes use CSI storages.

## 3. configure docker parameter via `/etc/docker/daemon.json`
```json
{
    "debug": true,
    "exec-opts": ["native.cgroupdriver=systemd"],
    "hosts": ["tcp://0.0.0.0:2376","unix:///var/run/docker.sock"],
    "ip": "0.0.0.0",
    "metrics-addr" : "0.0.0.0:9323",
    "experimental" : true,
    "insecure-registries" : [""],
    "storage-driver": "overlay2"

}
```
## docker registry proxy 
- [Config](./config.yml) file for docker registry
    ```yaml
    version: 0.1
    log:
      fields:
        service: registry
    storage:
      inmemory:
        delete:
          enabled: false
        cache:
          blobdescriptor: inmemory
    http:
      addr: :5000
      headers:
        X-Content-Type-Options: [nosniff]
    health:
      storagedriver:
        enabled: true
        interval: 10s
        threshold: 3
    proxy:
      remoteurl: https://registry.aliyuncs.com
    ```
- run the registry proxy
    ```sh
    # sudo docker run -d --restart=always -p 443:443  --name docker-registry-proxy -v `pwd`/config.yml:/etc/docker/registry/config.yml  -v "$(pwd)"/registry-certs:/certs  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker.pem -e REGISTRY_HTTP_TLS_KEY=/certs/docker.key  docker.io/library/registry
    ```

> Note: 
> - if we need to proxy multiple registries, should setup one proxy for each registry