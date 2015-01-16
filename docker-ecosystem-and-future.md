# Docker的生态系统和未来

---

作者：喻勇

---

> 编者按： Docker 风暴席卷全球，以轻量级容器为核心的生态系统发展迅猛。作者结合自己在开源领域的一些经验，梳理了 Docker 的技术发展史，分层剖析了现有的生态系统，并展望了未来可能的走向和突破。


---

再次纠正概念： Docker 不是轻量级容器。它由管理轻量级容器的引擎、客户端和 AUFS 文件系统三部分组成。轻量级容器（Lightweight Container）在 UNIX/Linux 领域经历了十多年的发展，并在最近5年突飞猛进。

## 轻量级容器技术发展历程

在分析Docker的生态系统之前，我们首先回顾轻量级容器技术的发展路线图。

- 2000年， **BSD Jail** ：Jail以多种方式改进和增强了BSD类操作系统中用于进程隔离的chroot环境。Jail不仅对文件系统访问实现隔离，还把用户、BSD的网络子系统及一些其他系统资源在内核中进行隔离。

- 2005年， **Solaris Containers** ： Solaris Containers 的实现包括两部分： Solaris Zones 和 System Resource Controls 。 Zones 实现了命名空间隔离和安全访问控制。在 non-global zone 中的进程既不能看见该 zone 之外的进程，也不能与之通信。 System Resource Controls 实现了资源管理功能。

- 2005年， **OpenVZ** ： OpenVZ 是 GPLv2 协议下的开源软件，是基于 Linux 平台的操作系统级服务器资源隔离解决方案。 OpenVZ 采用 SWsoft 的 Virutozzo 软件产品的内核， Virutozzo 是 SWsoft 公司提供的商业解决方案。

以上这些都是容器技术的先驱， Container 真正开始普及，由以下几个标志性事件推动。

- 2007年，从2.6.4版本开始， cgroups 正式进入 Linux 内核。
- 2008年，LXC 0.10出现，简化了容器的创建和管理。
- 2011年，Linux 开发者就容器技术的统一规范达成共识。
- 2012年，Cloud Foundry 选择使用 WARDEN Container 来承载 PaaS 应用。
- 2013年，Google发布开源容器管理工具 lmctfy(Let Me Contain That For You) 。

2013年是容器和周边技术高歌猛进的一年，这其中以 Docker 的流行为代表，以下两家公司和他们的产品具有标志意义。

- 2013年， **Docker version 0.10** ： Docker 是 PaaS 提供商 dotCloud（最近已经正式改名为 Docker Inc. ）开源的一个基于 LXC 的高级容器引擎，源代码托管在 GitHub 上，基于 Go 语言并遵从 Apache 2.0 协议开源。 Docker 的出现极大简化了容器的创建和管理，分层式的 AUFS 实现了 Docker 镜像。

- 2013年， **CoreOS** ：这家在硅谷某个车库里成立的创业公司发布了专门为大规模服务器部署定制的 Linux 精简系统，目的是为运行以轻量级容器为载体的应用提供一个高度优化的底层系统。
2014年，大量围绕 Docker 和 CoreOS 的创业公司、新近开源的软件项目、大型企业和互联网公司的加入，使轻量级容器技术的浪潮更上一层楼。

正如定义所言， Docker 是“ Container Engine ”，它是一个把 cgroup 、 namespace 等容器底层技术抽象的一个引擎，为用户提供了创建和管理容器的便捷界面（包括命令行和API）。

概念明晰了，我们先从技术栈的维度来看 Docker 和它的生态系统，把从 Linux 到 Docker 做四个层面的分层。

