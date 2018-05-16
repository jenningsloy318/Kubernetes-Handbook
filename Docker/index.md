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

Modify `ExecStart=` to `ExecStart=/usr/bin/dockerd`
and add `Slice=container.slice` in `[Service]` section.

## 3. configure docker parameter via `/etc/docker/daemon.json`
```json
{
    "debug": true,
    "exec-opts": ["native.cgroupdriver=systemd"],
    "cgroup-parent": "container.slice",
    "hosts": ["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"],
    "ip": "0.0.0.0",
    "log-driver": "journald",
    "log-opts":  {"tag":"docker_{{.Name}}_{{.ID}}"},
    "metrics-addr" : "0.0.0.0:9323",
    "experimental" : true,
    "insecure-registries" : [""]
}
```