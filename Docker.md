# Docker基本组成

![image-20220518133223129](Picture\Docker基本组成.png)

* 镜像（image）：

  docker镜像就好比是一个模板，可以通过这个模板来创建容器服务，（tomcat镜像===》run===》tomcat01容器

  通过这个镜像可以创建多个容器（最终服务运行或者项目运行在这个容器中）

* 容器（container）

  Docker利用容器技术，独立运行一个或者一个组应用，通过镜像来创建的

  启动，停止，删除等基本命令操作

  目前可以把这个容器理解为一个建议的Linux系统

* 仓库（repository）

  仓库就是存放镜像的地方

  仓库分为共有仓库和私有仓库

  DocketHub

# Docker安装

## 环境查看

```
[mnsx@localhost ~]$ uname -r
3.10.0-957.el7.x86_64
```

```
[mnsx@localhost ~]$ cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

## 卸载旧的docker

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 下载所需的安装包

```
yum install -y yum-utils
```

## 设置镜像的仓库

```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

## yum软件包索引

```
yum makecache fast
```

## 安装docker相关的docker-ce社区 ee企业版

```
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 启动Docker

```
systemctl start docker
```

## Hello world

```
docker run hello-world
```

## 查看下载的镜像

```
docker images
```

## 卸载Docker

* 卸载当前的镜像

  ```
  yum remove docker-ce docker-ce-cli containerd.io
  ```

* 删除资源

  ```
  rm -rf /var/lib/docker
  ```

# 底层原理

![image-20220518191113889](Picture\Run运行流程.png)

**Docker是怎么工作的**

Docker是一个Client-Server结构的系统，Docker的守护进行再主机上。通过Socket从客户端访问

SockerServer接收到Docker-Client的指令，就会执行这个命令

![image-20220518150714808](Picture\底层原理.png)

**Docker为什么比VM快**

1. Docker比VM有更少的抽象层
2. docker利用的是宿主主机的内核，vm需要是Guest OS

# Docker常用命令

## 帮助命令

```
docker version       # 显示docker的版本信息
docker info          # 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help    # 帮助命令

# 解释
REPOSITORY 镜像仓库源
TAG 	   镜像的标签
IMAGE ID   镜像的id
CREATED	   镜像的创建时间
SIZE  	   镜像的大小

# 可选项
-a, --all   列出所有镜像
-q, --quiet 只显示镜像的id
```

## 镜像命令

* 展示镜像

```
[root@localhost mnsx]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   7 months ago   13.3kB

```

* 搜索镜像

```
[root@localhost mnsx]# docker search mysql
NAME                           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                          MySQL is a widely used, open-source relation…   12589     [OK]       
mariadb                        MariaDB Server is a high performing open sou…   4839      [OK]       
percona                        Percona Server is a fork of the MySQL relati…   576       [OK]       
phpmyadmin                     phpMyAdmin - A web interface for MySQL and M…   540       [OK]       
bitnami/mysql                  Bitnami MySQL Docker Image                      71                   [OK]

# 可选项
--filter=stars=3000 #搜索出来的就是Start大于3000的
[root@localhost mnsx]# docker search mysql --filter=stars=5000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   12589     [OK]   
```

* 下载镜像

```
[root@localhost mnsx]# docker pull mysql 
Using default tag: latest # 如果不写tag，默认就是latest
latest: Pulling from library/mysql
72a69066d2fe: Pull complete # 分层下载，docker image的核心 联合文件系统
93619dbc5b36: Pull complete 
99da31dd6142: Pull complete 
626033c43d70: Pull complete 
37d5d7efb64e: Pull complete 
ac563158d721: Pull complete 
d2ba16033dad: Pull complete 
688ba7d5c01a: Pull complete 
00e060b6d11d: Pull complete 
1c04857f594f: Pull complete 
4d7cfa90e6ea: Pull complete 
e0431212d27d: Pull complete 
Digest: sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709 # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest # 真实地址

dcoker pull mysql
docker pull docker.io/library/mysql:latest

指定版本下载
[root@localhost mnsx]# docker pull mysql:5.7
5.7: Pulling from library/mysql
72a69066d2fe: Already exists 
93619dbc5b36: Already exists 
99da31dd6142: Already exists 
626033c43d70: Already exists 
37d5d7efb64e: Already exists 
ac563158d721: Already exists 
d2ba16033dad: Already exists 
0ceb82207cd7: Pull complete 
37f2405cae96: Pull complete 
e2482e017e53: Pull complete 
70deed891d42: Pull complete 
Digest: sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