- **Linux 操作系统**。完整的 Linux 内核，履行操作的使命：管理硬件，调度任务，提供用户界面和服务等。
- **容器的内核实现**。这主要包括 Linux 内核中的 cgroup 、 namespace 等，它们为容器（用户进程）的资源隔离性提供了内核层面的保障。
- **轻量级容器的基础工具**。通过 LXC 这样的工具可以完成容器创建、启动等基本操作，但使用 LXC 需要熟知容器内核实现原理。这对于普通用户来说有一定难度，并且 LXC 在不同 Linux 发行版不一致。
- **容器管理引擎**。 Docker 是这一层的主角。Docker 由 Docker engine 和 Docker client 组成。 Docker engine 将神秘的 cgroup 、复杂的 LXC 统统隐藏起来，使用简单的命令即可完成容器的运行和管理。它的另一个独特之处在于 AUFS 的运用， Copy on write 模式的分层文件系统使容器的镜像可以像搭积木一样灵活创建和修改，并在网络上实现增量分发。 Docker client ，特别是它的 API ，为在 Docker 之上的生态系统发展提供了可能性。

Docker的出现和标准化，为以轻量级容器为核心的生态系统提供了爆发式增长的机会。我们从以下几个角度来看Docker的生态系统。

## Docker和容器宿主

前文提到的Docker Inc.和CoreOS已经赚足眼球，投资者接踵而至，大规模融资此起彼伏。企业级厂商如红帽、Ubuntu等不甘寂寞，纷纷亮明旗帜，选择站队。

6月在旧金山举行的DockerCon 2014展示了Docker对未来的雄心壮志。在Docker engine逐渐稳定并标准化的背景下，Docker的未来目标是为互联网基础架构制定新的标准。最近开源的libcontainer、libchan和libswarm三个项目，吹响了这场战役的冲锋号。

- 在新版本Docker engine中，由Go语言开发的libcontainer库已取代LXC。我认为，它更大的目的是反向定义容器的实现标准，将底层实现（也许可以完全不用cgroup甚至Linux）都抽象化到libcontainer的接口。
- libchan类库，以标准接口的方式解决容器的互联互通，实现跨平台，能更好支持分布式系统和并发编程。
- libswarm是另一个很简单的类库，但它将实质性地推动互联网应用架构的创新。它抽象了应用部署和集群管理的细节，为应用程序赋予了跨云平台和互联网级弹性。

CoreOS的口号“A new way to think about servers”，这句话阐明了他们对改造互联网服务器的目标。CoreOS通过最小化的定制版Linux系统为容器运行提供载体。我曾一度认为CoreOS的发展方向是与硬件更紧密结合，推出基于ARM的版本，甚至集成进入服务器固件。

然而CoreOS以实际行动证明了我的判断错误：2014年8月14日，传来了CoreOS收购Quay.IO并推出CoreOS Enterprise Registry服务的新闻。

显然，CoreOS并不满足于服务器层的工作，其目标定位在为企业用户提供完整的容器技术服务栈，提供管理大型容器集群的整体解决方案。在这个类别中生存的是标准定义者，它们是整个Docker生态系统的基础。

## 镜像存储和容器托管

这包括容器的镜像存储和 CaaS（Container as a Service）类的容器运行托管，有代表性的公司是StackDock、Orchard、Tutum、Quay.IO、Baremetal.IO 等。

这几家几乎全都是创业公司，他们围绕轻量级容器的整个生命周期来设计自己的产品，有的聚焦容器镜像描述文件（Dockerfile）向导化生成和构建过程的优化（如StackDock），有的提供包括SSD在内的高性能托管环境（如StackDock和Tutum），有的在监控和弹性扩展方面做足文章（如Tutum），也有像Baremetal.IO这样针对企业级整体解决方案的公司。

容器的镜像存储和运行托管是Docker生态体系中非常接近最终用户的一层。这个类别中的公司也许并没有高深莫测的技术，也不是标准的定义者，但通过它们与细分市场中客户的长期沟通合作，积累了大量Docker商用化的经验和实践。这一层最近有两个并购案例：Docker收购Orchard/Fig团队和CoreOS收购Quay.IO。是不是有点像大鱼吃小鱼？我们来仔细看看这两家被“吃掉”的公司。

