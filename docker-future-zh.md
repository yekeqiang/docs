# Docker : 现在和未来 

---

作者：Chris Swan ，译者：张晓鹏

---

## Docker – 迄今为止的故事

Docker 是一种 Linux 容器工具集，它是为“构建（build）、交付（ship）和运行（run）”分布式应用而设计的。作为 dotCloud 公司的开源项目，其首发版本的时间是2013年的3月份。该项目很快就受到欢迎，这也使得 dotCloud 公司将其品牌改为 Docker （并最终[将其原有的PaaS业务出售](http://blog.dotcloud.com/dotcloud-paas-joins-cloudcontrol)而专注在Docker上）。Docker 1.0 在2014年6月发布，而且延续了之前每月发布一个版本的节奏。

其1.0版本标志着 Docker 公司认为 Docker 平台已经足够成熟，并可以被应用到产品中（公司及其合作伙伴们还提供了一些需要付费的支持选项）。每月的版本更新显示出该项目仍在快速发展，比如增加新的特性，解决发现的问题。这个项目成功地将“交付”和“运行”解耦，这样源自任意 Docker 版本的镜像都可以和其它任意不同版本一起工作（前向和后向均可兼容），这就为 Docker 应用提供了稳定的基础，以应对快速的变化。

Docker发展成最受欢迎的开源项目可能会被人看作是一种炒作，但其实这个结果还是有坚实的基础来支撑的。Docker吸引了业界众多知名大牌厂家的支持，其中包括亚马逊（Amazon）、 Canonical 、 CenturyLink 、谷歌（Google）、 IBM 、 Microsoft 、 New Relic 、Pivotal 、 Red Hat 和 VMware ，这使得只要在有 Linux 的地方， Docker 就几乎随处可用。除了这些大厂家，许多初创企业也围绕着 Docker 来发展，或是将它们的发展方向和 Docker 更好地结合起来。所有这些合作伙伴（或大或小）驱动着核心项目和周边生态系统的快速发展。

## Docker 技术的简要综述

Docker 利用了一些 Linux 核心工具，比如 [cGruops](https://www.kernel.org/doc/Documentation/cgroups/cgroups.txt) 、命名空间和 [SELinux](http://selinuxproject.org/page/Main_Page) 来支撑容器之间的隔离。起初Docker只是 [LXC](https://linuxcontainers.org/) 容器管理子系统的前端，但它在0.9版本中引入了 [libcontainer](http://blog.docker.com/2014/03/docker-0-9-introducing-execution-drivers-and-libcontainer/) ，这个原生的 Go 语言库提供了用户空间和内核之间的接口。

Docker 容器是基于联合文件系统（union file system）的，比如 [AUFS](http://aufs.sourceforge.net/aufs.html) ，利用它可以跨多个容器来共享一些组件，如操作系统镜像和安装的库文件。这种分层的方式也被 [Dockerfile](https://docs.docker.com/reference/builder/) DevOps 工具充分利用，这可以缓存那些已经成功完成的操作。这就省掉了那些安装操作系统和应用程序依赖文件的时间，大幅度加速了测试周期。另外，在容器之间共享库还能减少内存的占用。

Docker 容器是从镜像开始的，镜像可以是本地创建的、本地缓存的，或者是从注册库中下载的。 Docker 公司运营着 [DockerHub 公有注册库](https://registry.hub.docker.com/)，上面有官方的数据仓库，是为不同的操作系统、中间件和数据库而创建的。组织和个人可以在 Docker Hub 上发布镜像的公有库，也可以将其注册成私有库。由于上传的镜像文件可以包含任何东西，所以 Docker Hub 提供了一种自动构建工具（之前被称为“可信的构建”），镜像构建于一个 Dockerfile ，它作为镜像内容的清单。

## 容器和虚拟机的对比

Docker容器要比虚拟机有效率的多，这是因为它们可以共享内核和相关的库。同样的原因，容器所占用的内存也要比虚拟机少得多，虽然虚拟机利用了RAM的过度承诺技术（RAM overcommitment）。容器也减少了对存储的占用，因为部署的容器会共享镜像的底层。IBM的Boden Russel已经做了一个 [基准测试](http://bodenr.blogspot.co.uk/2014/05/kvm-and-docker-lxc-benchmarking-with.html?m=1)（ benchmarking ）来对比两者的不同。

容器也表现出比虚拟机更低的系统负载，所以同样的应用，在容器中相比在虚拟机中，性能通常会相当或者更好。IBM的研究者团队发布了一个 [虚拟机和Linux容器性能对比](http://domino.research.ibm.com/library/cyberdig.nsf/papers/0929052195DD819C85257D2300681E7B/%24File/rc25482.pdf) 的报告可以参考。

容器只是在隔离特性上要比虚拟机差，虚拟机可以使用ring-1特权的[硬件隔离](https://en.wikipedia.org/wiki/X86_virtualization#Hardware-assisted_virtualization)技术，如Intel的VT-d and VT-x。这种隔离技术可以防止虚拟机破出（breaking out）和彼此交互。容器没有任何形式的硬件隔离，这使得它容易受到漏洞的利用。从Shocker（可对Docker进行概念攻击）的验证来看，Docker 1.0之前的版本都是很脆弱的。尽管利用Shocker，Docker在1.0版本中修复了一些特定的问题，但Docker的CTO Solomon Hykes仍然[表示](https://news.ycombinator.com/item?id=7910117)：“当我们感觉可以轻松地宣称Docker开箱即（out-of-the-box）可以安全容纳非受信的uid0程序（译者注： root和超级用户权限）时，我们一定会明言相告”。Hykes的话从另一方面承认仍然存在一些其他的漏洞和相关的风险，在容器变得可靠之前还有很多工作要做。

对于许多用户案例，在容器和虚拟机二者之间选择其一是种错误的二分法。Docker可以在虚拟机中运行地很好，这可以让它应用在已有的虚拟化框架中，如私有云和公有云。同样也有可能在容器中运行虚拟机，这有点像谷歌在它的云平台中使用容器的方式。只要IaaS得到广泛应用，并可按需提供虚拟机服务，那么就有理由期待未来数年容器和虚拟机的应用可以并存。还有一种可能，即将容器管理和虚拟化技术进行融合以提供一种两全其美的方法：所以硬件信任锚微虚拟化实现支撑libcontainer能够与Docker的工具链和生态系统的前端进行集成，而使用不同的提供了更好隔离性的后端。微虚拟技术（类似于 Bromium 的 [vSentry](http://www.bromium.com/products/vsentry.html) 和 VMware 的 [Project Fargo](http://cto.vmware.com/vmware-docker-better-together/) ）已经应用到桌面环境中为应用提供基于硬件的隔离，所以相同的方法可以使用到libcontainer上，作为Linux核心容器机制的替代技术。

## “容器化”的应用

几乎所有的Linux应用都可以运行在Docker容器里，并且对编程语言或架构的选择没有任何的限制。实际上仅有的限制在于从操作系统的角度来看容器被允许做什么。即使如此，也可以通过在特权模式下运行容器来降低限制，以大幅度地减少受到的控制（与此对应的是装载到容器里的应用风险增加，并可能会导致对主机操作系统的损坏。）

容器从镜像开始，反过来镜像也可以从运行的容器中得到。从本质上说有两种方法可以把应用置入到容器中-手动和Dockerfile。

### 手动构建

手动构建从启动一个基础操作系统的容器开始，然后通过交互式终端，用所选Linux相关的包管理器安装应用程序及其依赖项（dependencies）。Zef Hemel在他的文章“[使用Linux容器以支持便捷的应用部署](http://www.infoq.com/articles/docker-containers)”中提供了这个过程的细致描述。一旦应用完成安装，新的容器就可以推送到注册库（比如Docker Hub）中或者被导出成一个tar文件。

### Dockerfile

Dockerfile是对Docker容器创建过程进行脚本化的系统。每个Dockerfile详细说明了开始的基础镜像，以及随后一系列在容器中运行的命令和添加到容器中的文件。Dockerfile也可以说明容器对外的端口，启动时的工作目录和缺省执行的命令。用Dockerfile构建的容器可以象手动构建那样被推送到注册库中或者导出成tar文件。Dockerfile也可以应用到Docker Hub的自动构建系统中，即在Docker公司的控制下，在系统中根据Dockerfile从头构建镜像，并且这个镜像的源对于使用它的任何人都是可见的。

### 一个进程？

手动构建还是使用Dockerfile来构建镜像，考虑的关键在于容器刚启动时只能执行一个单进程。如果容器的服务目的比较单一，比如只运行一个应用服务器，那么运行单个进程就没什么问题（一些争论说容器本应该只包括一个进程）。对于那些希望在容器中运行多个进程的情况，[管理进程](http://docs.docker.com/articles/using_supervisord/)(supervisor process)需要先启动，这样它可以接着孵化出其他期望的进程。此时容器中没有初始的系统，所以任何事都要依赖systemd，不修改新兴系统或类似系统都无法工作。

## 容器和微服务

全面描述使用微服务架构的哲理和益处已经超出了本文的范围（ [InfoQ eMag: Microservices](http://www.infoq.com/minibooks/emag-microservices) 中有全面的阐述）。容器仍然是一种方便的方法来绑定和部署微服务的实例。

虽然目前微服务大规模的部署实例还是在（大量）虚拟机上，但容器提供了一种小规模部署的机会。容器具有共享的内存和针对操作系统的磁盘占用、通用代码库，这也意味着可以非常有效地一起部署多个版本的服务。

## 连接的容器

一些小的应用适合放在单个容器中，但许多情况下应用需要扩展到多个容器。Docker成功催生了一系列新的应用合成工具、编制工具以及平台即服务实现。绝大多数努力的后面，是希望能简化从一组相互连接的容器来创建应用的过程。很多工具在扩展、容错、性能管理和部署资产的版本控制方面也提供了帮助。

### 连接性

Docker的网络能力相当原始，容器中的服务对相同主机的其它容器是可见的，并且Docker可以映射端口到主机操作系统，使得服务在网络中也是可见的。[Libchan](https://github.com/docker/libchan)是Docker官方赞助的连接方法，它提供了Go语言的网络服务库，类似于[channels](https://gobyexample.com/channels)。在libchan找到自己应用的道路之前，还是有空间留给第三方程序来提供一些补充性的网络服务。例如，Flocker采用了基于代理的方法，这使得服务（连同底层的存储）可以在主机间进行迁移。

### 合成

Docker有个原生的机制来连接容器，它所依赖的元数据可以被传送到相关的容器中，这些元数据被用做环境变量和主机入口。类似 [Fig](http://www.fig.sh/)和 [geard](http://openshift.github.io/geard/) 这样的应用合成工具，可以在单文件中表达这种依赖关系图，这样多个容器就可以互相配合成为一个系统。 CenturyLink 的 [Pannamax](http://panamax.io/) 合成工具在底层采用了和 Fig 、 geard 相似的方法，但加入了基于 Web 的用户接口，并且直接和 GitHub 进行了集成，这样就可以分享合成后的应用了。

### 编配

编配系统包括 [Decking](http://decking.io/) 、New Relic 公司的 [Centurion](https://github.com/newrelic/centurion) 和谷歌公司的 [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) ，它们的目标都是帮助实现容器的部署和它的生命周期管理。也有很多基于 [Apache Mesos](http://mesos.apache.org/) 系统（特别是它为应用长期运行设计的 [Marathon](https://github.com/mesosphere/marathon) 框架）的商业化实现（比如 [Mesosphere](https://mesosphere.io/2013/09/26/docker-on-mesos/) ），它们可以和 Docker 一起使用。通过提供在应用需求和底层架构间的抽象（例如，需求表达为CPU核数和内存大小），编制工具提供了两者之间的解耦，这种设计简化了应用开发和数据中心的运维。还有各种各样的编制系统，这主要是因为之前许多内部系统工具冒出来了，它们之前开发出来是用于管理容器大规模部署的。例如Kubernetes就是基于谷歌的 [Omega](http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/41684.pdf) 系统，而 Omega 系统是用来管理整个谷歌云环境中的容器。

合成工具和编配工具功能上会有部分的重合，所以使用时它们彼此可以作为补充。例如Fig可以用来描述容器功能上如何交互，同时 Kubernetes pods（译者注：pods可以被看成一个容器组）可以用来提供相关的监测和扩展功能。

### 平台即服务

有很多原生于 Docker 的 PaaS 实现，例如 [Deis](http://deis.io/) 和 [Flynn](https://flynn.io/) ，已经体现出Linux容器在支持开发灵活性上的强大优势（而不是那些自以为是给定的一套语言和框架）。其它的云平台如CloudFoundry、OpenShift和Apcera Continuum，已经采用集成基于Docker相关功能到现有系统中的技术路线，这样基于Docker镜像（或者是创建它们的Dockerfiles）的应用在部署和管理的同时，仍然可以使用之前系统支持的语言和框架。

## 所有的云

由于Docker可以运行在任何有合理数据内核的Linux虚拟机上，所以它可以运行在很多IaaS提供的云上。许多大的云提供商宣布了对Docker和它的生态系统的附加支持。

亚马逊已经引入Docker到它的弹性豆茎（Elastic Beanstalk）系统中（这是在IaaS之上的编制服务）。谷歌使Docker成为可管理的虚拟机（managed VMs），它提供了在应用程序引擎的PaaS和计算引擎的的IaaS之间的中间站。微软和IBM也都宣布了基于Kubernetes的服务，这样在他们的云上就可以部署和管理多容器应用了。

为了给当前使用的广泛多样的后端提供一致性的接口，Docker团队引入了 [libswarm](https://github.com/docker/libswarm )，它会被集成到多数的云和资源管理系统中。Libswarm一个明确的目标是“通过切换服务来源的办法来防止供应商锁定”。这通过呈现一组一致性的服务(以及相关的API)来完成，这些服务会附着到特定后端的实现上。比如Docker服务器服务会呈现Docker远程API到本地Docker命令行工具中，这样众多的服务提供者就可以管理相关的容器了。

基于 Docker 的新的服务类型仍在起步阶段。虽然位于伦敦的果园实验室（Orchard labs）可以提供Docker容器的托管服务,但 Docker 公司表示,在它收购果园实验室后相关服务不会被置于优先位置。Docker 公司还向 cloudControl 公司出售了其先前的 dotCloud PaaS 业务。如 [OpenVZ](http://openvz.org/Main_Page)之类基于旧的容器管理系统的服务已经比较普通了，所以在一定程度上 Docker 需要向主机提供商们证明其价值。

## Docker和它的发行版

Docker已经成为主流Linux版本的标准特性，比如Ubuntu, Red Hat Enterprise Linux (RHEL) 和CentOS。遗憾的是这些Linux发行版与Docker项目在步调上并不一致，从而导致这些Linux发行版中Docker的版本比当前能用的老得多。例如Ubuntu 14.04的Docker版本是0.9.1，并且在Ubuntu升级到14.04.1时Docker的版本也没有变（当时Docker项目版本是1.1.2）。在官方库中还有一个命名空间冲突的问题，因为Docker也是KDE系统托盘的名字，所以在Ubuntu 14.04中Docker包和命令行工具起了另一个名称“docker.io”。

在企业级Linux发行版中情况也没有太大的不同，CentOS 7中Docker的版本是0.11.1，这是Docker公司宣布1.0版本产品准备就绪之前的一个开发版。Linux发行版用户如果要使用最新的版本以保证稳定性、性能和安全，那么按照 Docker [安装指导](https://docs.docker.com/installation/#installation)操作并使用Docker公司提供的库，要比使用Linux发行版中的版本要好得多。

Docker的到来也催生了新的Linux发行版，比如 [CoreOS](https://coreos.com/) 和 Red Hat 的 [Project Atomic](http://www.projectatomic.io/) ，它们设计成能运行容器的最小环境系统。这些发行版相比传统Linux发行版本，有比较新的内核和Docker版本，对内存和硬盘的占用也比较小。新的发行版本中也有一些新的工具用来管理大容量的容器部署，比如 [fleet](https://github.com/coreos/fleet) 负责分布式系统启动，而 [etcd](https://github.com/coreos/etcd) 负责元数据管理。这些Linux发行版针对自身的分布式更新采用了新的机制，这样就可以使用最新版本的内核和Docker了。这表示对Docker应用的其中一种效果的认可，那就是把注意力重心从发行版和包管理解决方案转到Linux内核(和使用内核的Docker子系统)上来。

尽管新发行版（译者注：类似于CoreOS）可能是Docker最佳的运行方式，但支持容器的传统发行版和它们的包管理工具仍然非常重要。Docker Hub上提供了 Debian、Ubuntu和CentOS的正式镜像，还有“半官方”Fedora的镜像库。RHEL没有在Docker Hub上的镜像，因为它们是直接由Red Hat发行的。这意味着Docker Hub上的自动构建机制只对那些纯开源的Linux发行版本可用（并愿意信任那些起源于Docker公司团队策划的基础镜像）。

Docker Hub同时集成了源控制系统，如GitHub和Bitbucker，用来自动构建包管理器。包管理器可以在镜像构建过程中生成构建规格（在Dockerfile中）和最终构建镜像之间的复杂关系。构建过程的结果具有不确定性，这不是Docker的特定问题，而和包管理器如何工作相关。今天构建的是这个版本，在另一个时间构建可能会得到一个新的版本，这也就是包管理器需要更新功能的原因。容器抽象（即较少关注容器中的内容）与容器扩展（因为轻量级资源利用率​​）可能会使这种不确定性成为Docker相关的痛点。

## Docker的未来

Docker公司已经建立了清晰的道路，即发展核心能力（libcontainer）、跨业务管理（libswarm）和容器间消息（libchan）。与此同时，通过收购果园实验室（Orchard labs），Docker公司表达了利用自身生态系统的意愿。但是，这不仅仅关注Docker公司，这个项目的贡献者还来自于一些大牌公司，如谷歌、IBM和Red Hat。在仁慈的独裁者、首席技术官Solomon Hykes的掌舵下，Docker公司和Docker项目的技术领先有着明确的联系。在项目初始的18个月里，它已经显示出通过自己的输出来快速前进的能力，并且没有减弱的迹象。

许多投资者正着眼于十年前VMware公司ESX/ vSphere平台的功能矩阵，试图找出已经由虚拟机普及而驱动的企业预期和现有Docker生态系统之间的差距（和机会）。在网络存储和细粒度的版本管理（用于容器中的内容）领域，现有Docker生态系统做得并不好，这就为初创企业和在职人员提供了机会。

随着时间的推移，虚拟机和容器（Docker中的“运行”部分）之间的区别很可能变得不再那么重要，这将使注意力转到“构建（build）”和“交付（ship）”方面。这些变化将使 “Docker会发生什么？”的问题，相比“Docker会带给IT业什么？”的问题，变得更不重要。

## 关于作者

Chris Swan 是云网络软件供应商 [CohesiveFT](http://cohesiveft.com/) 的首席技术官。作为银行业的技术专家和技术领域的银行专家，他曾经有十几年的时间在从事金融服务业。他大部分时间是在大型瑞士银行中与应用服务器、计算网格、安全、移动和云这些基础设施打交道。克里斯还喜欢参与互联网上的修补工作，包括一些Raspberry Pi项目。

 
查看英文原文：[Docker: Present and Future](http://www.infoq.com/articles/docker-future)

感谢夏雪对本文的审校。

---

转载自 InfoQ 中文站，原文链接：[Docker : 现在和未来](http://www.infoq.com/cn/articles/docker-future)