#谷歌的容器工具得到微软、IBM 等公司的支持

#####作者：[Richard Seroter](http://www.infoq.com/cn/author/Richard-Seroter)

#####译者：[郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE)

***
近日，谷歌开源了其管理 Docker 容器的编配工具 Kubernetes 。上周，微软、IBM、红帽、 Docker 、 Mesosphere 、 CoreOS 和 SaltStack 纷纷加入并表示将积极支持该项目。虽然各个厂商都有他们支持 Kubernetes 的理由，但是这一快速的合作也证明了 Linux 容器的发展势头。

谷歌的高级副总裁 Urs Hölzle 在其 [博文](http://googlecloudplatform.blogspot.com/2014/07/welcome-microsoft-redhat-ibm-docker-and-more-to-the-kubernetes-community.html) 中介绍了容器的优势并欢迎前面提到的厂商加入该项目。

>今天，微软、红帽、IBM、Docker、Mesosphere、CoreOS 和 SaltStack 都正式加入 Kubernetes 社区，他们将积极为该项目作出贡献。各个公司都将为 Kubernetes 带来其独特的优势， 我们将共同确保 Kubernetes 在任何应用程序和环境下都是一个强大并且开放的容器管理框架——无论私有云、公共云还是混合云环境。

Hölzle 表示微软会努力使 Kubernetes 成功应用于其 Azure 云平台上；红帽计划在其混合云产品中提供对 Kubernetes 的支持；IBM 将为 Kubernetes 和 Docker 贡献代码并尝试建立一个管理模型；Docker 会保证 Kubernetes 与他们的相似服务Libswarm 紧密结合；CoreOS 会确保 Kubernetes 可以运行在其与 Docker 相关的操作系统中；Mesosphere 会把 Kubernetes 集成到他们的管理工具 Mesos 中； SaltStack 会把 Kubernetes 作为他们配置管理工具集的一部分。

尽管容器不是什么新的概念，但是去年 Docker 以一种 [简单的方式](http://www.infoq.com/articles/docker-containers) 忽然走进公众视野并激发了人们对这个技术概念的兴趣。当外界开始大量使用容器时（谷歌宣称他们每周要启动超过20亿个容器），如何高效的管理它们就变得至关重要。 CoreOS 团队解释了 Kubernetes 如何解决这一问题。

> Kubernetes 引入了 pod（容器仓）的概念，容器仓表示一组应该被当作一个单一逻辑服务来部署的容器。容器仓与日趋流行的“一个容器中运行一个服务”的模式配合得很好。 Kubernetes 可以轻松地在单一机器或者集群中运行多个容器仓以保证较好的资源利用以及高可用性 。 Kubernetes 可以监控容器仓以确保它们正常运行在集群中。

Docker 与 Kubernetes 掀起的热潮是否说明了大家对如今占主导地位的服务虚拟化模型的不满？来看看 VentureBeat 是怎么看这次的合作声明的。

> 厂商集体为 Kubernetes 站台，也可以看作是大家对更碎片化、更专有化的 hypervisor 技术的日渐失望， hypervisor 技术构建于服务操作系统之上，并在每个物理机上创建了很多的虚拟机来运行应用程序。当开发者以及公司都开始尝试 Kubernetes 时，销售 hypervisor 软件的公司（包括 VMware ）在此容器化的时代该何去何从。

这些形形色色的 Kubernetes 贡献者试图推动 Kubernetes 在公共云和私有云领域的发展，我们也注意到亚马逊云服务并不在这些厂商之列。 VentureBeat 指出类似 Kubernetes 这样的框架可以帮助云开发者以及非云开发者不至于受限于某一家云服务商或操作系统供应商。在[《纽约时报》的 Bits 博客中有一篇采访](http://bits.blogs.nytimes.com/2014/07/10/cloud-computing-giants-add-to-open-source-credentials-with-kubernetes/?_php=true&_type=blogs&_php=true&_type=blogs&_php=true&_type=blogs&_r=2&)，其中谷歌云平台总经理 Miles Ward 称 Kubernetes 可以帮助公司连接公共云和私有云环境，他表示谷歌完全支持公共云的想法，但也承认还有上万亿美元的闲钱在投资私有云计算业务 。容器意味着传统服务器虚拟化的结束？ Kubernetes 的合作商 Mesosphere 认为“虚拟化已经毫无意义”，容器才是未来；同时微软认为，在企业运行系统的一系列解决方案中，容器是其中一部分。

那谷歌联合这些合作伙伴的目的是什么呢？谷歌注意到容器是一个颠覆性的技术，这个技术应该得到广泛投入，而且它在自家不断增长的云平台上表现出色。 [GigaOm](http://gigaom.com/2014/07/10/with-microsoft-ibm-and-red-hat-backing-it-googles-kubernetes-is-a-peace-pipe-and-trojan-horse/) 上的一篇关于谷歌的采访描述了容器的未来。

>谷歌的产品经理 Craig McLuckie 承认合作伙伴的所付出的努力的重要性，他们和 Google 一起验证、推动它的发展并构建一个大家愿意为之努力的生态圈。“有很多的软件公司…….开始对容器感兴趣，这非常重要，容器一个颠覆性的技术”，他说。

>假使 Docker 和 Kubernetes 成为管理应用的标准方式，那就意味着云的区别会回归到原来的基础设施层，而谷歌认为他们有几十年的经验优势来建立世界上最大的系统。 McLuckie 说“我们对我们的基础设施质量非常自信，”一个可以让用户基于底层的基础设施来选择供应商的技术是非常有利于谷歌的，他补充道。

***

这篇文章原载于 [InfoQ](http://www.infoq.com/cn/) ，[Docker中文社区](https://www.dockboard.org) 得到授权后将其转载。您可以点击这里查看 位于 InfoQ 的 [文章页面](http://www.infoq.com/cn/news/2014/07/google-kubernetes) 。