[root@localhost mnsx]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         5.7       c20987f18b13   4 months ago   448MB
```

* 删除镜像

```
[root@localhost mnsx]# docker rmi -f c20987f18b13
Untagged: mysql:5.7
Untagged: mysql@sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
Deleted: sha256:c20987f18b130f9d144c9828df630417e2a9523148930dc3963e9d0dab302a76
Deleted: sha256:6567396b065ee734fb2dbb80c8923324a778426dfd01969f091f1ab2d52c7989
Deleted: sha256:0910f12649d514b471f1583a16f672ab67e3d29d9833a15dc2df50dd5536e40f
Deleted: sha256:6682af2fb40555c448b84711c7302d0f86fc716bbe9c7dc7dbd739ef9d757150
Deleted: sha256:5c062c3ac20f576d24454e74781511a5f96739f289edaadf2de934d06e910b92

[root@localhost mnsx]# docker rmi -f $(docker images -aq) 
Untagged: mysql:latest
Untagged: mysql@sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709
Deleted: sha256:3218b38490cec8d31976a40b92e09d61377359eab878db49f025e5d464367f3b
Deleted: sha256:aa81ca46575069829fe1b3c654d9e8feb43b4373932159fe2cad1ac13524a2f5
Deleted: sha256:0558823b9fbe967ea6d7174999be3cc9250b3423036370dc1a6888168cbd224d
Deleted: sha256:a46013db1d31231a0e1bac7eeda5ad4786dea0b1773927b45f92ea352a6d7ff9
Deleted: sha256:af161a47bb22852e9e3caf39f1dcd590b64bb8fae54315f9c2e7dc35b025e4e3
Deleted: sha256:feff1495e6982a7e91edc59b96ea74fd80e03674d92c7ec8a502b417268822ff
Deleted: sha256:8805862fcb6ef9deb32d4218e9e6377f35fb351a8be7abafdf1da358b2b287ba
Deleted: sha256:872d2f24c4c64a6795e86958fde075a273c35c82815f0a5025cce41edfef50c7
Deleted: sha256:6fdb3143b79e1be7181d32748dd9d4a845056dfe16ee4c827410e0edef5ad3da
Deleted: sha256:b0527c827c82a8f8f37f706fcb86c420819bb7d707a8de7b664b9ca491c96838
Deleted: sha256:75147f61f29796d6528486d8b1f9fb5d122709ea35620f8ffcea0e0ad2ab0cd0
Deleted: sha256:2938c71ddf01643685879bf182b626f0a53b1356138ef73c40496182e84548aa
Deleted: sha256:ad6b69b549193f81b039a1d478bc896f6e460c77c1849a4374ab95f9a3d2cea2
Untagged: hello-world:latest
Untagged: hello-world@sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
```

## 容器命令

说明：我们有了镜像才可以创建容器，Linux，下载一个CentOS镜像来测试学习

`docker pull centos`

新建容器并启动

```shell
docker run [可选参数] image

# 参数说明
--name="name" 容器名字
-d 后台模式启动
-it 使用交互方式运行，进入容器查看内容
-p 指定容器的端口
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口
	-p 容器端口
	容器端口
-P 随机指定端口

