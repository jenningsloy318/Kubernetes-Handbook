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
