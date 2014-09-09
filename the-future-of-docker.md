#Docker 的未来

#####作者：[Carlos Sanchez](http://www.infoq.com/cn/author/Carlos-Sanchez)

#####译者：[郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE)
***
[Dokku](https://github.com/progrium/dokku) 的作者以及 [Docker](http://docker.io/) 早期的贡献者 [Jeff Lindsay](http://progrium.com/) 在 [CenturyLink 的一个采访中](http://www.centurylinklabs.com/the-future-of-docker/) 讨论了他正在参与的 Docker 的相关项目以及他们打算如何解决涉及到面向 Docker 服务的架构的问题。

Jeff 联合 [Flynn](https://flynn.io/) 开发了一个类似 Heroku 的下一代开源 PaaS 平台。他的目标是像 Heroku 这样的 PaaS 服务商一样，使用容器作为服务代替虚拟机：

>我非常希望容器能成为理想的日常工具。[...] 人们使用容器的方式更像是 SaaS ，所以当你运行容器时将会使用给定的 API 来管理和重新配置它，你不需要像之前那样修改配置文件。[...]我就是喜欢提供 API 的所有系统。

有几个项目就是围绕 Docker 来开发的，以便构建面向服务的架构。

[Discoverd](https://github.com/flynn-archive/discoverd) 是一个简单又强大的服务发现系统，目前基于 [Etcd](https://github.com/coreos/etcd) ，但是也可以使用 ZooKeeper 或者其它的分布式一致性存储系统。 类似 [Consul](https://github.com/hashicorp/consul) 和 Etcd 这样的项目只是提供基础的服务发现功能，但是 Discoverd 在它之上提供了一个更加具体和更易扩展的API来实现服务发现。

[Ambassadord](https://github.com/progrium/ambassadord) 是 Docker 远程代理（ Ambassador ）模式的实现，它允许跨主机连接 Docker 容器，支持静态转发、基于 DNS 的转发或者基于 Consul+Etcd 的转发。通过使用 iptables ， Ambassadord 可以基于端口来选择跳转到哪个主机，因此，集群中只需要一个代理即可。

[Registrator](https://github.com/progrium/registrator)（原名 Docksul ）是一个为 Docker 而设计的服务注册项目，它监听跨主机运行的容器的启动和停止，检查并向 Consul 或者 Etcd 注册它们（容器）。

[Consulate](https://github.com/progrium/consulate) 是由 Consul 、 Ambassadord 和 Registrator 驱动的针对 Docker 的分布式服务发现和路由网格的项目。 Consulate 在主机中运行后，集群中的任意容器之间都可以互相通信，它是软件定义网络方案的一种选择，它使用服务发现技术。

[Duplex](https://github.com/progrium/duplex) 是一个简单的应用程序通讯协议和库，受 ZeroMQ 的启发，它打算在一个弱中间人的（ brokerless ）的消息架构中运行 RPC 。 Duplex 允许在 [libchan](https://github.com/docker/libchan) 之上运行 RPC 并支持完整的 RPC 语义， libchan 是 Docker 的轻量级网络包。

[Configurator](https://github.com/progrium/configurator) 把传统的软件配置文件如 Nginx 、 Haproxy 、 Apache 转变为工具。它也是 [confd](https://github.com/kelseyhightower/confd) 的一个替代，可以在无中心存储的情况下运行。 Configurator 暴露出来的 REST API 可以通过程序的方式来修改这些服务的配置。

此外， Jeff 也在开发 Manifold ， Manifold 是一个基于 Consulate 的服务发现和分布式调度系统。 Manifold 用以替代 [Apache Mesos](http://mesos.apache.org/) ，它不仅简化了概念模型，而且还易扩展和可控制。 Manifold 类似于 CoreOS 的 [Fleet](https://github.com/coreos/fleet) ，它允许定义在集群中部署容器的策略，但是并没有绑定 Systemd 。
***
另外，为了更好的促进Docker在国内的发展以及传播， InfoQ 开设了 [《深入浅出Docker》](http://www.infoq.com/cn/dockers)专栏，邀请Docker相关的布道师、开发人员、技术专家来讲述Docker的各方面内容。InfoQ希望Docker专栏能帮助读者迅速了解Docker，希望新的技术、新的理念能让更多的人受益。


***
#####这篇文章原载于 [InfoQ](http://www.infoq.com/cn/) ，[Docker中文社区](https://www.dockboard.org) 得到授权后将其转载。您可以查看位于 InfoQ 的 [中文译文](http://www.infoq.com/cn/news/2014/07/google-kubernetes) ，点击 [这里](http://www.infoq.com/news/2014/08/the-future-of-docker) 查看英文。 