[root@localhost mnsx]# docker run -it centos /bin/bash
[root@0715a3ad2266 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

列出所有的运行的容器

```
# docker ps 命令
		# 列出当前正在运行的容器
-a		# 列出当前正在运行的容器 + 带出历史运行过的容器
-n=?	# 显示最近创建的容器
-q      # 只显示容器的编号

[root@localhost mnsx]# docker ps -aqn=1
0715a3ad2266
```

退出容器

```
exit 退出并停止容器
Ctrl + P + Q 退出不停止容器
```

删除容器

```
docker rm 容器id 删除指定容器
docker rm -f $(docker ps -aq) 删除所有的容器
docker ps -a -q | xargs docker rm 删除所有的容器
```

启动和停止容器

```
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id
```

# 常用其他命令

* 后台启动容器

  ```
  [root@localhost mnsx]# docker run -d centos
  18f555426a8b27e40dc577f8d2097c32ae5739ae55572372120b1068634fe7f7
  
  # 使用docker run -d 发现容器使用后台使用时停止了
  # 容器使用后台运行，就必须要有一个前台进程，docker发小没有应用，就会自动停止
  # 容器启动后，发现自己没有提供服务，就会自动停止
  ```

* 查看日志

  ```
  docker logs -tf -tail x # 显示指定的条数
  docker logs -tf # 显示全部的日志
  ```

* 查看容器中进程信息

  ```
  [root@localhost mnsx]# docker top 5dac8ef79091
  UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
  root                8178                8158                0                   19:04               pts/0               00:00:00            /bin/bash
  ```

* 查看镜像中的原数据

  ```
  [root@localhost mnsx]# docker inspect 5dac8ef79091
  [
      {
          "Id": "5dac8ef79091b6ee8af94fbd1db5cc6e502ccd67ee62189323313d64c4ff02be",
          "Created": "2022-05-19T02:04:24.796076529Z",
          "Path": "/bin/bash",
          "Args": [],
          "State": {
              "Status": "running",
              "Running": true,
              "Paused": false,
              "Restarting": false,
              "OOMKilled": false,
              "Dead": false,
              "Pid": 8178,
              "ExitCode": 0,
              "Error": "",
              "StartedAt": "2022-05-19T02:04:25.558636102Z",
              "FinishedAt": "0001-01-01T00:00:00Z"
          },
          "Image": "sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6",
          "ResolvConfPath": "/var/lib/docker/containers/5dac8ef79091b6ee8af94fbd1db5cc6e502ccd67ee62189323313d64c4ff02be/resolv.conf",
          "HostnamePath": "/var/lib/docker/containers/5dac8ef79091b6ee8af94fbd1db5cc6e502ccd67ee62189323313d64c4ff02be/hostname",
          "HostsPath": "/var/lib/docker/containers/5dac8ef79091b6ee8af94fbd1db5cc6e502ccd67ee62189323313d64c4ff02be/hosts",
          "LogPath": "/var/lib/docker/containers/5dac8ef79091b6ee8af94fbd1db5cc6e502ccd67ee62189323313d64c4ff02be/5dac8ef79091b6ee8af94fbd1db5cc6e502ccd67ee62189323313d64c4ff02be-json.log",
          "Name": "/gallant_saha",
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
                  "LowerDir": "/var/lib/docker/overlay2/c9047fdb6c796581c6f9d87d3062ffb3284948821608f7db827e87d7ca8b0447-init/diff:/var/lib/docker/overlay2/ac9546d423f8a903340ac21000c0a45f4a39a60dc53db90923023240121b5337/diff",
                  "MergedDir": "/var/lib/docker/overlay2/c9047fdb6c796581c6f9d87d3062ffb3284948821608f7db827e87d7ca8b0447/merged",
                  "UpperDir": "/var/lib/docker/overlay2/c9047fdb6c796581c6f9d87d3062ffb3284948821608f7db827e87d7ca8b0447/diff",
                  "WorkDir": "/var/lib/docker/overlay2/c9047fdb6c796581c6f9d87d3062ffb3284948821608f7db827e87d7ca8b0447/work"
              },
              "Name": "overlay2"
          },
          "Mounts": [],
          "Config": {
              "Hostname": "5dac8ef79091",
              "Domainname": "",
              "User": "",
              "AttachStdin": true,
              "AttachStdout": true,
              "AttachStderr": true,
              "Tty": true,
              "OpenStdin": true,
              "StdinOnce": true,
              "Env": [
                  "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
              ],
              "Cmd": [
                  "/bin/bash"
              ],
              "Image": "centos",
              "Volumes": null,
              "WorkingDir": "",
              "Entrypoint": null,
              "OnBuild": null,
              "Labels": {
                  "org.label-schema.build-date": "20210915",
                  "org.label-schema.license": "GPLv2",
                  "org.label-schema.name": "CentOS Base Image",
                  "org.label-schema.schema-version": "1.0",
                  "org.label-schema.vendor": "CentOS"
              }
          },
          "NetworkSettings": {
              "Bridge": "",
              "SandboxID": "10572409ecfe6a7cfb4c0ec9f6c3ffdf7e4305d477830b72467384a5af9256c6",
              "HairpinMode": false,
              "LinkLocalIPv6Address": "",
              "LinkLocalIPv6PrefixLen": 0,
              "Ports": {},
              "SandboxKey": "/var/run/docker/netns/10572409ecfe",
              "SecondaryIPAddresses": null,
              "SecondaryIPv6Addresses": null,
              "EndpointID": "3ebef2ad5a0b316f21a90c2535ca03eb064f019a74550a0e4475b840c9dc0331",
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
                      "NetworkID": "19a630229dc2cef5c77b9d3d1e5d38835b7b68e8c7482f666304a67bf989629c",
                      "EndpointID": "3ebef2ad5a0b316f21a90c2535ca03eb064f019a74550a0e4475b840c9dc0331",
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

* 进入当前正在运行的容器

  ```shell
  # docker extc -it 容器id bashshell
  [root@localhost mnsx]# docker exec -it 5dac8ef79091 /bin/bash
  [root@5dac8ef79091 /]# ps -ef
  UID         PID   PPID  C STIME TTY          TIME CMD
  root          1      0  0 02:04 pts/0    00:00:00 /bin/bash
  root         15      0  0 02:22 pts/1    00:00:00 /bin/bash
  root         29     15  0 02:22 pts/1    00:00:00 ps -ef
  [root@5dac8ef79091 /]# sleep 1
  
  # docker attach 
  [root@localhost mnsx]# docker attach 5dac8ef79091
  [root@5dac8ef79091 /]#
  
  # docker exec # 进入容器侯开启一个新的终端，可以在里面操作（常用）
  # docker attach # 进入容器正在执行的终端，不会启动新的进程
  ```

* 从容器内拷贝文件到主机上

  ```shell
  [root@localhost mnsx]# docker cp bade029da9ba:/home /home
  
  ```

# 命令总结

![image-20220519105200141](Picture\docker命令.png)

# 可视化

* Portainer
* Rancher

## portainer简介

Docker图形化界面管理工具！提供一个后台面板供我们使用

```shell
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock --privileged=true portainer/portainer

[root@localhost mnsx]# docker run -d -p 8088:9000 \
> --restart=always -v /var/run/docker.sock --privileged=true portainer/portainer
Unable to find image 'portainer/portainer:latest' locally
latest: Pulling from portainer/portainer
94cfa856b2b1: Pull complete 
49d59ee0881a: Pull complete 
a2300fd28637: Pull complete 
Digest: sha256:fb45b43738646048a0a0cc74fcee2865b69efde857e710126084ee5de9be0f3f
Status: Downloaded newer image for portainer/portainer:latest
c788ad062c3b5758a3f592c5020c9d1b2b630f5b3e57f6d52c89e5ef0b2801da
```

![image-20220519110506500](Picture\portainer.png)

![image-20220519110554916](Picture\第二界面.png)

# Docker镜像讲解

## 镜像简介

镜像是一种轻量级、可执行的执行软件包，用来打包软件

# 容器数据卷

## 容器数据卷的简介

将应用和运行环境打包成一个镜像

如果容器被删除，那么数据也会丢失，希望数据能够持久化

容器之间可以有一个数据共享的技术

Docker容器中产生的数据， 同步到本地

这就是卷技术，目录的挂载，将我们容器内的目录，挂载到外部linux上

## 使用数据卷

使用命令挂载

```shell
docker run -it -v 主机目录:容器目录 -p 主机端口:容器端口

[root@localhost mnsx]# docker run -it -v /home/test:/home centos

-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境变量
--name 容器名字
```

运行反向操作，在主机目录中修改文件，同时容器中的文件也会被修改

## 具名和匿名挂载

* 匿名挂载

  ```shell
  docker run -d -p --name nginx01 -v /etc/nginx nginx
  
  [root@localhost mnsx]# docker run -d -P --name nginx01 -v /etc/nginx nginx
  1c5719cf19ad37b9063302537c292e76c948b7a8e8f8b51578095c34a1fd5067
  
  [root@localhost mnsx]# docker volume list
  DRIVER    VOLUME NAME
  local     0c3d04b5f08b165d8db0e6b1aac6758ee6a525e1262b98fc5ddc5a7fb9f63c14
  local     0180275b606e1793a3a418898fe8d0e35f30f058888aeda4c781076c0a3bcfab
  local     dabb41bf721e3aff4b90a8ee0d6772d881c6528794fb3b1f841a8e3d5ca4f609
  ```

* 具名挂载

  ```shell
  [root@localhost mnsx]# docker run -d -P --name nginx01 -v test-nginx:/etc/nginx nginx
  d1bb1d316b9021b240648b1703d0fe50a82e32e807060ffd3da4217f046fb509
  
  [root@localhost mnsx]# docker volume ls
  DRIVER    VOLUME NAME
  local     0c3d04b5f08b165d8db0e6b1aac6758ee6a525e1262b98fc5ddc5a7fb9f63c14
  local     0180275b606e1793a3a418898fe8d0e35f30f058888aeda4c781076c0a3bcfab
  local     dabb41bf721e3aff4b90a8ee0d6772d881c6528794fb3b1f841a8e3d5ca4f609
  local     test-nginx
  ```

所有docker容器内的卷，没有指定目录的情况下都是在/var/lib/docker/volumes/xxx/\_data中

**扩展**

通过 -v 容器内路径:ro / rw 改变权限

* ro——read only：只读
* rw——read write：可读可写

ro只要看到ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作的

## 数据卷——DockerFile

DockerFile 就是用来构建docker镜像的构建文件

通过这个脚本可以生成镜像，镜像是一层一层的，脚本一个个的命令，每个命令都是一层

```shell
# 创建一个dockerfile文件，名字可以随机
# 文件内容
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "---end---"
CMD /bin/bash

[root@localhost mnsx]# docker build -f /home/mnsx/dockerfile -t mnsx/centos:1.0 .
Sending build context to Docker daemon  3.939MB
Step 1/4 : FROM centos
 ---> 5d0da3dc9764
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in b49caaa44663
Removing intermediate container b49caaa44663
 ---> 53a753ec818c
Step 3/4 : CMD echo "---end---"
 ---> Running in cb978ea05099
Removing intermediate container cb978ea05099
 ---> fb7901f94caa
Step 4/4 : CMD /bin/bash
 ---> Running in 1c11b7b26948
Removing intermediate container 1c11b7b26948
 ---> ad062652c4f3
Successfully built ad062652c4f3
Successfully tagged mnsx/centos:1.0
```

## 数据卷容器

```shell
docker run -it --name docker02 --volumns-from docker01 mnsx/centos:1.0
```

结论：

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止

但是一旦你持久化到了本地，这个时候，本地的数据是不会删除的

# DockerFile

dockerFile用来构建docker镜像文件！命令参数脚本！

构建步骤——

1. 编写一个dockersfile文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

很多官方镜像都是基础包，很多功能都没有，我们通常会自己搭建自己的镜像

也可以自己制作镜像

## DockerFile构建过程

1. 每个保留关键字都必须是大写字母
2. 执行从上到下顺序执行
3. \#标识注释
4. 每一个指令都会创建提交一个新的容器，并提交

dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件非常简单

Docker镜像逐渐成为企业交付的标准，必须掌握

dockerfile：构建文件，定义了一切的步骤，源代码

dockerimages：通过dockerfile构建生成的镜像，最终发布和运行的产品

docker容器：容器就是镜像运行起来提供服务器

## DockerFile指令

```shell
FROM # 基础镜像，一切从这里开始构建
MAINTAINER # 镜像创建者——姓名+邮箱
RUN # 镜像运行时需要执行的命令
ADD # 步骤，添加所需要的的内容
WORKDIR # 镜像的工作目录
VOLUME # 设置容器卷，挂载的为止
EXPOSE # 暴露端口
CMD # 指定容器启动时运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT # 指定容器启动时运行的命令
ONBUILD # 当构建一个被继承dockerfile这个时候就会运行ONBUILD的指令
COPY # 类似ADD命令，将文件拷贝到镜像中
ENV # 构建时设置环境变量
```

1. 编写一个dockersfile文件

   ```shell
   [root@localhost mnsx]# cat dockerfile 
   FROM centos
   
   MAINTAINER Mnsx_x<xx1527030652@gmail.com>
   
   ENV MYPATH /usr/local
   
   WORKDIR $MYPATH
   
   RUN yum -y install vim
   RUM yum -y install net-tools
   
   EXPOSE 80
   
   CMD echo $MYPATH
   CMD echo "---end---"
   CMD /bin/bash
   ```

2. docker build 构建成为一个镜像

   ```shell
   [root@localhost mnsx]# docker build -f /home/mnsx/dockerfile -t mnsx/centos:1.0 .
   
   Successfully built 5ac178b62d2e
   Successfully tagged mnsx/centos:1.0
   ```

3. docker run 运行镜像

   ```shell
   [root@localhost mnsx]# docker run -it mnsx/centos:1.0
   
   [root@localhost mnsx]# docker history 5ac178b62d2e
   IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
   5ac178b62d2e   10 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
   f8586f0bcf0e   10 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
   6f80a35fac22   10 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
   6012ea44f178   10 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
   ad4783ee63a3   10 minutes ago   /bin/sh -c yum -y install net-tools             171MB     
   aaa6dce5016f   11 minutes ago   /bin/sh -c yum -y install vim                   226MB     
   55ea16a60ca9   11 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
   942af03a24c3   11 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
   9f04bbd4fa19   11 minutes ago   /bin/sh -c #(nop)  MAINTAINER Mnsx_x<xx15270…   0B        
   eeb6ee3f44bd   8 months ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
   <missing>      8 months ago     /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
   <missing>      8 months ago     /bin/sh -c #(nop) ADD file:b3ebbe8bd304723d4…   204MB     
   ```

### CMD和ENTRYPOINT的区别

Dockerfile中很多命令都十分相似，我们需要了解他们的区别

## 发布自己的镜像

1. 通过自己的账号在服务器上登录

   ```shell
   [root@localhost mnsx]# docker login -u mnsx2002 
   Password: 
   WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
   Configure a credential helper to remove this warning. See
   https://docs.docker.com/engine/reference/commandline/login/#credentials-store
   
   Login Succeeded
   ```

2. 上传自己的镜像

   ```shell
   [root@localhost mnsx]# docker push mnsx/centos:1.0
   The push refers to repository [docker.io/mnsx/centos]
   cb14003e5496: Preparing 
   5094f650e302: Preparing 
   174f56854903: Preparing 
   ```

# Docker网路

启动一个docker容器，就会给docker容器分配一个ip，只要安装了docker就会生成一个docker0网卡

--link

# Docker Compose

Docker Compose可以基于Compose帮我们快速的部署分布式应用，而无需手动一个个创建和运行容器

## 初始Dockers Compose

Compose文件是一个文本文件，通过指令定义集群中的每个容器如何运行。格式如下：

```json
version: "3.8"
 services:
  mysql:
    image: mysql:5.7.25
    environment:
     MYSQL_ROOT_PASSWORD: 123 
    volumes:
     - "/tmp/mysql/data:/var/lib/mysql"
     - "/tmp/mysql/conf/hmy.cnf:/etc/mysql/conf.d/hmy.cnf"
  web:
    build: .
    ports:
     - "8090:8090"

```

上面的Compose文件就描述一个项目，其中包含两个容器：

- mysql：一个基于`mysql:5.7.25`镜像构建的容器，并且挂载了两个目录
- web：一个基于`docker build`临时构建的镜像容器，映射端口时8090

DockerCompose的详细语法参考官网：https://docs.docker.com/compose/compose-file/

其实DockerCompose文件可以看做是将多个docker run命令写到一个文件，只是语法稍有差异。

## DockersCompose安装

### 下载

Linux下需要通过命令下载：

```sh
# 安装
curl -L https://github.com/docker/compose/releases/download/1.23.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

### 修改文件权限

修改文件权限：

```sh
# 修改权限
chmod +x /usr/local/bin/docker-compose
```

### Base自动补全命令

```sh
# 补全命令
curl -L https://raw.githubusercontent.com/docker/compose/1.29.1/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

如果这里出现错误，需要修改自己的hosts文件：

```sh
echo "199.232.68.133 raw.githubusercontent.com" >> /etc/hosts
```

## 案例

```yaml
version: "3.2"

services:
  nacos:
    image: nacos/nacos-server
    environment:
      MODE: standalone
    ports:
      - "8848:8848"
  mysql:
    image: mysql:5.7.25
    environment:
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "$PWD/mysql/data:/var/lib/mysql"
      - "$PWD/mysql/conf:/etc/mysql/conf.d/"
  userservice:
    build: ./user-service
  orderservice:
    build: ./order-service
  gateway:
    build: ./gateway
    ports:
      - "10010:10010"
```
