---
layout: post
title:  Docker笔记
date:   2021-06-25
author: Jay Chen
tags:   [Docker]
mathjax: false
description: "How to start with Docker, and how to use the Docker-compose"
---


# Docker下载安装

## Linux centos7版

安装地址：https://docs.docker.com/engine/install/centos/

```shell
# 安装gcc相关环境
yum -y install gcc
yum -y install gcc-c++
#卸载已有的docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#需要的安装包
yum install -y yum-utils

#设置镜像仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo #默认是国外的，速度慢
    #可以替换成阿里云的镜像
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#安装docker相关内容
yum install docker-ce docker-ce-cli containerd.io#此时报错，需要安装Podman
#添加该步骤
yum erase podman buildah
#后再进行安装docker
yum install docker-ce docker-ce-cli containerd.io

#启动docker
systemctl start docker

     
```





# Docker的常用命令

## 帮助命令

官网帮助文档：https://docs.docker.com/engine/reference/commandline/

## 镜像命令

* `docker images` 

查看所有镜像信息，--help可以查看相关命令

```shell
Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

* `docker search`

搜索镜像

```shell
[root@10-13-103-211 ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10999     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4165      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   815                  [OK]
percona                           Percona Server is a fork of the MySQL relati…   544       [OK]       
phpmyadmin                        phpMyAdmin - A web interface for MySQL and M…   240       [OK]       
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   88                   
mysql/mysql-cluster               Experimental MySQL Cluster Docker images. Cr…   86                   
centurylink/mysql                 Image containing mysql. Optimized to be link…   59                   [OK]
bitnami/mysql                     Bitnami MySQL Docker Image                      52                   [OK]
databack/mysql-backup             Back up mysql databases to... anywhere!         44                   
deitch/mysql-backup               REPLACED! Please use http://hub.docker.com/r…   41                   [OK]


[root@10-13-103-211 ~]# docker search --help

Usage:  docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output

```

* `docker pull`

下载镜像

```shell
[root@10-13-103-211 ~]# docker pull mysql
Using default tag: latest#不写tag默认是latest
latest: Pulling from library/mysql
69692152171a: Pull complete #分层下载，联合文件系统，不同镜像有些部分可以共用
1651b0be3df3: Pull complete 
...
```

* `docker rmi`

删除镜像

```shell
#docker rmi -f 镜像id  //删除指定的镜像
#docker rmi -f 镜像id 镜像id 镜像id //删除多个镜像
#docker rmi -f $(docker images -aq) //删除所有镜像
```



## 容器命令

备注：有了镜像才可以创建容器，首先下载一个centos作为镜像

```shell
[root@10-13-103-211 ~]# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
7a0437f04f83: Downloading [==========================================>        ]   63.3MB/75.18MB
```

新建容器并启动

```shell
docker run [可选参数] image

#参数说明
Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cgroupns string                Cgroup namespace to use (host|private)
                                       'host':    Run the container in the Docker host's cgroup namespace
                                       'private': Run the container in its own private cgroup namespace
                                       '':        Use the cgroup namespace as configured by the
                                                  default-cgroupns-mode option on the daemon (default)
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --pull string                    Pull image before running ("always"|"missing"|"never") (default "missing")
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
  
  
  
 #启动并进入容器
[root@10-13-103-211 ~]# docker run -it centos /bin/bash
[root@7a707b24b200 /]# 

#退回主机
[root@7a707b24b200 /]# exit


```

**列出所有运行的容器**

```shell
# docker ps 命令 列出正在运行的容器

#参数
-a #列出所有运行过的容器
-n=? #列出最近创建的容器
-q #只显示容器的编号
[root@10-13-103-211 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@10-13-103-211 ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED         STATUS                      PORTS     NAMES
7a707b24b200   centos        "/bin/bash"   2 minutes ago   Exited (0) 39 seconds ago             hopeful_brahmagupta
db78854c3339   hello-world   "/hello"      42 hours ago    Exited (0) 42 hours ago               xenodochial_brattain
b743855766cb   hello-world   "/hello"      42 hours ago    Exited (0) 42 hours ago               kind_stonebraker
38fe103c387b   hello-world   "/hello"      42 hours ago    Exited (0) 42 hours ago               suspicious_haslett
```

==进入当前正在运行的容器==

```shell
# 通常容器都是后台运行的，需要进入容器，修改一些配置

# 命令
docker exec -it 容器id #进入容器后开启新的终端

[root@10-13-103-211 ~]# docker exec -it 20f358b00913 /bin/bash

docker attach 容器id #进入容器正在执行的终端，不会启动新的终端
[root@10-13-103-211 ~]# docker attach 20f358b00913
```

**退出容器**

```shell
exit #停止容器并退出
Ctrl + P + Q #容器后台运行
```

**删除容器**

```shell
docker rm 容器id
docker rm -f $(docker ps -aq) #强制删除所有的容器，参数传递查询所有容器的id
```

**启动和停止容器的操作**

```shell
docker start 容器id
docker restart 容器id
docker stop 容器id
dicler kill 容器id
```

## 常用其他命令

**后台启动容器**

```shell
[root@10-13-103-211 ~]# docker run -d centos

#问题：docker ps 发现没有容器在运行

#常见的坑，docker容器使用后台运行，就必须要有一个前台的进程，否则docker会自动停止
```

**查看日志**

```shell
docker logs -f -t --tail (正在运行的容器id)

#自己编写一段shell脚本
[root@10-13-103-211 ~]# docker run -d centos /bin/sh -c "while true; do echo hello world; sleep 2; done"
#一直运行打印操作

[root@10-13-103-211 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
20f358b00913   centos    "/bin/sh -c 'while t…"   33 seconds ago   Up 32 seconds             distracted_pascal

#显示日志
-tf #显示日志
--tail [number] #显示日志的条数

[root@10-13-103-211 ~]# docker logs -tf --tail 10 20f358b00913
2021-06-18T08:36:14.056714223Z hello world
2021-06-18T08:36:16.058667099Z hello world
2021-06-18T08:36:18.060694853Z hello world
2021-06-18T08:36:20.062664582Z hello world
2021-06-18T08:36:22.064525610Z hello world
2021-06-18T08:36:24.066390388Z hello world
2021-06-18T08:36:26.068522810Z hello world
2021-06-18T08:36:28.070491153Z hello world
2021-06-18T08:36:30.072575259Z hello world
2021-06-18T08:36:32.074955590Z hello world
2021-06-18T08:36:34.077074175Z hello world
```

**查看容器中进程信息**

```shell
# docker top 容器id
[root@10-13-103-211 ~]# docker top 20f358b00913
```

**查看镜像的源数据**

```shell
# 命令 docker inspect 容器id
[root@10-13-103-211 ~]# docker inspect 20f358b00913
[
    {
        "Id": "20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04",
        "Created": "2021-06-18T08:34:53.68543703Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true; do echo hello world; sleep 2; done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 17986,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-06-18T08:34:53.975942389Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04/hostname",
        "HostsPath": "/var/lib/docker/containers/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04/hosts",
        "LogPath": "/var/lib/docker/containers/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04-json.log",
        "Name": "/distracted_pascal",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/2a30d3912842cb4818438f47bfe92528c09d35475161c2605abdebeb8e77d680-init/diff:/var/lib/docker/overlay2/bd08aba24bc52fd61c1a6596333b5230b0a055fa3fd763fa16ab452bf013e3bb/diff",
                "MergedDir": "/var/lib/docker/overlay2/2a30d3912842cb4818438f47bfe92528c09d35475161c2605abdebeb8e77d680/merged",
                "UpperDir": "/var/lib/docker/overlay2/2a30d3912842cb4818438f47bfe92528c09d35475161c2605abdebeb8e77d680/diff",
                "WorkDir": "/var/lib/docker/overlay2/2a30d3912842cb4818438f47bfe92528c09d35475161c2605abdebeb8e77d680/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "20f358b00913",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true; do echo hello world; sleep 2; done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "ccb548363841bc5cba11a6f08a260a46af06bb897c0f13c5084b5439b94fc629",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/ccb548363841",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "eef8d8e1c46f67d3ac055d26fd418a28ce94bc4a730bdbc65a3a8d91e558d8d9",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "8e97a1a75b87beae8d44664db390af82d87b6124cb078dea7dc5304130b5c3ac",
                    "EndpointID": "eef8d8e1c46f67d3ac055d26fd418a28ce94bc4a730bdbc65a3a8d91e558d8d9",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

**容器内拷贝文件到主机上**

```shell
docker cp 容器id：容器内路径 主机路径
```

拷贝是一个手动过程，未来可以通过 -v 卷的技术，可以实现自动同步。

## 小结

<img width="780" alt="image-20210618174858353" src="https://user-images.githubusercontent.com/48710834/123402283-2a0cc980-d5da-11eb-9e7e-e7be96089858.png">
<img width="651" alt="image-20210621153656713" src="https://user-images.githubusercontent.com/48710834/123402568-73f5af80-d5da-11eb-9da7-0e1f7ad6fe8c.png">


```shell
attach    Attach to a running container  #当前shell下attach连接指定运行镜像
build     Build an image from a Dockerfile  #通过Dockerfile定制镜像
commit    Create a new image from a containers changes  #提交当前容器为新的镜像
cp    Copy files/folders from a container to a HOSTDIR or to STDOUT  #从容器中拷贝指定文件或者目录到宿主机中
create    Create a new container  #创建一个新的容器，同run 但不启动容器
diff    Inspect changes on a containers filesystem  #查看docker容器变化
events    Get real time events from the server#从docker服务获取容器实时事件
exec    Run a command in a running container#在已存在的容器上运行命令
export    Export a containers filesystem as a tar archive  #导出容器的内容流作为一个tar归档文件(对应import)
history    Show the history of an image  #展示一个镜像形成历史
images    List images  #列出系统当前镜像
import    Import the contents from a tarball to create a filesystem image  #从tar包中的内容创建一个新的文件系统映像(对应export)
info    Display system-wide information  #显示系统相关信息
inspect    Return low-level information on a container or image  #查看容器详细信息
kill    Kill a running container  #kill指定docker容器
load    Load an image from a tar archive or STDIN  #从一个tar包中加载一个镜像(对应save)
login    Register or log in to a Docker registry#注册或者登陆一个docker源服务器
logout    Log out from a Docker registry  #从当前Docker registry退出
logs    Fetch the logs of a container  #输出当前容器日志信息
pause    Pause all processes within a container#暂停容器
port    List port mappings or a specific mapping for the CONTAINER  #查看映射端口对应的容器内部源端口
ps    List containers  #列出容器列表
pull    Pull an image or a repository from a registry  #从docker镜像源服务器拉取指定镜像或者库镜像
push    Push an image or a repository to a registry  #推送指定镜像或者库镜像至docker源服务器
rename    Rename a container  #重命名容器
restart    Restart a running container  #重启运行的容器
rm    Remove one or more containers  #移除一个或者多个容器
rmi    Remove one or more images  #移除一个或多个镜像(无容器使用该镜像才可以删除，否则需要删除相关容器才可以继续或者-f强制删除)
run    Run a command in a new container  #创建一个新的容器并运行一个命令
save    Save an image(s) to a tar archive#保存一个镜像为一个tar包(对应load)
search    Search the Docker Hub for images  #在dockerhub中搜索镜像
start    Start one or more stopped containers#启动容器
stats    Display a live stream of container(s) resource usage statistics  #统计容器使用资源
stop    Stop a running container  #停止容器
tag         Tag an image into a repository  #给源中镜像打标签
top       Display the running processes of a container #查看容器中运行的进程信息
unpause    Unpause all processes within a container  #取消暂停容器
version    Show the Docker version information#查看容器版本号
wait         Block until a container stops, then print its exit code  #截取容器停止时的退出状态值
```



## 作业练习

> 作业一：Docker 安装Nginx

* 1、搜索镜像 search
* 2、下载镜像 docker pull
* 3、运行测试

```shell
[root@10-13-103-211 ~]# docker run -d --name nginx01 -p 3340:80
"docker run" requires at least 1 argument.
See 'docker run --help'.

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container
[root@10-13-103-211 ~]# docker run -d --name nginx01 -p 3340:80 nginx
02371c0ad7dfb7b305e3ff954c4940d99dfe54f9e37efa182f03305eaf6b19b6
[root@10-13-103-211 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
02371c0ad7df   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 8 seconds   0.0.0.0:3340->80/tcp, :::3340->80/tcp   nginx01
[root@10-13-103-211 ~]# curl localhost:3340
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

思考：每次改动容器内部文件都需要进入容器，是否可以在容器外部提供一个映射路径，在外部修改文件？

**-v 数据卷**



> 作业二：使用docker 安装tomcat

```shell
#	官方使用
$ docker run -it --rm tomcat:9.0

# --rm方法，用完即删除

# 下载再启动
docker pull tomcat:9.0

# 启动运行
 docker run -d -p 3355:8080 --name tomcat01 tomcat
 
 #进入容器
 docker exec -it tomcat01 /bin/bash
 
```



> 作业三：部署es + kibana

```shell
# es 暴露的端口很多
# es 十分耗内存
# es 数据需要放置到安全目录：挂载

# --net somenetwork 网络配置
 docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.13.2
#启动后linux卡住，因为elasticsearch非常耗内存
docker stats 容器id#查看cpu状态

#关闭elasticsearch 并增加内存限制，修改配置文件 -e 环境配置修改
 docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.13.2
#测试启动es是否成功
curl localhost:9200
{
  "name" : "6b9bb1b2aa7e",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "QO7-WTNeRZW1HQM6k38cXA",
  "version" : {
    "number" : "7.13.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "4d960a0733be83dd2543ca018aa4ddc42e956800",
    "build_date" : "2021-06-10T21:01:55.251515791Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



> 作业四：使用kibana连接elaticsearch



## 可视化

* portainer

Docker图形化界面管理工具

```shell
#下载
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
#访问测试
#注意先打开防火墙访问设置
117.50.101.24:8088   #外网8088
#设置账户密码选择 local挂载即可

```

可视化面板平时很少用到。

* Rancher（CI/CD再用）

# Docker镜像

## Docker镜像加载原理

> UFS（联合文件系统）

<font color=blue>现象：下载镜像时，文件是一层一层下载的，每一层都有其ID，可以给其他镜像复用，节省空间；</font>

==UFS：== 联合文件系统（[UnionFS](https://en.wikipedia.org/wiki/UnionFS)）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。



> Docker镜像加载原理



![Uploading image-20210621153656713.png…]()


Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层就是我们通常说的容器层，容器之下的都叫做镜像层。

> 分层存储

因为镜像包含操作系统完整的 `root` 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

## commit镜像

```shell
#启动容器进入后，可以制作新的镜像
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```



# 容器数据卷

## 什么是容器数据卷

==Docker理念==

讲应用和环境打包成一个镜像。如果数据都在容器中，删除容器之后，数据都会丢失。

<font color=red> 需求：数据可以持久化，数据可以存储在本地！</font>

**容器之间可以有一个数据共享的技术，Docker容器中产生的数据同步到本地。**

将容器内的目录挂载到宿主机/其他容器上面

## 使用数据卷

> 方式一：直接使用命令来挂载 -v

```shell
docker run -it -v 主机目录:容器内目录

#同步主机上 /ceshi 目录与容器内 /home 目录
[root@10-9-70-238 ~]# docker run -it -v /ceshi:/home centos /bin/bash

#查看容器信息,Mounts项即为挂载信息
[root@10-9-70-238 ~]# docker inspect 3e28dcdd7f95
"Mounts": [
            {
                "Type": "bind",
                "Source": "/ceshi",
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],                
```

* 测试：

1、停止容器；

2、宿主机上修改文件；

3、启动容器；

4、容器内的数据依旧会同步；

==Q&A==

Q：删除文件也会同步，容器删除之后，映射路径处文件是否会删除？

A: 经过测试，文件依旧会保留。

## 实战：安装MySQL

```shell
# 获取镜像
[root@10-9-70-238 ceshi]# docker pull mysql

# 运行容器，并做数据挂载
# dockerhub中mysql启动命令，需要配置，密码
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动mysql
-d 后台运行
-p 端口映射(宿主机端口3310，容器端口3306)
-v 数据卷挂载
-e 环境配置（密码设置）
--name 容器名字
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql

# 在本地使用mysql客户端连接测试
# 客户端新建数据库，在宿主机挂载目录下有同步更新

# 容器删除之后，本地mysql数据库依然存在。
```



## 具名和匿名挂载

```shell
# 匿名挂载，可以不写宿主机路径
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx
# 查看所有 volume 卷 的情况
docker volume ls


# 具名挂载，给挂载的卷指定名字但是没有写宿主机路径
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx

[root@10-9-70-238 ~]# docker volume ls
DRIVER    VOLUME NAME
local     4bd5255c6926b4e4f192756ecb5ab8100cf4c71263aa0b8227faaf385b9cba5d
local     91e4816758284ae4f54260aba206bcacf7e405f1026936223d1661a73c82b1b5
local     juming-nginx
# 查看卷的信息
[root@10-9-70-238 ~]# docker volume inspect juming-nginx 
[
    {
        "CreatedAt": "2021-06-22T13:57:51+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]

```

<font color=ff8033>注：所有的Docker容器内的卷，没有指定目录的情况下都是在此路径：</font> `/var/lib/docker/volumes/xxx/_data`

拓展

```shell
# 通过 -v 容器内路径，ro rw 改变读写权限
ro # 只读
rw # 可读可写
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

# ro 表示：路径下文件只能通过宿主机来操作，容器内部无法操作！
```

## 初识Dockerfile

Dockerfile ：用来构建Docker镜像的文件，类似于命令脚本

```shell
#	dockerfile文件 dockerfile1
# dockerfile 内容中直接进行挂载
FROM centos
VOLUME ["volume01", "volume02"]

CMD echo "-----end------"
CMD /bin/bash
# 每个命令就是镜像的每一层

```

## 数据卷容器

```shell
# 首先启动一个容器，docker01

docker run -it --name docker02 --volumes-from docker01 centos

# 容器之间继承关系，删除docker01之后，其他容器依旧可以访问这个文件
```

<font color = red> 容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止。但是一旦持久化到了本地，本地数据不会删除</font>

# Dockerfile

## Dockerfile介绍

Dockerfile ：用来构建Docker镜像的文件，类似于命令脚本

构建步骤：

* 编写一个dockerfile文件
* docker build 构建成为一个镜像
* docker run 运行镜像
* docker push 发布镜像（DockerHub或者阿里云镜像仓库）

## Dockerfile构建过程

基础知识：

* 每个保留关键字都是大写字母
* 执行顺序为从上到下
* 注释符号：#
* 每一个指令都会创建一个新的镜像层

## Dockerfile指令

```shell
FROM					#基础镜像
MAINTAINER		#镜像维护者信息，姓名+邮箱
RUN						#镜像构建的时候需要运行的命令
ADD						#添加内容，copy文件，会自动解压
WORKDIR				#镜像的工作目录
VOLUME				#设置卷，挂载主机目录
EXPOSE				#制定对外暴露端口
CMD						#指定容器启动的时候需要运行的命令，最后一个命令会替代之前的
ENTRYPOINT		#指定容器启动的时候需要运行的命令，可以追加命令
ONBUILD				#当构建一个被继承Dockerfile，这时候就会运行ONBULID命令，属于触发指令。
COPY					#类似ADD，将文件拷贝到镜像中
ENV						#构建的时候设置环境变量
```

Docker Hub中大部分镜像都是从基础镜像创建的

`FROM scratch`



> 创建一个自己的centos
>
> 

```shell
# 编写配置文件Dockerfile-centos
FROM centos
MAINTAINER jay.chen<jay.chen@ucloud.cn>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "-----end------"
CMD /bin/bash

# 通过dockerfile构建镜像
docker build -f Dockerfile-centos -t mycentos:1.0 . # 注意要加一个.

# 查看镜像版本历史
docker history 镜像id

```

## 发布镜像

> 发布到dockerhub

步骤：

* 登陆账号
* 在服务器上提交

```shell
# 登陆账号
docker login -u jaychen1996
# 提交镜像
docker push 镜像ID：tag
```



> 发布到阿里云镜像服务上

* 登陆阿里云
* 找到容器镜像服务
* 创建命名空间
* 创建镜像仓库
* 参考官方文档

## 小结

<img width="607" alt="image-20210623141215167" src="https://user-images.githubusercontent.com/48710834/123402699-9982b900-d5da-11eb-9fcb-98696058120b.png">


# Docker网络

## 理解Docker0

```shell
# 查看ip地址
[root@10-9-70-238 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1452 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:5f:f6:a7 brd ff:ff:ff:ff:ff:ff
    inet 10.9.70.238/16 brd 10.9.255.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe5f:f6a7/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:66:de:df:64 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:66ff:fede:df64/64 scope link 
       valid_lft forever preferred_lft forever
```

> docker是如何处理容器网络访问的

```shell
# 启动一个tomcat容器
[root@10-9-70-238 ~]# docker run -d -P --name tomcat01 tomcat

# 查看容器内部网络地址 ip addr,得到eth0@if19 ip地址，为docker分配的
[root@10-9-70-238 ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
18: eth0@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
# linux可以ping通容器内部
[root@10-9-70-238 ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.051 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.048 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.048 ms

```

> 原理

* 每次启动一个docker容器，docker就会给docker容器分配一个ip，只要安装了docker ，就会有一个网卡docker0。桥接模式，使用veth-pair技术。

  ```shell
  [root@10-9-70-238 ~]# ip addr
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host 
         valid_lft forever preferred_lft forever
  2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1452 qdisc fq_codel state UP group default qlen 1000
      link/ether 52:54:00:5f:f6:a7 brd ff:ff:ff:ff:ff:ff
      inet 10.9.70.238/16 brd 10.9.255.255 scope global noprefixroute eth0
         valid_lft forever preferred_lft forever
      inet6 fe80::5054:ff:fe5f:f6a7/64 scope link 
         valid_lft forever preferred_lft forever
  3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
      link/ether 02:42:66:de:df:64 brd ff:ff:ff:ff:ff:ff
      inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
         valid_lft forever preferred_lft forever
      inet6 fe80::42:66ff:fede:df64/64 scope link 
         valid_lft forever preferred_lft forever
  19: veth769efa0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
      link/ether 66:a4:99:2d:76:33 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      inet6 fe80::64a4:99ff:fe2d:7633/64 scope link 
         valid_lft forever preferred_lft forever
         
  # veth-pair技术增加了一个19: veth769efa0@if18 ip地址
  # 增加了一个接口可以打通容器内部网络与宿主机网络
  ```



* tomcat01 与 tomcat02 之间是可以ping通的

<img width="740" alt="image-20210623155205686" src="https://user-images.githubusercontent.com/48710834/123402760-ab645c00-d5da-11eb-92cf-c78811395d6c.png">


Docker中的网络接口都是虚拟的。容器删除后，对应网桥消失。

## 自定义网络

> 查看所有的docker网络

```shell
[root@10-9-70-238 ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cc48bf7d6cdb   bridge    bridge    local
bceef51a266d   host      host      local
3afebd3225db   none      null      local
```

网络模式

* bridge：桥接模式，docker中默认
* none：不配置网络
* host：与宿主机共享网络
* container：容器网络联通

```shell
# 创建自定义网络
[root@10-9-70-238 ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

# 查看自定义网络信息
[root@10-9-70-238 ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "e880a82b34ac10955ae0f5e1d56ba60ace642befe63d85113a463cde5d48985a",
        "Created": "2021-06-23T16:56:55.879201401+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

# 启动容器并绑定自定义网络上
[root@10-9-70-238 ~]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
0af4b6baf1524e5e8d50ade05435a769e70ab6030cd65451c2be56b694d8d011
[root@10-9-70-238 ~]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
e4ab88e485622dbab77da565de81cc1d45d90a114dff23499eab73b8116b8284

# 查看网络信息
[root@10-9-70-238 ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "e880a82b34ac10955ae0f5e1d56ba60ace642befe63d85113a463cde5d48985a",
        "Created": "2021-06-23T16:56:55.879201401+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0af4b6baf1524e5e8d50ade05435a769e70ab6030cd65451c2be56b694d8d011": {
                "Name": "tomcat-net-01",
                "EndpointID": "bc1ecdc1ddff3e5073a1fc361aec97de8b29022c956fac1d8fbfc1425c8371ec",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "e4ab88e485622dbab77da565de81cc1d45d90a114dff23499eab73b8116b8284": {
                "Name": "tomcat-net-02",
                "EndpointID": "78a3f259f29abd99f1ea9ed20c3120e9d1cbc595b53a1cbd4761fc3f90142335",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

# 测试ping连接，不使用--link也可以ping名字
[root@10-9-70-238 ~]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.064 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.063 ms

```

==使用自定义网络的好处：==

不同的集群使用不同的网络，保证集群是安全的。



## 网络连通

容器中的tomcat与宿主机中的tomcat无法直接连通，但是通过自定义网络，利用connect创建静态路由表，可以将容器中tomcat连接到自定义网络之中。
<img width="675" alt="image-20210624105504134" src="https://user-images.githubusercontent.com/48710834/123402813-b8814b00-d5da-11eb-91f3-c2f55b364603.png">


```shell
[root@10-9-70-238 ~]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.

# 可以使用connect将一个容器连接到自定义宿主机网络上
[root@10-9-70-238 ~]# docker network connect mynet tomcat01 

# 查看自定义网络信息
[root@10-9-70-238 ~]# docker network inspect mynet
        "Containers": {
            "0af4b6baf1524e5e8d50ade05435a769e70ab6030cd65451c2be56b694d8d011": {
                "Name": "tomcat-net-01",
                "EndpointID": "bc1ecdc1ddff3e5073a1fc361aec97de8b29022c956fac1d8fbfc1425c8371ec",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "9e756fba71cc002e71e31d69739f52ddc8d852e04841dae5b59e27a80b499b74": {
                "Name": "tomcat01",
                "EndpointID": "6afb0838d3d06c518a622a8879d3aec0b7766db6fd9e4f8da1b02efbe30f4a30",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "e4ab88e485622dbab77da565de81cc1d45d90a114dff23499eab73b8116b8284": {
                "Name": "tomcat-net-02",
                "EndpointID": "78a3f259f29abd99f1ea9ed20c3120e9d1cbc595b53a1cbd4761fc3f90142335",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },

# 01连通ok，02不可连通
[root@10-9-70-238 ~]# docker exec -it tomcat01 ping tomcat-net-01
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.111 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.097 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.083 ms
```



## 部署Redis集群

```shell
# 创建自定义子网
[root@10-9-70-238 ~]# docker network create redis --subnet 172.38.0.0/16

# 通过脚本创建六个Redis配置
for port in $(seq 1 6);\
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

# 启动六个redis容器
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; \

docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

#部署集群
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: 2758f5660ea5dbddf56250f8c88da5dd120f86ab 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: 8d00e899016c4b6cd10e8dfa98c562c774c1284a 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: 7ada335c8811fa5482836cff83946d10c407bd34 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 898c6bba011e26bb5d64853dc49f0cb292c1eec3 172.38.0.14:6379
   replicates 7ada335c8811fa5482836cff83946d10c407bd34
S: c4b740de0c168f6f7d31a4bee8107195a44746d8 172.38.0.15:6379
   replicates 2758f5660ea5dbddf56250f8c88da5dd120f86ab
S: 17a8b4969af8dc458c7a6740f977dc8178999522 172.38.0.16:6379
   replicates 8d00e899016c4b6cd10e8dfa98c562c774c1284a
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: 2758f5660ea5dbddf56250f8c88da5dd120f86ab 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 7ada335c8811fa5482836cff83946d10c407bd34 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 898c6bba011e26bb5d64853dc49f0cb292c1eec3 172.38.0.14:6379
   slots: (0 slots) slave
   replicates 7ada335c8811fa5482836cff83946d10c407bd34
S: c4b740de0c168f6f7d31a4bee8107195a44746d8 172.38.0.15:6379
   slots: (0 slots) slave
   replicates 2758f5660ea5dbddf56250f8c88da5dd120f86ab
S: 17a8b4969af8dc458c7a6740f977dc8178999522 172.38.0.16:6379
   slots: (0 slots) slave
   replicates 8d00e899016c4b6cd10e8dfa98c562c774c1284a
M: 8d00e899016c4b6cd10e8dfa98c562c774c1284a 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 进入集群
/data # redis-cli -c

# 查看集群信息
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:1137
cluster_stats_messages_pong_sent:1144
cluster_stats_messages_sent:2281
cluster_stats_messages_ping_received:1139
cluster_stats_messages_pong_received:1137
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:2281

# 查看集群节点，三个master、三个slave
127.0.0.1:6379> cluster nodes
7ada335c8811fa5482836cff83946d10c407bd34 172.38.0.13:6379@16379 master - 0 1624514251548 3 connected 10923-16383
898c6bba011e26bb5d64853dc49f0cb292c1eec3 172.38.0.14:6379@16379 slave 7ada335c8811fa5482836cff83946d10c407bd34 0 1624514251146 4 connected
2758f5660ea5dbddf56250f8c88da5dd120f86ab 172.38.0.11:6379@16379 myself,master - 0 1624514252000 1 connected 0-5460
c4b740de0c168f6f7d31a4bee8107195a44746d8 172.38.0.15:6379@16379 slave 2758f5660ea5dbddf56250f8c88da5dd120f86ab 0 1624514253152 5 connected
17a8b4969af8dc458c7a6740f977dc8178999522 172.38.0.16:6379@16379 slave 8d00e899016c4b6cd10e8dfa98c562c774c1284a 0 1624514251000 6 connected
8d00e899016c4b6cd10e8dfa98c562c774c1284a 172.38.0.12:6379@16379 master - 0 1624514252150 2 connected 5461-10922

# 在redis-3上写健值对
127.0.0.1:6379> set a b
-> Redirected to slot [15495] located at 172.38.0.13:6379
OK
# 关闭redis-3
[root@10-9-70-238 /]# docker stop redis-3
redis-3
[root@10-9-70-238 /]# docker exec -it redis-1 /bin/sh
/data # redis-cli -c

# 测试取数据，数据自动转移到 redis-4上，redis-4成为master
127.0.0.1:6379> get a
-> Redirected to slot [15495] located at 172.38.0.14:6379
"b"
172.38.0.14:6379> cluster nodes
17a8b4969af8dc458c7a6740f977dc8178999522 172.38.0.16:6379@16379 slave 8d00e899016c4b6cd10e8dfa98c562c774c1284a 0 1624514723061 6 connected
7ada335c8811fa5482836cff83946d10c407bd34 172.38.0.13:6379@16379 master,fail - 1624514560299 1624514559000 3 connected
8d00e899016c4b6cd10e8dfa98c562c774c1284a 172.38.0.12:6379@16379 master - 0 1624514722560 2 connected 5461-10922
2758f5660ea5dbddf56250f8c88da5dd120f86ab 172.38.0.11:6379@16379 master - 0 1624514722000 1 connected 0-5460
c4b740de0c168f6f7d31a4bee8107195a44746d8 172.38.0.15:6379@16379 slave 2758f5660ea5dbddf56250f8c88da5dd120f86ab 0 1624514723000 5 connected
898c6bba011e26bb5d64853dc49f0cb292c1eec3 172.38.0.14:6379@16379 myself,master - 0 1624514721000 7 connected 10923-16383
```



# Docker compose

## 简介

> 官方介绍

* 定义运行多容器的工具
* YAML file 配置文件

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

* Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.

作用：批量容器编排

> 其他注意事项

compose是Docker官方的开源项目，需要自己安装；

A `docker-compose.yml` looks like this:

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

`docker compose up` 可以将配置文件中的服务都启起来。



## 安装

```shell
# 官网文档 https://docs.docker.com/compose/install/
# 下载
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 授权
sudo chmod +x /usr/local/bin/docker-compose
```

## Get started

1. Create a directory for the project:

   ```shell
   $ mkdir composetest
   $ cd composetest
   ```

2. Create a file called `app.py` in your project directory and paste this in:

   ```python
   import time
   
   import redis
   from flask import Flask
   
   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)
   
   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)
   
   @app.route('/')
   def hello():
       count = get_hit_count()
       return 'Hello World! I have been seen {} times.\n'.format(count)
   ```

3. Create another file called `requirements.txt` in your project directory and paste this in:

```shell
flask
redis
```

4. In this step, you write a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself.

   In your project directory, create a file named `Dockerfile` and paste the following:

   ```shell
   # syntax=docker/dockerfile:1
   FROM python:3.7-alpine
   WORKDIR /code
   ENV FLASK_APP=app.py
   ENV FLASK_RUN_HOST=0.0.0.0
   RUN apk add --no-cache gcc musl-dev linux-headers
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   EXPOSE 5000
   COPY . .
   CMD ["flask", "run"]
   ```

   This tells Docker to:

   - Build an image starting with the Python 3.7 image.
   - Set the working directory to `/code`.
   - Set environment variables used by the `flask` command.
   - Install gcc and other dependencies
   - Copy `requirements.txt` and install the Python dependencies.
   - Add metadata to the image to describe that the container is listening on port 5000
   - Copy the current directory `.` in the project to the workdir `.` in the image.
   - Set the default command for the container to `flask run`.

   For more information on how to write Dockerfiles, see the [Docker user guide](https://docs.docker.com/develop/) and the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

   

5. Define services in a Compose file

Create a file called `docker-compose.yml` in your project directory and paste the following:

```shell
version: "3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

This Compose file defines two services: `web` and `redis`.

* Web service

The `web` service uses an image that’s built from the `Dockerfile` in the current directory. It then binds the container and the host machine to the exposed port, `5000`. This example service uses the default port for the Flask web server, `5000`.

* Redis service

The `redis` service uses a public [Redis](https://registry.hub.docker.com/_/redis/) image pulled from the Docker Hub registry.



6. Build and run your app with Compose

From your project directory, start up your application by running `docker-compose up`.

> 流程分析

1. 创建网络
2. 执行Docker-compose yaml
3. 启动服务

Creating composetest_web_1  ... done

Creating composetest_redis_1 ... done

![image-20210625140241502](https://user-images.githubusercontent.com/48710834/123402889-cafb8480-d5da-11eb-93bb-e64cb8328945.png)

==好处：==通过docker-compose 编写yaml配置文件。可以通过compose一件启动所有服务。



## yaml规则（k8s常用）

Docker-compose.yaml 核心

```shell
# 一共三层

version: '' 
services:
	服务1:web
		#服务配置
		images
		build
		network
	服务2:redis
		···
# 其他配置
volumes
configs
network	
```

## 开源项目

> 搭建博客-wordpress

官网地址：https://docs.docker.com/samples/wordpress/

1、配置yaml文件

2、`docker-compose up -d`可以直接启动成功



# Dokcer Swarm

首先创建4个主机。

## Raft协议

双主双从：假设一个节点挂了，其他节点是否可以用；

Raft协议：保证大多数节点存活才可以用，集群至少有3台！（如果只有两台，一台宕机之后，没法判断是自己离线了还是对方离线）；









































