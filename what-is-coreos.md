# CoreOS 实战：CoreOS 及管理工具介绍

---

##### 作者：[桂阳](http://www.infoq.com/cn/author/%E6%A1%82%E9%98%B3)

---

【编者按】CoreOS是一个基于Docker的轻量级容器化Linux发行版，专为大型数据中心而设计，旨在通过轻量的系统架构和灵活的应用程序部署能力简化数据中心的维护成本和复杂度。CoreOS作为Docker生态圈中的重要一员，日益得到各大云服务商的重视，目前已经完成了A轮融资，发展风头正劲。InfoQ希望《CoreOS实战》系列文章能够帮助读者了解CoreOS以及相关的使用方法。如果说Docker是下一代的虚拟机，那CoreOS就应该是下一代的服务器Linux，InfoQ愿和您一起探索这个新生事物。另外，欢迎加入InfoQ Docker技术交流群，QQ群号：365601355。

## 1. 概述

随着 Docker 的走红，CoreOS 作为一个基于 Docker 的轻量级容器化 Linux 发行版日益得到大家的重视，目前所有的主流云服务商都提供了对 CoreOS 的支持。CoreOS 是新时代下的Linux 发行版，它有哪些独特的魅力了？本篇作为《CoreOS实战》的第一部分，将向大家简要介绍 CoreOS 以及CoreOS 相关的管理工具，试图向您揭开CoreOS背后神秘的面纱。

## 2. CoreOS之禅

云计算新星 Docker 正在以火箭般的速度发展，与它相关的生态圈也渐入佳境，CoreOS 就是其中之一。CoreOS 是一个全新的、面向数据中心设计的 Linux 操作系统，在2014年7月发布了首个稳定版本，目前已经完成了800万美元的A轮融资。 CoreOS 专门针对大型数据中心而设计，旨在以轻量的系统架构和灵活的应用程序部署能力简化数据中心的维护成本和复杂度。现在CoreOS 已经推出了付费产品。通过付费，用户可以使用可视化工具管理自己的 CoreOS 集群。

与其他历史悠久、使用广泛的 Linux 操作系统相比，CoreOS 拥有下面几个优点。