Orchard Laboratories（好邪乎的名字，其实只有两名员工）开发并维护一个名为Fig的开源工具。Fig被称为“by far the easiest way to orchestrate the deployment of multi-container applications”，也被冠以“the perfect Docker companion for developers”。简单地说，Fig以Docker为基础，用容器贯穿整个软件开发流程，快速实现隔离开发环境。Fig让用户编写一个简单的fig.yml文件列出应用需要的所有Docker容器，以及它们是如何连接在一起的。编写好fig.yml以后，只需要加上-d参数，应用就能开始上线运行了。这意味着从此开发、测试、运行环境得到统一，容器成为软件发布的新载体。前文提到过Docker的目标是为互联网基础架构制定新的标准，Fig的加入使面向开发者和开发流程这个环节得到极大增强。

Quay.IO，这个团队为用户提供私有的Docker镜像仓库（Image Repository）托管服务。通过这次收购，CoreOS增强了CoreOS Enterprise Registry服务。Quay.IO也只有两名员工。

8月20日，传来了 tutum.co 获得260万美元前期投资的新闻，他们是这个领域的大公司（有七名员工），作为CaaS平台提供商，目前有1500个开发者使用其服务。

## 基于Docker的微PaaS

镜像存储（静态）和容器托管（动态）都是以容器为单位的。下面我们将要讲述以应用为单位，以容器为底层技术实现的微PaaS。

这几年随着Microsoft Azure、Cloud Foundry的普及（我有幸分别参与了这两个产品在中国市场的早期推广工作），PaaS的概念已经深入人心。传统意义上PaaS实例一般都与一个特定的IaaS平台绑定，提供部署接口、负载平衡、服务绑定等，然而Docker世界中产生的微PaaS，在此基础上进一步创新。这个领域比较有代表性的是Flynn和Deis.IO，它们都是开源项目。

Flynn分为Layer 0和Layer 1两层。Layer 0主要做底层硬件和云平台的抽象，分布式配置、任务调度、服务发现等基础工作，它为上层的容器运行环境提供了一个抽象的资源平台。Flynn可以快速部署在AWS上，今后也可扩展到其他公有云和私有云。Layer 1主要服务于应用，是PaaS功能的具体实现层，它提供了基本的管理API和客户端，实现了Git Receiver、Heroku Buildpacks、Routing和Datastore等PaaS核心功能。Layer 1本身和它所管理的应用，都以容器为载体。

Deis.IO，它的一个亮点是用CoreOS承担底层资源管理的任务。在部署Deis PaaS环境时，首先安装的Controller会创建一个CoreOS系统，然后在其之上以容器的方式运行Deis的所有组件。对CoreOS的支持是一个非常聪明的选择，目前CoreOS已可以运行在多个公有云平台、虚拟机和物理机环境下，这为Deis提供了与生俱来的跨云平台能力。

Flynn和Deis的共同特点，是对复杂和大规模分布式应用的原生支持。Heroku创始人Adam Wiggins曾发布著名的“十二要素应用宣言（The Twelve-Factor App）”，这个宣言定义了以服务方式和通过互联网交付的软件应该遵循的十二个要素。Flynn和Deis都是十二要素的忠实拥护者，它们的微PaaS平台与Heroku有极好的兼容性。

微PaaS创业公司层出不穷，竞争十分激烈，但也许走到最后的只是少数。在这一轮容器技术热潮中，微PaaS正在影响软件开发和运维流程，改变软件的交付方式，把十二要素类互联网应用架构标准化。

## Orchestration 、Management 和 Moni-t­oring

围绕Docker API做Web UI的门槛相对较低，受到了大家的追捧，这一类主要有DockerUI、Shipyard、maDocker等。它们无一例外都调用Docker API和其他类库，把对容器的管理和监控呈现在Web页面中，这在某种程度上降低了企业网管对这些新技术的恐惧。

这一领域有三个不得不提的高富帅项目：Google Kubernetes、Cloud Foundry的BOSH和Diego。

Kubernetes是构建在Docker之上的容器集群管理系统，Google在2014年6月将这个项目开源。它可以为用户提供跨平台的处理能力，不但能够在Google的基础架构中运行，同时可以访问其他的云计算服务器，如AWS，甚至是私有云。

