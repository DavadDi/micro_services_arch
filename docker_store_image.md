# Docker Image Storage

## 1. Docker 基本信息

安装Docker后目录结构, 采用overlay driver

17.03.1-ce 安装需要Centos7.3以上版本。

安装Docker后目录结构, 采用overlay driver

```
# cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
```

```
# docker version
Client:
 Version:      17.03.1-ce
 API version:  1.27
 Go version:   go1.7.5
 Git commit:   c6d412e
 Built:        Mon Mar 27 17:05:44 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.1-ce
 API version:  1.27 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   c6d412e
 Built:        Mon Mar 27 17:05:44 2017
 OS/Arch:      linux/amd64
 Experimental: false

```


```
# docker info

Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 1
Server Version: 17.03.1-ce
 Storage Driver: overlay    ------- overlay (overlay2 kenerl >= 4.0)
 ------------------------
 Backing Filesystem: xfs
 Supports d_type: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 4ab9917febca54791c5f071a9d1f404867857fcc
runc version: 54296cf40ad8143b62dbcaa1d90e520a2136ddfe
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-514.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 976.5 MiB
Name: localhost.localdomain
ID: 6OUR:TNDP:NVJK:M2PM:GPC3:WGTR:EWVB:2GQT:2HHJ:3WYJ:E6HV:L5EG
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false

```

## 2. Overlay

