---
layout: post
title:  Docker - learning notes
date:   2021-06-25
categories:
  - Cloud Computing
tags:
  - Docker
  - Learning notes
---

How to start with Docker, and how to use the Docker-compose

{% include toc %}

# Docker download and installation

## Linux centos7 version

Installation address: https://docs.docker.com/engine/install/centos/

```shell
# Install gcc related environment
yum -y install gcc
yum -y install gcc-c++
# Uninstall existing docker
sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
# Required installation package
yum install -y yum-utils

# Set up the image repository
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo # The default is foreign, slow
# Can be replaced with Alibaba Cloud's image
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#Install docker related content
yum install docker-ce docker-ce-cli containerd.io#An error occurs at this time, and Podman needs to be installed
#Add this step
yum erase podman buildah
#Then install docker
yum install docker-ce docker-ce-cli containerd.io

#Start docker
systemctl start docker

```

# Common commands for Docker

## Help commands

Official website help document: https://docs.docker.com/engine/reference/commandline/

## Image command

* `docker images`

View all image information, --help can view related commands

```shell
Options:
-a, --all Show all images (default hides intermediate images)
--digests Show digests
-f, --filter filter Filter output based on conditions provided
--format string Pretty-print images using a Go template 
--no-trunc Don't truncate output
 -q, --quiet Only show image IDs
```

* `docker search`

Search for images

```shell
[root@10-13-103-211 ~]# docker search mysql
NAME DESCRIPTION STARS OFFICIAL AUTOMATED
mysql MySQL is a widely used, open-source relation… 10999 [OK]
mariadb MariaDB Server is a high performing open sou… 4165 [OK]
mysql/mysql-server Optimized MySQL Server Docker images. Create… 815 [OK]
percona Percona Server is a fork of the MySQL relative… 544 [OK]
phpmyadmin phpMyAdmin - A web interface for MySQL and M… 240 [OK]
centos/mysql-57-centos7 MySQL 5.7 SQL database server 88
mysql/mysql-cluster Experimental MySQL Cluster Docker images. Cr… 86
centurylink/mysql Image containing mysql. Optimized to be link… 59 [OK]
bitnami/mysql Bitnami MySQL Docker Image 52 [OK]
databack/mysql-backup Back up mysql databases to... anywhere! 44
deitch/mysql-backup REPLACED! Please use http://hub.docker.com/r… 41 [OK]


[root@10-13-103-211 ~]# docker search --help

Usage: docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
 -f, --filter filter Filter output based on conditions provided
 --format string Pretty-print search using a Go template
 --limit int Max number of search results (default 25)
 --no-trunc Don't truncate output

```

* `docker pull`

Download image

```shell
[root@10-13-103-211 ~]# docker pull mysql
Using default tag: latest#Default is latest if tag is not written
latest: Pulling from library/mysql
69692152171a: Pull complete #Layered download, joint file system, some parts of different images can be shared
1651b0be3df3: Pull complete
...
```

* `docker rmi`

Delete image

```shell
#docker rmi -f image id //Delete the specified image
#docker rmi -f image id image id image id //Delete multiple images
#docker rmi -f $(docker images -aq) //Delete all images
```

## Container command

Note: Only with an image can you create a container. First download a centos as an image

```shell
[root@10-13-103-211 ~]# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
7a0437f04f83: Downloading [==========================================> ] 63.3MB/75.18MB
```

Create a new container and start it