这个系统一经开源，就得到了IBM、红帽、微软、Docker、Mesosphere、CoreOS和SaltStack等厂商的支持：微软将确保Kubernetes能够在其Azure云中作为基于Linux的虚拟机系统容器并正常运作；红帽则将其引入了自己的云产品；IBM的计划是为Kubernetes与Docker贡献代码；CoreOS将在其操作系统发行版中为Kubernetes提供支持；SaltStack正努力简化Kubernetes运行在其他环境下的部署流程；而Mesosphere则打算将这项技术加入到自己的Mesos同名开源项目当中。Google一呼百应的大将之风展露无遗。

Cloud Foundry的BOSH是部署和运维工具，它通过类似操作系统驱动程序的CPI（Cloud Provider Interface）来实现对多种异构云平台的支持和抽象，以近乎优雅的方式管理VM模板【注：在Cloud Foundry术语中称为干细胞（stemcell）】、软件发布（release）和部署配置脚本文件。最近BOSH推出了一个试验性质的项目BOSH Release for Docker。

Cloud Foundry在它的DEA（Droplet Execution Agent）中使用基于Warden的容器技术来做PaaS的应用隔离。最新的Diego（Go语言版DEA）项目目标是让Cloud Foundry在跨运行时环境方面更具有扩展性，这些运行时环境就包括Docker，也可能会原生支持Windows Server。

## 网络层的增强和解决方案

容器之间如何互联互通？Docker引擎中的内联网络能否满足企业级别网络的需求？当容器像今天的虚拟机一样在企业环境大规模部署时，复杂的网络需求如网络配置管理、安全监控、流量QoS、网络隔离等一定会出现。

在虚拟化的世界里，这些需求催生了庞大的网络虚拟化（SDN）产业，在容器的环境中，是否有同样的挑战和机会？在这个领域中，目前受关注较多的是Skydock和VNS3开源项目，但整体上还都处在萌芽起步阶段。

## 谁是容器技术的最终用户

上面列出了很多公司和产品，谁将是容器技术的最终用户呢？我认为会在以下几个领域取得突破。

- **互联网公司**

互联网公司的开发运维环境复杂，应用多采用分布式架构，后台使用服务的种类繁多，这些都是Docker最擅长解决的问题。据统计，国内外已有一定数量的互联网公司将Docker集成到内部的开发测试流程，并以Docker为载体发布应用。GROUPON曾在社区分享他们使用Docker与Jenkins结合做持续集成的案例，国内例如七牛等新兴互联网公司也开始应用Docker。

- **传统ISV**

在整个SDLC（Systems Development Life Cycle）环节中引入Docker，特别是增强以容器为核心的持续集成和持续交付，最终将容器作为软件向云平台交付的实体。这方面目前并没有产品化的整体解决方案，国外如Shippable，国内如Coding等创业公司在向这个方向努力。

- **移动开发**

这是软件开发最热门的领域，围绕社交、移动、游戏的MBaaS（Mobile Backend as a Service）已有不少成型的产品。Docker，微PaaS如何与移动应用开发相结合，是另一个值得关注的领域。

除此以外，Docker生态系统在大数据等领域也发展了若干开源工具和项目，这里不一一赘述。

以上是Docker生态系统的一个快照，这个领域的发展可谓一日千里，标准化、开源开放、创业公司、大企业支持、风险投资等特征形成了一个滚雪球的模式，将助推这一轮技术革新到更高的一个台阶。

---

作者介绍

喻勇：VMware中国研发中心高级经理

2012年初加入VMware中国研发中心，担任中国区开发者关系团队高级经理，负责Cloud Foundry、大数据、软件定义存储等前沿产品在开发者、技术社区及战略合作伙伴中的推广。主导了Cloud Foundry中国社区和生态系统的建设工作。在系统架构、云计算和开发应用平台等方面有超过10年的经验，在加入VMware之前担任微软技术布道师，曾负责微软战略ISV的Windows Azure PaaS云计算平台搭建工作，对企业软件架构在云平台的设计、迁移、安全及云计算商业模式有较深入的研究。

---
本文原载于 《程序员》 杂志，我们在得到授权后将其转载。您也可以阅读原文： [Docker的生态系统和未来](http://code.csdn.net/news/2821832) 。