![alt](http://resource.docker.cn/coreos.png)

首先，CoreOS 没有提供包管理工具，而是通过容器化 (containerized) 的运算环境向应用程序提供运算资源。应用程序之间共享系统内核和资源，但是彼此之间又互不可见。这样就意味着应用程序将不会再被直接安装到操作系统中，而是通过 Docker 运行在容器中。这种方式使得操作系统、应用程序及运行环境之间的耦合度大大降低。相对于传统的部署方式而言，在 CoreOS 集群中部署应用程序更加灵活便捷，应用程序运行环境之间的干扰更少，而且操作系统自身的维护也更加容易。

其次， CoreOS 采用双系统分区 (dual root partition) 设计。两个分区分别被设置成主动模式和被动模式并在系统运行期间各司其职。主动分区负责系统运行，被动分区负责系统升级。一旦新版本的操作系统被发布，一个完整的系统文件将被下载至被动分区，并在系统下一次重启时从新版本分区启动，原来的被动分区将切换为主动分区，而之前的主动分区则被切换为被动分区，两个分区扮演的角色将相互对调。同时在系统运行期间系统分区被设置成只读状态，这样也确保了 CoreOS 的安全性。CoreOS 的升级过程在默认条件下将自动完成，并且通过 cgroup 对升级过程中使用到的网络和磁盘资源进行限制，将系统升级所带来的影响降至最低。

另外，CoreOS 使用 Systemd 取代 SysV 作为系统和服务的管理工具。与 SysV 相比，Systemd 不但可以更好的追踪系统进程，而且也具备优秀的并行化处理能力，加之按需启动等特点，并结合 Docker 的快速启动能力，在 CoreOS 集群中大规模部署 Docker Containers 与使用其他操作系统相比在性能上的优势将更加明显。Systemd 的另一个特点是引入了 “target” 的概念，每个 target 应用于一个特定的服务，并且可以通过继承一个已有的 target 扩展额外的功能，这样使得操作系统对系统上运行的服务拥有更好的控制力。

通过对系统结构的重新设计，CoreOS 剔除了任何不必要的软件和服务。在一定程度上减轻了维护一个服务器集群的复杂度，帮助用户从繁琐的系统及软件维护工作中解脱出来。虽然CoreOS 最初源自于Google ChromeOS，但是从一开始就决定了 CoreOS 更加适合应用于一个集群环境而不是一个传统的服务器操作系统。

## 3. CoreOS相关工具

除了操作系统之外，CoreOS 团队和其他团队还提供了若干工具帮助用户管理 CoreOS 集群以及部署 Docker containers。

### 3.1. etcd

![alt](http://resource.docker.cn/etcd.png)

在CoreOS 集群中处于骨架地位的是 etcd。 etcd 是一个分布式 key/value 存储服务，CoreOS 集群中的程序和服务可以通过 etcd 共享信息或做服务发现 。etcd 基于非常著名的 raft 一致性算法：通过选举形式在服务器之中选举 Lead 来同步数据，并以此确保集群之内信息始终一致和可用。etcd 以默认的形式安装于每个 CoreOS 系统之中。在默认的配置下，etcd 使用系统中的两个端口：4001和7001，其中4001提供给外部应用程序以HTTP+Json的形式读写数据，而7001则用作在每个 etcd 之间进行数据同步。用户更可以通过配置 CA Cert让 etcd 以 HTTPS 的方式读写及同步数据，进一步确保数据信息的安全性。

### 3.2. fleet

fleet 是一个通过 Systemd对CoreOS 集群中进行控制和管理的工具。fleet 与 Systemd 之间通过 D-Bus API 进行交互，每个 fleet agent 之间通过 etcd 服务来注册和同步数据。fleet 提供的功能非常丰富，包括查看集群中服务器的状态、启动或终止 Docker container、读取日志内容等。更为重要的是 fleet 可以确保集群中的服务一直处于可用状态。当出现某个通过 fleet 创建的服务在集群中不可用时，如由于某台主机因为硬件或网络故障从集群中脱离时，原本运行在这台服务器中的一系列服务将通过fleet 被重新分配到其他可用服务器中。虽然当前 fleet 还处于非常早期的状态，但是其管理 CoreOS 集群的能力是非常有效的，并且仍然有很大的扩展空间，目前已提供简单的 API 接口供用户集成。

### 3.3. Kubernetes

Kuberenetes 是由 Google 开源的一个适用于集群的 Docker containers 管理工具。用户可以将一组 containers 以 “POD” 形式通过 Kubernetes 部署到集群之中。与 fleet 更加侧重 CoreOS 集群的管理不同，Kubernetes 生来就是一个 Containers 的管理工具。Kubernetes 以 “POD” 为单位管理一系列彼此联系的 Containers，这些 Containers 被部署在同一台物理主机中、拥有同样地网络地址并共享存储配额。

### 3.4. flannel (rudder)

flannel (rudder) 是 CoreOS 团队针对 Kubernetes 设计的一个覆盖网络 (overlay network) 工具，其目的在于帮助每一个使用 Kuberentes 的 CoreOS 主机拥有一个完整的子网。Kubernetes 会为每一个 POD 分配一个独立的 IP 地址，这样便于同一个 POD 中的 Containers 彼此连接，而之前的 CoreOS 并不具备这种能力。为了解决这一问题，flannel 通过在集群中创建一个覆盖网格网络 (overlay mesh network) 为主机设定一个子网。

## 4. 下篇介绍

在下一篇中，笔者将为大家展示如何建立一个 CoreOS 集群并通过 Kubernetes 管理其中的 Docker Containers。

感谢[郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE)对本文的策划和审校。