```shell
docker run [optional parameter] image

#Parameter description
Options:
 --add-host list Add a custom host-to-IP mapping (host:ip)
 -a, --attach list Attach to STDIN, STDOUT or STDERR
 --blkio-weight uint16 Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
 --blkio-weight-device list Block IO weight (relative device weight) (default [])
 --cap-add list Add Linux capabilities
 --cap-drop list Drop Linux capabilities
 --cgroup-parent string Optional parent cgroup for the container
 --cgroupns string Cgroup namespace to use (host|private)
 'host': Run the container in the Docker host's cgroup namespace
 'private': Run the container in its own private cgroup namespace
 '': Use the cgroup namespace as configured by the
 default-cgroupns-mode option on the daemon (default)
 --cidfile string Write the container ID to the file
 --cpu-period int Limit CPU CFS (Completely Fair Scheduler) period
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
 -P, --publish-all Publish all exposed ports to random ports
 --pull string Pull image before running ("always"|"missing"|"never") (default "missing")
 --read-only Mount the container's root filesystem as read only
 --restart string Restart policy to apply when a container exits (default "no")
 --rm Automatically remove the container when it exits
 --runtime string Runtime to use for this container
 --security-opt list Security Options
 --shm-size bytes Size of /dev/shm
 --sig-proxy Proxy received signals to the process (default true)
 --stop-signal string Signal to stop a container (default "SIGTERM")
 --stop-timeout int Timeout (in seconds) to stop a container
 --storage-opt list Storage driver options for the container
 --sysctl map Sysctl options (default map[])
 --tmpfs list Mount a tmpfs directory
 -t, --tty Allocate a pseudo-TTY
--ulimit ulimit Ulimit options (default [])
-u, --user string Username or UID (format: <name|uid>[:<group|gid>])
--userns string User namespace to use
--uts string UTS namespace to use
-v, --volume list Bind mount a volume
--volume-driver string Optional volume driver for the container
--volumes-from list Mount volumes from the specified container(s)
-w, --workdir string Working directory inside the container

#Start and enter the container
[root@10-13-103-211 ~]# docker run -it centos /bin/bash
[root@7a707b24b200 /]#

#Return to the host
[root@7a707b24b200 /]# exit

```

**List all running containers**

```shell
# docker ps command List running containers

#Parameters
-a #List all running containers
-n=? #List recently created containers
-q #Show only container IDs
[root@10-13-103-211 ~]# docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
[root@10-13-103-211 ~]# docker ps -a
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
7a707b24b200 centos "/bin/bash" 2 minutes ago Exited (0) 39 seconds ago hopeful_brahmagupta
db78854c3339 hello-world "/hello" 42 hours ago Exited (0) 42 hours ago xenodochial_brattain
b743855766cb hello-world "/hello" 42 hours ago Exited (0) 42 hours ago kind_stonebraker
38fe103c387b hello-world "/hello" 42 hours ago Exited (0) 42 hours ago suspicious_haslett
```

==Enter the currently running container==

```shell
# Usually the container is running in the background, you need to enter the container and modify some configurations

# Command
docker exec -it container id #Open a new terminal after entering the container

[root@10-13-103-211 ~]# docker exec -it 20f358b00913 /bin/bash

docker attach container id #Enter the terminal where the container is executing, and will not start a new terminal
[root@10-13-103-211 ~]# docker attach 20f358b00913
```

**Exit the container**

```shell
exit #Stop the container and exit
Ctrl + P + Q #Container background operation
```

**Delete container**

```shell
docker rm container id
docker rm -f $(docker ps -aq) #Forced deletion of all containers, parameter passing query all container ids
```

**Start and stop container operations**

```shell
docker start container id
docker restart container id
docker stop container id
dicler kill container id
```

## Other common commands

**Background start container**

```shell
[root@10-13-103-211 ~]# docker run -d centos

#Problem: docker ps found no container running

#Common pitfalls, docker container uses background operation, there must be a foreground process, otherwise docker will stop automatically
```

**View logs**

