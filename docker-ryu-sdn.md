# Docker 部署基于 Ryu 的 SDN 环境

---

作者：李呈

---

【编者按】Ryu是由日本NTT公司负责设计研发的一款开源SDN控制器，采用Apache License开源协议标准。Ryu基于Python语言实现，使用者可以用Python语言在其上实现自己的应用。Ryu目前支持OpenFlow V1.0、V1.2、V1.3，同时支持OpenStack上的部署应用。作者通过Docker和Ryu部署了一个非常简单的SDN网络。

作者简介：李呈，北邮在读研究生，主要研究方向为SDN。个人博客： [http://www.muzixing.com/](http://www.muzixing.com/)

---

## 基本概念

- **镜像**（Image）：镜像是一个只读模板。用户上传制作好的镜像供其他人下载使用。用户可以基于镜像去创建Container。 

- **容器**（Container）：容器可以理解为一个隔离起来的Linux环境，用于运行应用，Namespace可以帮助你理解。 

- **仓库**（Repository）：如果你会使用Git/Github的话，不难理解，就是用于存放镜像的场所。

本文的实验环境是Ubuntu14.04-amd64。非常需要注意的一点是，目前 **Docker 只支持64位机器**。Ubuntu14.04安装方式有两种：1）通过系统自带包安装和2）通过Docker源安装。推荐第二种方式，能安装比较新的版本。

## Docker安装

```
sudo apt-get install apt-transport-https 
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9 
sudo bash -c "echo deb https://get.docker.io/ubuntudocker main > /etc/apt/sources.list.d/docker.list" 
sudo apt-get update 
sudo apt-get install lxc-docker
```

## 获取镜像

首先，推荐到 [Docker](http://www.muzixing.com/hubhub.docke.com) 注册帐号，这样可以像使用 Git/Github 那样使用 Docker/DockerHub 。注册和登录可通过如下命令完成：

```
docker login
```

注册之后，可以通过如下命令进行搜索，如搜索 ryu :

```
docker search ryu
```

可以从搜索结果中的 Star 来确定资源的好坏，从而找到合适的 images ，如 muzixing/ryu 。然后使用如下命令，将其拉到本地：

```
docker pull  muzixing/ryu
```

下载完成后，可以通过如下命令查看已存在的 images

```
docker images
```

![alt](http://resource.docker.cn/muzi-images.jpg)
 
## 创建容器

创建容器使用示例如下， -t=tty, -i=interactive, -d=debug, -p=port, --name 可以用于命名container。 其他的命令可以通过 `--help` 来查看。

```
docker run -i -t --name <name> muzixing/ryu:SDN  /bin/bash
```

如果你需要对端口映射，或者网络配置方面的设置，还需要仔细去查看手册。举例如下：

```
docker run -i -t -p  <ip>:<host port>:<container port>  --name <name> muzixing/ryu:SDN  /bin/bash
```

以上命令创建并运行了一个名叫 muzixing/ryu:SDN 的 container ，并且将容器内部的端口 port 映射到主机的某个 port ，完成了端口映射，允许外网访问容器。这是容器与外界通信的方式之一。如果希望永久绑定到某个固定的 IP 地址，可以在 Docker 配置文件 /etc/default/docker 中指定 `DOCKER_OPTS="--ip=IP_ADDRESS"` ，之后重启 Docker 服务即可生效。设置网络访问的参数默认是 `--icc=ture` ，如果 `--icc=false` ，则禁止网络访问。

查看容器：

```
docker ps [-opt]
```

`-a` 为全部容器。

查看打印信息可以通过：

```
dokcer logs <name>
```

暂停容器：

```
docker stop <name>
```

开启暂停的容器：

```
docker start <name>
```

重启容器：

```
docker restart <name>
```

有时候我们开启了容器，但是没有开窗口，在后台运行，可以通过一下命令进入容器：

```
docker attach <name>
```

## 部署SDN控制器RYU

首先获取镜像：

```
docker pull  muzixing/ryu
```

然后创建容器，并将容器的6633端口绑定到主机的6633端口。

```
docker run -i -t -p  0.0.0.0:6633:6633  --name ryu3.15 muzixing/ryu:SDN  /bin/bash
```

进入容器之后，运行 ryu 。

在另一个能ping通运行容器主机的机器上运行mininet，如下图：

![alt](http://resource.docker.cn/ryu-1.jpg)
 
从上图可以看出控制器IP是172.16.192.128。这个IP地址的主机网卡信息如下：

![alt](http://resource.docker.cn/ryu-2.jpg)
 
从图上可以看出，与mininet通信的是主机（实际情况下会是某台服务器）eth0的地址。但是从下面的图的容器信息中，可以看出运行的RYU地址是172.17.0.5。为什么可以通信呢？

![alt](http://resource.docker.cn/ryu-3.jpg)
 
*容器信息*

因为做了端口映射，将主机的所有接口的6633端口的地址都转发到容器172.17.0.5的6633端口，从而完成数据通信。其实现的原理是：Docker在启动之后，会创建一个docker0的网桥，从上图可以看到，然后还会创建veth pair。其中一端挂载在网桥上，如上图的vethba5f9f3，另一端是容器的网卡eth0，此案例中是172.17.0.5的网卡。其实这相当与一个link。Docker网络通信原理图如下：

![alt](http://resource.docker.cn/ryu-4.jpg)
 
*Docker网络通信原理图* 

在运行容器的主机上使用iptables命令查看NAT规则： 

![alt](http://resource.docker.cn/ryu-5.jpg)
 
*iptables查看NAT*

同理mininet，或者其他的应用程序也可以使用容器部署，不再赘述，读者可自行尝试。

## 上传镜像

首先需要将部署了应用的容器导出为tar文件。可以使用 `docker export container > file` 命令。举例如下：

```
docker export ryu3.15 > ryu.tar
```

然后使用 `docker import` 命令将其导入为镜像：

```
cat ryu.tar | sudo docker import - muzixing/ryu:sdn
```

以上命令为读取 `ryu.tar` 将其导入成 `muzixing/ryu:sdn` 的 image 。完成之后可通过 `docker images` 查看。

确保无误之后，可将其推送到 DockerHub 。

```
docker push muzixing/ryu
```

读者也可以尝试更好的自动创建方式。

## 网络配置

我们完全可以将 Docker 理解成一个独立的主机，可以对其网络进行配置，如配置 DNS 、 iptables 等。可以通过启动时配置，也可以通过修改文件的方式配置。

- -b BRIDGE or --bridge=BRIDGE —— 指定容器挂载的网桥
- --bip=CIDR —— 定制 docker0 的掩码
- -H SOCKET... or --host=SOCKET... —— Docker 服务端接收命令的通道
- --icc=true|false —— 是否支持容器之间进行通信
- --ip-forward=true|false —— 请看下文容器之间的通信
- --iptables=true|false —— 禁止 Docker 添加 iptables 规则
- --mtu=BYTES —— 容器网络中的 MTU

文件配置则如同正常的主机配置，进入到/etc/目录下，修改制定文件即可。同样的，Dokcer可以配置网络链接的网桥，可以不选择docker0网桥，而选择其他网桥，如使用brctl创建的网桥，或者使用OpenvSwitch创建的网桥，具体操作不再赘述。

## 后语

工欲善其事，必先利其器。 Docker 可以允许我们更灵活地使用资源，并且可以很方便地迁移环境。比如以后需要安装 Ryu 的同学就不需要再去关注，为什么 six 版本不够？为什么 gcc 报错这些问题了。只需要有一台 64 位的机器，然后安装 Docker ，理论上是不会有错的。然后将镜像下载下来，创建并运行容易，就可以得到 Ryu 控制器运行的环境。同理 Nginx 、 Tornado 和 MySQL 等软件也可以直接获取，而不需要自己安装配置环境。这大大加快了生产环境的部署，也显著提高了资源的利用率，个人认为将在未来对虚拟机产生强烈的冲击。

原文链接： [Docker部署SDN环境](http://www.muzixing.com/pages/2014/12/03/dockerbu-shu-sdnhuan-jing.html)（责编：周小璐）