![overlay](https://docs.docker.com/engine/userguide/storagedriver/images/overlay_constructs.jpg) 

因为 overlay driver 工作在单个层上，因此
multi-layered 的实现需要采用hard link，在overlay2中已经natively支持multiple lower OverlayFS layers，最大到128层，因此在某些命令上效率更高。

> While the overlay driver only works with a single lower OverlayFS layer and hence requires hard links for implementation of multi-layered images, the overlay2 driver natively supports multiple lower OverlayFS layers (up to 128).
> 
> Hence the overlay2 driver offers better performance for layer-related docker commands (e.g. docker build and docker commit), and consumes fewer inodes than the overlay driver.
> 
> <b>Inode limits</b>. Use of the overlay storage driver can cause excessive inode consumption. This is especially so as the number of images and containers on the Docker host grows. A Docker host with a large number of images and lots of started and stopped containers can quickly run out of inodes. The overlay2 does not have such an issue.

Image From [Docker and OverlayFS in practice](https://docs.docker.com/engine/userguide/storagedriver/overlayfs-driver/#image-layering-and-sharing-with-overlayfs-overlay)

```
# mount | grep overlay
/dev/sda3 on /var/lib/docker/overlay type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

Link 

* [Select a storage driver](https://docs.docker.com/engine/userguide/storagedriver/selectadriver/)
* [image/spec/v1.2.md](https://github.com/docker/docker/blob/master/image/spec/v1.2.md)
* [The new stored format of Docker image on disk and Distribution](http://hustcat.github.io/docker-image-new-format/)

```
tree /var/lib/docker/
/var/lib/docker/
|-- containers
|-- image
|   `-- overlay
|       |-- distribution
|       |-- imagedb
|       |   |-- content
|       |   |   `-- sha256
|       |   `-- metadata
|       |       `-- sha256
|       |-- layerdb
|       `-- repositories.json
|-- network
|   `-- files
|       `-- local-kv.db
|-- overlay
|-- plugins
|   |-- storage
|   |   `-- blobs
|   |       `-- tmp
|   `-- tmp
|-- swarm
|-- tmp
|-- trust
`-- volumes
    `-- metadata.db
```

## 3. Docker Image
### 3.1 Pull busybox Image

```
docker pull busybox
Using default tag: latest
Trying to pull repository docker.io/library/busybox ...
latest: Pulling from docker.io/library/busybox
7520415ce762: Pull complete
Digest: sha256:32f093055929dbc23dec4d03e09dfe971f5973a9ca5cf059cbfb644c206aa83f
Status: Downloaded newer image for docker.io/busybox:latest
```

### 3.2 Pull Image后文件结构

省略了与第一次tree相同的内容

```
|-- containers
tree /var/lib/docker/
/var/lib/docker/
|-- containers
|-- image
|   `-- overlay
|       |-- distribution
|       |   |-- diffid-by-digest   hashes of compressed data
|       |   |   `-- sha256 [7520415ce762: Pull complete] 见 docker pull busybox
|       |   |       `-- 7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3
|       |   |           sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9
|       |   `-- v2metadata-by-diffid
|       |       `-- sha256  ③  hashes of uncompressed data
|       |           `-- c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9
|       |      [{"Digest":"sha256:7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3", --> diffid-by-digest
|       |                 "SourceRepository":"docker.io/library/busybox","HMAC":""}]
|       |-- imagedb
|       |   |-- content
|       |   |   `-- sha256   [IMAGE ID] ①  文件包含了 image json数据，详见下文 SHA256 hash of its configuration JSON
|       |   |      、      该文件名称为内容的 sha256sum 值，可以通过linux   sha256sum 命令进行验证
|       |   |       `-- 00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff
|       |   `-- metadata
|       |       `-- sha256
|       |-- layerdb
|       |   |-- sha256   [Layer ID --> v2metadata-by-diffid]
|       |   |   `-- c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9 ④ [ChainID] 只有一层等于 Layer DiffID 否则 ChainID(layerN) = SHA256hex(ChainID(layerN-1) + " " + DiffID(layerN))
|       |   |       |-- cache-id ⑤ [ccb47fc4077d37cb1c22c1db317b014347807d3cb5d41e2437a623788b438f5e]
|       |   |       |-- diff [sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9]
|       |   |       |-- size [1109996]
|       |   |       `-- tar-split.json.gz Layers must be packed and unpacked reproducibly to avoid changing the layer ID, for example by using tar-split to save the tar headers. 
|       |   `-- tmp
|       `-- repositories.json
|-- overlay  [保存真正数据的目录]
|   `-- ccb47fc4077d37cb1c22c1db317b014347807d3cb5d41e2437a623788b438f5e ⑥
|       `-- root  ⑦ 真正的文件
|           |-- bin
|           |   |-- arp
|           |   ............
|           |
|           |-- etc
|           |   |-- group
|           |   |-- localtime
|           |   |-- passwd
|           |   `-- shadow
|           |-- home
|           |-- root
|           |-- tmp
|           |-- usr
|           |   `-- sbin
|           `-- var
|               |-- spool
|               |   `-- mail
|               `-- www
```


```
docker images --no-trunc
REPOSITORY          TAG                 IMAGE ID                                                                  CREATED             SIZE
busybox             latest              sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff   4 weeks ago         1.11 MB
```

```
docker rmi docker.io/busybox:latest
Untagged: docker.io/busybox:latest
Deleted: sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff
Deleted: sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9
```

```
docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox
Digest: sha256:32f093055929dbc23dec4d03e09dfe971f5973a9ca5cf059cbfb644c206aa83f [tag digest]--> repositories.json
Status: Image is up to date for busybox:latest
```

```
# cat repositories.json | python -m json.tool
{
    "Repositories": {
        "busybox": {
            "busybox:latest": "sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff",
            "busybox@sha256:32f093055929dbc23dec4d03e09dfe971f5973a9ca5cf059cbfb644c206aa83f": "sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff"
        }
    }
}

```


```
# cat image/overlay/imagedb/content/sha256/00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff | python -m json.tool
{
    "architecture": "amd64",
    "config": {
        "AttachStderr": false,
        "AttachStdin": false,
        "AttachStdout": false,
        "Cmd": [
            "sh"
        ],
        "Domainname": "",
        "Entrypoint": null,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Hostname": "1295ff10ed92",
        "Image": "sha256:0d7e86beb406ca2ff3418fa5db5e25dd6f60fe7265d68a9a141a2aed005b1ae7",
        "Labels": {},
        "OnBuild": null,
        "OpenStdin": false,
        "StdinOnce": false,
        "Tty": false,
        "User": "",
        "Volumes": null,
        "WorkingDir": ""
    },
    "container": "d12e9fb4928df60ac71b4b47d56b9b6aec383cccceb3b9275029959403ab4f73",
    "container_config": {
        "AttachStderr": false,
        "AttachStdin": false,
        "AttachStdout": false,
        "Cmd": [
            "/bin/sh",
            "-c",
            "#(nop) ",
            "CMD [\"sh\"]"
        ],
        "Domainname": "",
        "Entrypoint": null,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Hostname": "1295ff10ed92",
        "Image": "sha256:0d7e86beb406ca2ff3418fa5db5e25dd6f60fe7265d68a9a141a2aed005b1ae7",
        "Labels": {},
        "OnBuild": null,
        "OpenStdin": false,
        "StdinOnce": false,
        "Tty": false,
        "User": "",
        "Volumes": null,
        "WorkingDir": ""
    },
    "created": "2017-03-09T18:28:04.586987216Z",
    "docker_version": "1.12.6",
    "history": [
        {
            "created": "2017-03-09T18:28:03.975884948Z",
            "created_by": "/bin/sh -c #(nop) ADD file:c9ecd8ff00c653fb652ad5a0a9215e1f467f0cd9933653b8a2e5e475b68597ab in / "
        },
        {
            "created": "2017-03-09T18:28:04.586987216Z",
            "created_by": "/bin/sh -c #(nop)  CMD [\"sh\"]",
            "empty_layer": true
        }
    ],
    "os": "linux",
    "rootfs": {
        "diff_ids": [ ② --> v2metadata-by-diffid/sha256/xxxxx
            "sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9"
        ],
        "type": "layers"
    }
}
```

see : 

* [Docker 简析 1 - 镜像与容器](http://ryerh.com/2017/03/17/docker-inside-section-1-image-and-container.html)
* [docker pull命令实现与镜像存储（3）](http://blog.csdn.net/idwtwt/article/details/53493745)
* [10张图带你深入理解Docker容器和镜像](http://dockone.io/article/783) [english](http://merrigrove.blogspot.jp/2015/10/visualizing-docker-containers-and-images.html)


## 4. Docker Container

### 4.1 Create Container

```
# docker create -it  --name busybox busybox
6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d
```

文件变化情况如下：

```
tree
|-- containers
|   `-- 6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d
|       |-- checkpoints [empty]
|       |-- config.v2.json
|       `-- hostconfig.json
|-- image
|   `-- overlay
|       |-- layerdb
|       |   |-- mounts
|       |   |   `-- 6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d
|       |   |       |-- init-id [acb73196bd7d563a3a167b61e564a6597d63dad559c0731343d030d5707f06a5-init]
|       |   |       |-- mount-id [acb73196bd7d563a3a167b61e564a6597d63dad559c0731343d030d5707f06a5]
|       |   |       `-- parent [sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9]
|-- overlay
|   |-- acb73196bd7d563a3a167b61e564a6597d63dad559c0731343d030d5707f06a5
|   |   |-- lower-id [ccb47fc4077d37cb1c22c1db317b014347807d3cb5d41e2437a623788b438f5e]
|   |   |-- merged [empty]
|   |   |-- upper
|   |   |   |-- dev
|   |   |   |   |-- console
|   |   |   |   |-- pts
|   |   |   |   `-- shm
|   |   |   |-- etc
|   |   |   |   |-- hostname [empty]
|   |   |   |   |-- hosts [empty]
|   |   |   |   |-- mtab -> /proc/mounts
|   |   |   |   `-- resolv.conf [empty]
|   |   |   |-- proc
|   |   |   `-- sys
|   |   `-- work
|   |       `-- work
|   |-- acb73196bd7d563a3a167b61e564a6597d63dad559c0731343d030d5707f06a5-init
|   |   |-- lower-id [ccb47fc4077d37cb1c22c1db317b014347807d3cb5d41e2437a623788b438f5e]
|   |   |-- merged
|   |   |-- upper
|   |   |   |-- dev
|   |   |   |   |-- console
|   |   |   |   |-- pts
|   |   |   |   `-- shm
|   |   |   |-- etc
|   |   |   |   |-- hostname [empty]
|   |   |   |   |-- hosts [empty]
|   |   |   |   |-- mtab -> /proc/mounts
|   |   |   |   `-- resolv.conf [empty]
|   |   |   |-- proc
|   |   |   `-- sys
|   |   `-- work
|   |       `-- work


70 directories, 406 files
```


```
# cat containers/6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d/config.v2.json | python -m json.tool
{
    "AppArmorProfile": "",
    "Args": [],
    "Config": {
        "AttachStderr": true,
        "AttachStdin": true,
        "AttachStdout": true,
        "Cmd": [
            "sh"
        ],
        "Domainname": "",
        "Entrypoint": null,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Hostname": "6b89258c98ed",
        "Image": "busybox",
        "Labels": {},
        "OnBuild": null,
        "OpenStdin": true,
        "StdinOnce": true,
        "Tty": true,
        "User": "",
        "Volumes": null,
        "WorkingDir": ""
    },
    "Created": "2017-04-08T11:50:37.794940171Z",
    "Driver": "overlay",
    "HasBeenManuallyStopped": false,
    "HasBeenStartedBefore": false,
    "HostnamePath": "",
    "HostsPath": "",
    "ID": "6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d",
    "Image": "sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff",
    "LogPath": "",
    "Managed": false,
    "MountLabel": "",
    "MountPoints": {},
    "Name": "/busybox",
    "NetworkSettings": {
        "Bridge": "",
        "HairpinMode": false,
        "HasSwarmEndpoint": false,
        "IsAnonymousEndpoint": false,
        "LinkLocalIPv6Address": "",
        "LinkLocalIPv6PrefixLen": 0,
        "Networks": {
            "bridge": {
                "Aliases": null,
                "EndpointID": "",
                "Gateway": "",
                "GlobalIPv6Address": "",
                "GlobalIPv6PrefixLen": 0,
                "IPAMConfig": null,
                "IPAMOperational": false,
                "IPAddress": "",
                "IPPrefixLen": 0,
                "IPv6Gateway": "",
                "Links": null,
                "MacAddress": "",
                "NetworkID": ""
            }
        },
        "Ports": null,
        "SandboxID": "",
        "SandboxKey": "",
        "SecondaryIPAddresses": null,
        "SecondaryIPv6Addresses": null,
        "Service": null
    },
    "NoNewPrivileges": false,
    "Path": "sh",
    "ProcessLabel": "",
    "ResolvConfPath": "",
    "RestartCount": 0,
    "SeccompProfile": "",
    "SecretReferences": null,
    "ShmPath": "",
    "State": {
        "Dead": false,
        "Error": "",
        "ExitCode": 0,
        "FinishedAt": "0001-01-01T00:00:00Z",
        "Health": null,
        "OOMKilled": false,
        "Paused": false,
        "Pid": 0,
        "RemovalInProgress": false,
        "Restarting": false,
        "Running": false,
        "StartedAt": "0001-01-01T00:00:00Z"
    },
    "StreamConfig": {}
}

```

```
# cat containers/6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d/hostconfig.json | python -m json.tool
{
    "AutoRemove": false,
    "Binds": null,
    "BlkioDeviceReadBps": null,
    "BlkioDeviceReadIOps": null,
    "BlkioDeviceWriteBps": null,
    "BlkioDeviceWriteIOps": null,
    "BlkioWeight": 0,
    "BlkioWeightDevice": null,
    "CapAdd": null,
    "CapDrop": null,
    "Cgroup": "",
    "CgroupParent": "",
    "ConsoleSize": [
        0,
        0
    ],
    "ContainerIDFile": "",
    "CpuCount": 0,
    "CpuPercent": 0,
    "CpuPeriod": 0,
    "CpuQuota": 0,
    "CpuRealtimePeriod": 0,
    "CpuRealtimeRuntime": 0,
    "CpuShares": 0,
    "CpusetCpus": "",
    "CpusetMems": "",
    "Devices": [],
    "DiskQuota": 0,
    "Dns": [],
    "DnsOptions": [],
    "DnsSearch": [],
    "ExtraHosts": null,
    "GroupAdd": null,
    "IOMaximumBandwidth": 0,
    "IOMaximumIOps": 0,
    "IpcMode": "",
    "Isolation": "",
    "KernelMemory": 0,
    "Links": [],
    "LogConfig": {
        "Config": {},
        "Type": "json-file"
    },
    "Memory": 0,
    "MemoryReservation": 0,
    "MemorySwap": 0,
    "MemorySwappiness": -1,
    "NanoCpus": 0,
    "NetworkMode": "default",
    "OomKillDisable": false,
    "OomScoreAdj": 0,
    "PidMode": "",
    "PidsLimit": 0,
    "PortBindings": {},
    "Privileged": false,
    "PublishAllPorts": false,
    "ReadonlyRootfs": false,
    "RestartPolicy": {
        "MaximumRetryCount": 0,
        "Name": "no"
    },
    "Runtime": "runc",
    "SecurityOpt": null,
    "ShmSize": 67108864,
    "UTSMode": "",
    "Ulimits": null,
    "UsernsMode": "",
    "VolumeDriver": "",
    "VolumesFrom": null
}
```

### 4.1 Start Container

```
# docker start busybox
busybox
```

```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6b89258c98ed        busybox             "sh"                20 minutes ago      Up 3 seconds                            busybox

```

变化的文件系统：

```
# tree
.
|-- containers
|   `-- 6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d
|       |-- 6b89258c98ed9f2622aae7bc5f285f179e1f342a774043313634e1642ae84c4d-json.log
|       |-- hostname [6b89258c98ed]
|       |-- hosts [default hosts]
|       |-- resolv.conf [search localdomain\n nameserver 172.16.132.2]
|       |-- resolv.conf.hash [ad04558079be07f81ba7495b46cf93dc224f00df70c812d520bb28ca193b503a]
|       `-- shm

|-- overlay
|   |-- acb73196bd7d563a3a167b61e564a6597d63dad559c0731343d030d5707f06a5
|   |   |-- lower-id
|   |   |-- merged [此处包含]
|   |   |   |-- bin
|   |   |   |   ...... root system, 此处包含了root指向的全部文件
|   |   |  
```


## 5. Docker Regitstry

### 5.1 Image Storage 结构
```
# docker push 172.16.132.8:5000/busybox
The push refers to a repository [172.16.132.8:5000/busybox]
c0de73ac9968: Pushed
latest: digest: sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd size: 505
```

```
latest: Pulling from 172.16.132.8:5000/busybox
Digest: sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd
Status: Downloaded newer image for 172.16.132.8:5000/busybox:latest
```

```
# tree
`-- docker
    `-- registry
        `-- v2
            |-- blobs
            |   `-- sha256
            |       |-- 00
            |       |   `-- 00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff
            |       |       `-- data [image json]
            |       |-- 75
            |       |   `-- 7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3
            |       |       `-- data [gzip->tar->root根系统]
            |       `-- 92
            |           `-- 92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd
            |               `-- data
            `-- repositories
                `-- busybox
                    |-- _layers
                    |   `-- sha256
                    |       |-- 00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff
                    |       |   `-- link [sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff]
                    |       `-- 7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3
                    |           `-- link [sha256:7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3]
                    |-- _manifests
                    |   |-- revisions
                    |   |   `-- sha256
                    |   |       `-- 92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd
                    |   |           `-- link [sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd]
                    |   `-- tags
                    |       `-- latest
                    |           |-- current
                    |           |   `-- link [sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd]
                    |           `-- index
                    |               `-- sha256
                    |                   `-- 92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd
                    |                       `-- link [sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd]
                    `-- _uploads
```

```
# cat blobs/sha256/00/00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff/data

{
    "architecture": "amd64",
    "config": {
        "AttachStderr": false,
        "AttachStdin": false,
        "AttachStdout": false,
        "Cmd": [
            "sh"
        ],
        "Domainname": "",
        "Entrypoint": null,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Hostname": "1295ff10ed92",
        "Image": "sha256:0d7e86beb406ca2ff3418fa5db5e25dd6f60fe7265d68a9a141a2aed005b1ae7",
        "Labels": {},
        "OnBuild": null,
        "OpenStdin": false,
        "StdinOnce": false,
        "Tty": false,
        "User": "",
        "Volumes": null,
        "WorkingDir": ""
    },
    "container": "d12e9fb4928df60ac71b4b47d56b9b6aec383cccceb3b9275029959403ab4f73",
    "container_config": {
        "AttachStderr": false,
        "AttachStdin": false,
        "AttachStdout": false,
        "Cmd": [
            "/bin/sh",
            "-c",
            "#(nop) ",
            "CMD [\"sh\"]"
        ],
        "Domainname": "",
        "Entrypoint": null,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "Hostname": "1295ff10ed92",
        "Image": "sha256:0d7e86beb406ca2ff3418fa5db5e25dd6f60fe7265d68a9a141a2aed005b1ae7",
        "Labels": {},
        "OnBuild": null,
        "OpenStdin": false,
        "StdinOnce": false,
        "Tty": false,
        "User": "",
        "Volumes": null,
        "WorkingDir": ""
    },
    "created": "2017-03-09T18:28:04.586987216Z",
    "docker_version": "1.12.6",
    "history": [
        {
            "created": "2017-03-09T18:28:03.975884948Z",
            "created_by": "/bin/sh -c #(nop) ADD file:c9ecd8ff00c653fb652ad5a0a9215e1f467f0cd9933653b8a2e5e475b68597ab in / "
        },
        {
            "created": "2017-03-09T18:28:04.586987216Z",
            "created_by": "/bin/sh -c #(nop)  CMD [\"sh\"]",
            "empty_layer": true
        }
    ],
    "os": "linux",
    "rootfs": {
        "diff_ids": [
            "sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9"
        ],
        "type": "layers"
    }
}
```

```
# ls -hl blobs/sha256/75/7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3/data
-rw-r--r-- 1 root root 662K Apr  9 02:36 data

file data
data: gzip compressed data

mkdir test && cp data test/ && cd test && mv data data.gz && gzip -d data.gz && tar xvf data

# ls -hl  # 可见data保存的gzip(tar(data))
total 1.3M
drwxr-xr-x 2 root   root   8.0K Mar  8 16:05 bin
-rw-r--r-- 1 root   root   1.3M Apr  9 02:51 data
drwxr-xr-x 2 adm    sys       6 Mar  8 16:05 dev
drwxr-xr-x 2 root   root     60 Mar  8 16:05 etc
drwxr-xr-x 2 nobody nobody    6 Mar  8 16:05 home
drwxr-xr-x 2 root   root      6 Mar  8 16:05 root
drwxrwxrwt 2 root   root      6 Mar  8 16:05 tmp
drwxr-xr-x 3 root   root     17 Mar  8 16:05 usr
drwxr-xr-x 4 root   root     28 Mar  8 16:05 var
```


```
# cat v2/blobs/sha256/92/92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd/data
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/octet-stream",
      "size": 1465,
      "digest": "sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 677607,
         "digest": "sha256:7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3"
      }
   ]
}
```

## 5.2 拉取Image接口

```
GET /v2/busybox/manifests/latest HTTP/1.1
Host: 172.16.132.8:5000
User-Agent: docker/17.03.1-ce go/go1.7.5 git-commit/c6d412e kernel/4.9.13-moby os/linux arch/amd64 UpstreamClient(Docker-Client/17.03.1-ce \(darwin\))
Accept: application/vnd.docker.distribution.manifest.v2+json
Accept: application/vnd.docker.distribution.manifest.list.v2+json
Accept: application/vnd.docker.distribution.manifest.v1+prettyjws
Accept: application/json
Accept-Encoding: gzip
Connection: close

HTTP/1.1 200 OK
Content-Length: 505
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Docker-Content-Digest: sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:92b7c19467bf868e52949d7e5404e309486ba9ee7eb4b88b882747ee1522d3cd"
X-Content-Type-Options: nosniff
Date: Sun, 09 Apr 2017 10:29:55 GMT
Connection: close

{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/octet-stream",
      "size": 1465,
      // Image ID 
      "digest": "sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 677607,
         // Layer diff data
         "digest": "sha256:7520415ce76232cdd62ecc345cea5ea44f5b6b144dc62351f2cd2b08382532a3"
      }
   ]
}
```

Link: 

* [Image Manifest V 2, Schema 2](https://docs.docker.com/registry/spec/manifest-v2-2/#image-manifest-field-descriptions)
* [Docker Registry HTTP API V2](https://docs.docker.com/registry/spec/api/)
* [开启docker之旅](http://blog.suconghou.cn/post/using-docker/)
* [Docker rReference](https://docs.docker.com/reference/)



## 5.3 Create New Image

update: 2017.04.24


	$ cat Dockerfile
	FROM busybox
	
	ADD hello.txt /
	
	$cat hello.txt
	hello

	$ docker build -t busybox:v1.0 .
	
	$ docker images
	busybox                     v1.0                7dbf4402b79c        13 minutes ago      1.11 MB



	|-- imagedb
	|   |-- content
	|   |   `-- sha256
	|   |       |-- 00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff
	|   |       `-- 7dbf4402b79c68a58c11d2343f6bd59a22ee52980b3f5d5be29990fa49426bbc  新增加，镜像id
	|   `-- metadata
	|       `-- sha256
	|           `-- 7dbf4402b79c68a58c11d2343f6bd59a22ee52980b3f5d5be29990fa49426bbc
	|               `-- parent  [sha256:00f017a8c2a6e1fe2ffd05c281f27d069d2a99323a8cd514dd35f228ba26d2ff]
	
	
	|-- layerdb
	|   |-- sha256
	|   |   |-- 3e1127ba68b26d89fe62c41714bb5ff55a73cc58f8caa399156c074642646ae5 CHAIN_ID
	|   |   |   |-- cache-id [22c2ea6bd5847202f9c959538aced6b49ecaa963a72ac8a535c4003375a3a2db]
	|   |   |   |-- diff [sha256:794b01691d296c35552ed723fcdc865625149137a7f19f565fed5b123919687f] DIFFID
	|   |   |   |-- parent [sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9] PARENT CHAINID
	|   |   |   |-- size
	|   |   |   `-- tar-split.json.gz
	
	
	3e1127ba68b26d89fe62c41714bb5ff55a73cc58f8caa399156c074642646ae5 
	= sha256((parent_chainid) + " " + diffid) 
	= sha256("sha256:c0de73ac99683640bc8f8de5cda9e0e2fc97fe53d78c9fd60ea69b31303efbc9 sha256:794b01691d296c35552ed723fcdc865625149137a7f19f565fed5b123919687f")