```shell
docker logs -f -t --tail (container id currently running)

#Write a shell script
[root@10-13-103-211 ~]# docker run -d centos /bin/sh -c "while true; do echo hello world; sleep 2; done"
#Run the print operation all the time

[root@10-13-103-211 ~]# docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
20f358b00913 centos "/bin/sh -c 'while t..." 33 seconds ago Up 32 seconds distracted_pascal

#Display logs
-tf #Display logs
--tail [number] #Display the number of logs

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

**View the process information in the container**

```shell
# docker top container id
[root@10-13-103-211 ~]# docker top 20f358b00913
```

**View the source data of the image**

```shell
# Command docker inspect container id
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
 "HostsPath": "/var/lib/docker/containers/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04/hosts", "LogPath": "/var/lib/docker/containers/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ec abedcb04/20f358b009133283faa64cfb7cd04c2b7e44e7f2d2b58dc96c6096ecabedcb04-json.log",
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
 "ContainerIDFile": "", "LogConfig": {
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
 "PublishAllPorts": false, "ReadonlyRootfs": false,
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
 "CpuQuota": 0, "CpuRealtimePeriod": 0,
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
 "/proc/kcore", "/proc/keys",
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
 "LowerDir": "/var/lib/docker/overlay2/2a30d3912842cb4818438f47bfe92528c09d35475161c2605abdebeb8e77d680-init/di ff:/var/lib/docker/overlay2/bd08aba24bc52fd61c1a6596333b5230b0a055fa3fd763fa16ab452bf013e3bb/diff",
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
 "AttachStdout": false, "AttachStderr": false,"Tty": false,
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
 "org.label-schema.schema-version": "1.0", "org.label-schema.vendor": "CentOS"
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

**Copy files from the container to the host**

```shell
docker cp container id: path in container host path
```

Copying is a manual process. In the future, automatic synchronization can be achieved through the -v volume technology.

## Summary

<img width="780" alt="image-20210618174858353" src="https://user-images.githubusercontent.com/48710834/123402283-2a0cc980-d5da-11eb-9e7e-e7be96089858.png">
<img width="651" alt="image-20210621153656713" src="https://user-images.githubusercontent.com/48710834/123402568-73f5af80-d5da-11eb-9da7-0e1f7ad6fe8c.png">

```shell
attach Attach to a running container #Attach to the specified running image in the current shell
build Build an image from a Dockerfile #Customize the image through Dockerfile
commit Create a new image from a containers changes #Submit the current container as a new image
cp Copy files/folders from a container to a HOSTDIR or to STDOUT #Copy the specified files or directories from the container to the host
create Create a new container #Create a new container, the same as run but without starting the container
diff Inspect changes on a containers filesystem #View docker container changes
events Get real time events from the server #Get container real-time events from the docker service
exec Run a command in a running container #Run a command on an existing container
export Export a containers filesystem as a tar archive #Export the content stream of the container as a tar archive file (corresponding to import)
history Show the history of an image #Show the history of an image formation
images List images #List the current system image
import Import the contents from a tarball to create a filesystem image #Create a new file system image from the contents of the tarball (corresponding to export)
info Display system-wide information #Display system-related information
inspect Return low-level information on a container or image #View container details
kill Kill a running container #kill the specified docker container
load Load an image from a tar archive or STDIN #load an image from a tar package (corresponding to save)
login Register or log in to a Docker registry#register or log in to a docker source server
logout Log out from a Docker registry #exit from the current Docker registry
logs Fetch the logs of a container #output the current container log information
pause Pause all processes within a container#pause container
port List port mappings or a specific mapping for the CONTAINER #view the internal source port of the container corresponding to the mapped port
ps List containers #list container list
pull Pull an image or a repository from a registry #pull the specified image or library image from the docker image source server
push Push an image or a repository to a registry #push the specified image or library image to the docker source server
rename Rename a container #rename the container
restart Restart a running container #restart the running container
rm Remove one or more containers #remove one or more containers
rmi Remove one or more images #Remove one or more images (can only be deleted if no container uses the image, otherwise you need to delete the related container to continue or -f to force deletion)
run Run a command in a new container #Create a new container and run a command
save Save an image(s) to a tar archive#Save an image as a tar archive (corresponding to load)
search Search the Docker Hub for images #Search for images in Dockerhub
start Start one or more stopped containers#Start container
stats Display a live stream of container(s) resource usage statistics #Statistics container resource usage
stop Stop a running container #Stop container
tag Tag an image into a repository #Tag the image in the source
top Display the running processes of a container #View the process information running in the container
unpause Unpause all processes within a container #Unpause the container
version Show the Docker version information#View the container version number
wait Block until a container stops, then print its exit code #Intercept the exit status value when the container stops
```

## Homework Exercise

> Homework 1: Docker installs Nginx

* 1. Search for images search
* 2. Download images docker pull
* 3. Run tests

```shell
[root@10-13-103-211 ~]# docker run -d --name nginx01 -p 3340:80
"docker run" requires at least 1 argument.
See 'docker run --help'.

Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container
[root@10-13-103-211 ~]# docker run -d --name nginx01 -p 3340:80 nginx
02371c0ad7dfb7b305e3ff954c4940d99dfe54f9e37efa182f03305eaf6b19b6
[root@10-13-103-211 ~]# docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
02371c0ad7df nginx "/docker-entrypoint.…" 8 seconds ago Up 8 seconds 0.0.0.0:3340->80/tcp, :::3340->80/tcp nginx01
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

Thinking: Every time you modify the files inside the container, you need to enter the container. Is it possible to provide a mapping path outside the container and modify the files outside?

**-v data volume**

> Homework 2: Install tomcat using docker

```shell
# Official use
$ docker run -it --rm tomcat:9.0

# --rm method, delete after use

# Download and restart
docker pull tomcat:9.0

# Start running
docker run -d -p 3355:8080 --name tomcat01 tomcat

# Enter the container
docker exec -it tomcat01 /bin/bash

```

> Homework 3: Deploy es + kibana

```shell
# es exposes many ports
# es consumes a lot of memory
# es data needs to be placed in a safe directory: mount

# --net somenetwork network configuration
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.13.2
# Linux stuck after startup because elasticsearch consumes a lot of memory
docker stats container id# View cpu status
# Close elasticsearch and increase memory limit, modify configuration file -e environment configuration modification
docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.13.2
# Test whether es is started successfully
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

> Homework 4: Use kibana to connect to elaticsearch

## Visualization

* portainer

Docker graphical interface management tool

```shell
#Download
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
#Access test
#Note to open the firewall access settings first
117.50.101.24:8088 #External network 8088
#Set the account password and select local mount

```

The visualization panel is rarely used.

* Rancher (CI/CD reuse)

# Docker image

## Docker image loading principle

> UFS (Union File System)

<font color=blue>Phenomenon: When downloading an image, the file is downloaded layer by layer, and each layer has its ID, which can be reused for other images to save space; </font>

==UFS: == Union File System ([UnionFS](https://en.wikipedia.org/wiki/UnionFS)) is a layered, lightweight and high-performance file system that supports layer-by-layer superposition of file system modifications as a single commit, and can mount different directories under the same virtual file system (unite several directories into a single virtual filesystem). Union File System is the basis of Docker images. Images can be inherited through layering, and various specific application images can be made based on the basic image (without the parent image). In addition, different Docker containers can share some basic file system layers, and at the same time add their own unique modification layers, which greatly improves storage efficiency.

> Docker image loading principle

![Uploading image-20210621153656713.png…]()

Docker images are all read-only. When the container is started, a new writable layer is loaded on top of the image. This layer is what we usually call the container layer, and everything below the container is called the image layer.

> Layered storage

Because the image contains the complete `root` file system of the operating system, its volume is often huge. Therefore, when Docker was designed, it made full use of the technology of [Union FS](https://en.wikipedia.org/wiki/Union_mount) and designed it as a layered storage architecture. So strictly speaking, the image is not a packaged file like an ISO. The image is just a virtual concept. Its actual embodiment is not composed of a file, but a group of file systems, or in other words, it is composed of multiple layers of file systems.

When the image is built, it will be built layer by layer, and the previous layer is the basis of the next layer. After each layer is built, it will not change again, and any changes on the next layer will only occur in its own layer. For example, the operation of deleting the previous layer of files does not actually delete the previous layer of files, but only marks the file as deleted in the current layer. When the final container is running, although this file will not be seen, it will actually follow the image. Therefore, when building an image, you need to be extra careful. Each layer should only contain what needs to be added to the layer, and any extra things should be cleaned up before the layer is built.

The characteristics of layered storage also make it easier to reuse and customize images. You can even use the previously built image as the base layer, and then further add new layers to customize the content you need and build a new image.

## commit image

```shell
#After starting the container, you can create a new image
docker commit -m="description information" -a="author" container id target image name: [TAG]
```

# Container data volume

## What is a container data volume

==Docker concept==

Packaging applications and environments into an image. If all data is in the container, the data will be lost after deleting the container.

<font color=red> Requirements: Data can be persistent and stored locally! </font>

**There can be a data sharing technology between containers, and the data generated in the Docker container can be synchronized to the local. **

Mount the directory in the container to the host/other container

## Use data volume

> Method 1: Use the command to mount directly -v

```shell
docker run -it -v host directory: directory in container

#Synchronize the /ceshi directory on the host with the /_home directory in the container
[root@10-9-70-238 ~]# docker run -it -v /ceshi:/_home centos /bin/bash

#View the container information, the Mounts item is the mount information
[root@10-9-70-238 ~]# docker inspect 3e28dcdd7f95
"Mounts": [
{ "Type": "bind",
"Source": "/ceshi",
"Destination": "/home",
"Mode": "",
"RW": true,
"Propagation": "rprivate"
}
],
```

* Test:

1. Stop the container;

2. Modify the file on the host;

3. Start the container;

4. The data in the container will still be synchronized;

==Q&A==

Q: Deleting files will also be synchronized. After the container is deleted, will the files at the mapped path be deleted?

A: After testing, the files will still be retained.

## Actual combat: Install MySQL

```shell
# Get the image
[root@10-9-70-238 ceshi]# docker pull mysql

# Run the container and mount the data
# mysql startup command in dockerhub, need to configure the password
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# Start mysql
-d background run
-p port mapping (host port 3310, container port 3306)
-v data volume mount
-e environment configuration (password setting)
--name container name
docker run -d -p 3310:3306 -v /_home/mysql/conf:/etc/mysql/conf.d -v /_home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql

# Use mysql client to connect and test locally
# The client creates a new database, and the host mount directory is updated synchronously

# After the container is deleted, the local mysql database still exists.
```

## Named and anonymous mounts

```shell
# Anonymous mount, host path can be omitted
-v container path
docker run -d -P --name nginx01 -v /etc/nginx nginx
# View all volume volumes
docker volume ls

# Named mount, specify a name for the mounted volume but do not write the host path
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx

[root@10-9-70-238 ~]# docker volume ls
DRIVER VOLUME NAME
local 4bd5255c6926b4e4f192756ecb5ab8100cf4c71263aa0b8227faaf385b9cba5d
local 91e4816758284ae4f54260aba206bcacf7e405f1026936223d1661a73c82b1b5
local juming-nginx
# View volume information
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

<font color=ff8033>Note: All volumes in Docker containers are in this path if no directory is specified: </font> `/var/lib/docker/volumes/xxx/_data`

Expansion

```shell
# Change read and write permissions through -v container path, ro rw
ro # Read-only
rw # Readable and writable
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

# ro means: files under the path can only be operated through the host machine, and cannot be operated inside the container!
```

## Introduction to Dockerfile

Dockerfile: A file used to build a Docker image, similar to a command script

```shell
# Dockerfile file dockerfile1
# Mount directly in the dockerfile content
FROM centos
VOLUME ["volume01", "volume02"]

CMD echo "-----end------"
CMD /bin/bash
# Each command is each layer of the image

```

## Data volume container

```shell
# First start a container, docker01

docker run -it --name docker02 --volumes-from docker01 centos

# Inheritance relationship between containers. After deleting docker01, other containers can still access this file
```

<font color = red> Transmission of configuration information between containers. The life cycle of the data volume container continues until no container is used. However, once persisted locally, local data will not be deleted</font>

# Dockerfile

## Introduction to Dockerfile

Dockerfile: A file used to build a Docker image, similar to a command script

Build steps:

* Write a dockerfile file

* docker build Build an image

* docker run Run the image

* docker push Release the image (DockerHub or Alibaba Cloud Image Repository)

## Dockerfile build process

Basic knowledge:

* Each reserved keyword is capitalized

* The execution order is from top to bottom

* Comment symbol: #

* Each instruction will create a new image layer

## Dockerfile instructions

```shell
FROM #Base image
MAINTAINER #Image maintainer information, name + email
RUN #Commands to be run when building the image
ADD #Add content, copy files, automatically decompress
WORKDIR #Working directory of the image
VOLUME #Set volume, mount host directory
EXPOSE #Set external exposure port
CMD #Specify the command to be run when the container is started. The last command will replace the previous one.
ENTRYPOINT #Specify the command to be run when the container is started. You can append commands.
ONBUILD #When building an inherited Dockerfile, the ONBULID command will be run, which is a trigger instruction.
COPY #Similar to ADD, copy the file to the image
ENV #Set environment variables when building
```

Most images in Docker Hub are created from base images

`FROM scratch`

> Create your own centos
>
>

```shell
# Write the configuration file Dockerfile-centos
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

# Build an image through dockerfile
docker build -f Dockerfile-centos -t mycentos:1.0 . # Note that you need to add a .

# View the image version history
docker history image id

```

## Release the image

> Publish to dockerhub

Steps:

* Login account
* Submit on the server

```shell
# Login account
docker login -u jaychen1996
# Submit image
docker push image ID: tag
```

> Publish to Alibaba Cloud Image Service

* Login to Alibaba Cloud
* Find container image service
* Create namespace
* Create image repository
* Refer to official documents

## Summary

<img width="607" alt="image-20210623141215167" src="https://user-images.githubusercontent.com/48710834/123402699-9982b900-d5da-11eb-9fcb-98696058120b.png">

# Docker network

## Understanding Docker0

```shell
# View ip address
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

>docHow does ker handle container network access

```shell
# Start a tomcat container
[root@10-9-70-238 ~]# docker run -d -P --name tomcat01 tomcat

# View the internal network address ip addr of the container, and get the eth0@if19 ip address, which is allocated by docker
[root@10-9-70-238 ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
18: eth0@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
 link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
 inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
 valid_lft forever preferred_lft forever

# Linux can ping the inside of the container
[root@10-9-70-238 ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.051 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.048 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.048 ms

```

> Principle

* Every time a docker container is started, docker will assign an ip to the docker container. As long as docker is installed, there will be a network card docker0. Bridge mode, using veth-pair technology.

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

 # veth-pair technology adds a 19: veth769efa0@if18 ip address
 # An interface is added to connect the internal network of the container with the host network
```

* tomcat01 and tomcat02 can be pinged

<img width="740" alt="image-20210623155205686" src="https://user-images.githubusercontent.com/48710834/123402760-ab645c00-d5da-11eb-92cf-c78811395d6c.png">

The network interfaces in Docker are all virtual. After the container is deleted, the corresponding bridge disappears.

## Custom network

> View all docker networks

```shell
[root@10-9-70-238 ~]# docker network ls
NETWORK ID NAME DRIVER SCOPE
cc48bf7d6cdb bridge bridge local
bceef51a266d host host local
3afebd3225db none null local
```

Network mode

* bridge: bridge mode, default in docker
* none: no network configuration
* host: share network with host
* container: container network connection

```shell
# Create a custom network
[root@10-9-70-238 ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

# View custom network information
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
 "Ingress": false, "ConfigFrom": {
"Network": ""
},
"ConfigOnly": false,
"Containers": {},
"Options": {},
"Labels": {}
}
]

# Start the container and bind to the custom network
[root@10-9-70-238 ~]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
0af4b6baf1524e5e8d50ade05435a769e70ab6030cd65451c2be56b694d8d011
[root@10-9-70-238 ~]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
e4ab88e485622dbab77da565de81cc1d45d90a114dff23499eab73b8116b8284

# View network information
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
 "Subnet":"192.168.0.0/16",
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

# Test ping connection, you can also ping the name without using --link
[root@10-9-70-238 ~]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.064 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.063 ms

```

==Benefits of using custom network: ==

Different clusters use different networks to ensure the cluster is secure.

## Network connectivity

The tomcat in the container cannot be directly connected to the tomcat in the host, but through the custom network, using connect to create a static routing table, the tomcat in the container can be connected to the custom network.
<img width="675" alt="image-20210624105504134" src="https://user-images.githubusercontent.com/48710834/123402813-b8814b00-d5da-11eb-91f3-c2f55b364603.png">


```shell
[root@10-9-70-238 ~]# docker network --help

Usage: docker network COMMAND

Manage networks

Commands:
 connect Connect a container to a network
 create Create a network
 disconnect Disconnect a container from a network
 inspect Display detailed information on one or more networks
 ls List networks
 prune Remove all unused networks
 rm Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.

# You can use connect to connect a container to a custom host network
[root@10-9-70-238 ~]# docker network connect mynet tomcat01

# View custom network information
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
 }, "e4ab88e485622dbab77da565de81cc1d45d90a114dff23499eab73b8116b8284": {
"Name": "tomcat-net-02",
"EndpointID": "78a3f259f29abd99f1ea9ed20c3120e9d1cbc595b53a1cbd4761fc3f90142335",
"MacAddress": "02:42:c0:a8:00:03",
"IPv4Address": "192.168.0.3/16",
"IPv6Address": ""
}
},

# 01 is connected, 02 is not connected
[root@10-9-70-238 ~]# docker exec -it tomcat01 ping tomcat-net-01
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.111 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.097 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.083 ms
```



## Deploy Redis cluster

```shell
# Create a custom subnet
[root@10-9-70-238 ~]# docker network create redis --subnet 172.38.0.0/16

# Create six Redis configurations through the script
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

# Start six redis containers
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; \

docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9 -alpine3.11 redis-server /etc/redis/redis.conf

#Deploy cluster
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

# Enter the cluster
/data # redis-cli -c

# View cluster information
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

# Check the cluster nodes, three masters and three slaves
127.0.0.1:6379> cluster nodes
7ada335c8811fa5482836cff83946d10c407bd34 172.38.0.13:6379@16379 master - 0 1624514251548 3 connected 10923-16383
898c6bba011e26bb5d64853dc49f0cb292c1eec3 172.38.0.14:6379@16379 slave 7ada335c8811fa5482836cff83946d10c407bd34 0 1624514251146 4 connected
2758f5660ea5dbddf56250f8c88da5dd120f86ab 172.38.0.11:6379@16379 myself,master - 0 1624514252000 1 connected 0-5460
c4b740de0c168f6f7d31a4bee8107195a44746d8 172.38.0.15:6379@16379 slave 2758f5660ea5dbddf56250f8c88da5dd120f86ab 0 1624514253152 5 connected
17a8b4969af8dc458c7a6740f977dc8178999522 172.38.0.16:6379@16379 slave 8d00e899016c4b6cd10e8dfa98c562c774c1284a 0 1624514251000 6 connected
8d00e899016c4b6cd10e8dfa98c562c774c1284a 172.38.0.12:6379@16379 master - 0 1624514252150 2 connected 5461-10922

# Write a key-value pair on redis-3
127.0.0.1:6379> set a b
-> Redirected to slot [15495] located at 172.38.0.13:6379
OK
# Close redis-3
[root@10-9-70-238 /]# docker stop redis-3
redis-3
[root@10-9-70-238 /]# docker exec -it redis-1 /bin/sh
/data # redis-cli -c

# Test data retrieval, data is automatically transferred to redis-4, redis-4 becomes master
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

## Introduction

> Official introduction

* Tool for defining and running multiple containers
* YAML file configuration file

Compose is a tool for defining and running multiple containers i-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

* Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.

Purpose: batch container orchestration

> Other notes

Compose is an official open source project of Docker and needs to be installed by yourself;

A `docker-compose.yml` looks like this:

```yaml
version: "3.9" # optional since v1.27.0
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

`docker compose up` can start all the services in the configuration file.

## Install

```shell
# Official website documentation https://docs.docker.com/compose/install/
# download
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Authorization
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
 WORKDIR/code
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
- Set environment variables used by the `flask` ​​command.
  -Install gcc and other dependencies
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

*Web service

The `web` service uses an image that’s built from the `Dockerfile` in the current directory. It then binds the container and the host machine to the exposed port, `5000`. This example service uses the default port for the Flask web server, `5000`.

* Redis service

The `redis` service uses a public [Redis](https://registry.hub.docker.com/_/redis/) image pulled from the Docker Hub registry.



6. Build and run your app with Compose

From your project directory, start up your application by running `docker-compose up`.

> Process analysis

1. Create a network
2. Execute Docker-compose yaml
3. Start the service

Creating composetest_web_1 ... done

Creating composetest_redis_1 ... done

![image-20210625140241502](https://user-images.githubusercontent.com/48710834/123402889-cafb8480-d5da-11eb-93bb-e64cb8328945.png)

==Benefits: ==Write yaml configuration files through docker-compose. All services can be started through compose.

## yaml rules (commonly used in k8s)

Docker-compose.yaml core

```shell
# Three layers in total

version: ''
services:
Service 1:web
#Service configuration
images
build
network
Service 2:redis
···
# Other configurations
volumes
configs
network
```

## Open source project

> Build a blog-wordpress

Official website address: https://docs.docker.com/samples/wordpress/

1. Configure yaml file

2. `docker-compose up -d` can start directly successfully

# Dokcer Swarm

First create 4 hosts.

## Raft protocol

Dual master and dual slave: If one node is down, are the other nodes available?

Raft protocol: It can only be used if most nodes are alive. The cluster must have at least 3 nodes! (If there are only two nodes, after one node goes down, it is impossible to determine whether it is offline or the other node is offline);