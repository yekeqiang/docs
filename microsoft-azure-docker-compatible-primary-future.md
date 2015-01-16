# 微软Azure云的Docker之路：兼容、原生与未来

---

##### 作者：张天雷

---

全球范围内技术领先的微软Azure云，无论从技术储备上还是从开发社区上，一直以来都受到业界的广泛称赞。与传统的云计算服务商不同，Azure这个后起之秀凭借Windows Server和混合云等技术，正逐步获得更多的市场占有率。从新的存储架构到基于Visual Studio的开发框架，甚至新的硬件架构设计，Azure云仍然一直在不断自我革新和进步。

现今，商业模式与商业气候对人们的创新要求日益增加，如何简单快速部署可移植的分布式的应用是近几年云计算领域努力解决的问题，Docker在这样的大背景下应运而生，并迅速获得社区认可，它能够在几分钟甚至更短的时间之内就将代码开发转换成产品，实现实时转换。Docker 是一个开源的应用容器引擎，它能帮助开发者打包应用以及应用的依赖包，并构建为一个可移植的容器，从而发布到任何流行的 Linux 或者Windows机器上，或者虚拟机。Container完全使用沙箱机制，容器相互之间不会有任何接口，就如同iPhone 的应用之间没有公共部分。这样的优势非常明显，应用的移植几乎没有性能开销，可以很轻松地在机器和数据中心中运行。最重要的是，这些容器不依赖于任何语言、框架或包括系统。

在业界对容器技术强烈的需求导向下，各大云计算厂商都纷纷开始考虑采用Docker作为其虚拟化技术的一部分。同样的，Azure也走上了一条先兼容再原生最后为己所用的Docker技术之路。

## 一、兼容

鉴于Docker在云计算虚拟化领域的迅速火热，Azure云首先采取了在自己的Linux虚拟机上兼容Docker的方式来吸引Docker社区的开发这。2014年6月9日，[Docker开发者大会](http://msopentech.com/blog/2014/06/09/docker-on-microsoft-azure/)上，Azure云合作伙伴项目经理Corey Sanders展示了直接利用[Azure跨平台工具集](http://azure.microsoft.com/en-us/documentation/articles/xplat-cli/)（由微软开放技术组开发）在其Linux虚拟机上运行Docker。这种兼容的方式，仅通过一条简单的“azure vm docker create”命令即可调用Docker进行容器的创建。更多细节步骤可以参考微软开放技术组给出的[用户说明](http://msopentech.com/blog/2014/08/15/getting_started_docker_on_microsoft_azure/)。在而后的7月，Azure则进一步[宣布](http://azure.microsoft.com/blog/2014/07/10/azure-collaboration-with-google-and-docker/)与Google和Docker合作来支持[Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)和[libswarm](https://github.com/docker/libswarm)开源项目在其云平台上的运行。Kubernetes是Google公司多年以来进行大规模容器管理的经验汇总而来的开源工具，发布以来收到业界广泛的好评，目前处于容器管理方面的领头位置，此前InfoQ也对其基本概念、构件等相关内容进行了[介绍](http://www.infoq.com/cn/articles/Kubernetes-system-architecture-introduction)。Libswarm则是Docker官方团队开源的一款容器管理工具。Azure云在这些工具的帮助下，更加灵活的支持着开发者的需求，使他们能够快速的构建、部署和管理跨系统、跨语言甚至公有私有混合的容器集群。在这里，无论是.NET、Python、Ruby、Node.js还是Hadoop和Oracle，都能够和Azure云平台无缝结合并运行，极大的简化了Windows Server系开发者的开发工作。更多的工具正逐渐加入Azure云管理工具集，如Puppet、Chef等。

在Windows Server下一个开发版本中，Docker引擎将会成为一个重要组成部分。同时，支持Windows Server的Docker引擎镜像将会在Docker Hub平台上发布，超过45000个Docker应用已经发布在了这个社区上。这将会大大帮助开发者在Windows Server和Linux平台上灵活地进行选择。

## 二、原生

兼容模式虽然是最快使用Docker的方式，但是开发者仍然需要准备Linux虚拟机作为Docker管理主机，这在一定程度上干扰了开发者的便捷开发。为此，2014年11月18日，Azure云高级经理Khalid Mouss在官网[发布](http://azure.microsoft.com/blog/2014/11/18/docker-cli-for-windows-clients/)了可以直接在Windows服务器环境下运行的原生Docker客户端，用来管理运行在Linux虚拟机上的Docker镜像。而在此之前，开发者只能使用Linux下的Docker命令或boot2docker工具来进行管理。这一举措极大简化了开发者使用Docker容器技术的曲折程度，得到了社区的热烈反馈。更多编译和使用原生客户端可以参考官网给出的[教程](https://ahmetalpbalkan.com/blog/compiling-docker-cli-on-windows/)。当然，目前原生软件还存在[很多问题](https://github.com/docker/docker/pull/9113)。让原本运行于Unix系统下的Docker在Windows系统上跑起来绝对不是一件轻而易举的事情。而目前软件的功能还仅限于将Docker客户端的代码编译通过，还不能在Windows环境下运行Docker监控程序或Docker容器。

除了积极采用Docker容器技术以外，Azure云团队也利用微软研究院强大的实力做了一些容器技术的自主研发：Drawbridge。主要针对现在Docker容器的安全性。相比之下，Docker功能比安全更引人瞩目，但这并不意味着安全是可以忽略的。坦率来讲，现在的容器并不安全。虚拟机与宿主之间共享数据过多又不能有效隔离，影响了容器安全性，至少在云计算容器用户间还达不到安全性要求。Drawbridge由两部分构成，分别是一个安全隔离容器picoprocess和一个运行于安全容器之内的系统Library OS。整个技术基于Windows Server。

## 三、未来

2014年10月15日，Azure云和Docker共同举办了[Docker全球开发者大会](https://blog.docker.com/2014/10/announcing-docker-global-hack-day-2/)。在Azure云副总裁Jason Zander[宣布](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/)了微软与Docker的合作伙伴关系以后，强强联合的两只技术团队对未来做出了如下设想：

- 在下一个版本的Windows Server中自带Docker容器引擎；
- 在Azure管理门户和镜像库中融合Docker Hub；
- Azure持续发布对Docker开放API的贡献，保证跨平台移植得以实现；

微软与Docker的合作是强强联合，在各自的领域中两个公司都是行业翘楚。Windows Server是企业级的应用系统，Docker的容器技术已经日趋炉火纯青。不难预见，Azure云的Docker之路将会给应用创新的商业市场中带来巨大变革，带动整个产业竞争力的提高。

---

感谢[郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE)对本文的策划和审校。

---

本文原载于InfoQ中文站，原文地址：[微软Azure云的Docker之路：兼容、原生与未来](http://www.infoq.com/cn/articles/microsoft-azure-docker-compatible-primary-future)