#Clocker : 用 Apache Brokklyn 搭建的 Docker Cloud

#####作者：[Andrew Kennedy](https://twitter.com/cloudsoft)

#####译者：[刘梦馨](http://weibo.com/oilbeater)

***

这篇文章将介绍 [Clocker](https://github.com/brooklyncentral/clocker)， 一个能够让你快速搭建起 Docker Cloud 的一个开源项目。下面将会展示如何通过 Apache Brooklyn 建立 Docker Cloud，并在上面对应用进行部署和管理。

[Docker](http://docker.io/) 是针对开发者和系统管理员的一个分布式应用平台， [Apache Brooklyn](http://brooklyn.io/) 是一个开源的通过蓝图自动进行应用建模、监控以及管理框架，目前是 Apache Software Foundation 的一个孵化项目。

最近我们已经见到了一些关于 Docker 集群的管理项目，他们大多针对单独一个环境提供商提供管理服务。 Clocker 的设计目标是提供一种可移动的和提供商无关的管理 Docker 集群的方案。这样的话 Clocker 甚至可以在那些企业级的虚拟云或者私有云上进行部署。

## Docker Cloud的功能 

相对传统的虚拟机切分的方式 Docker 可以更有效灵活的使用设备上的资源，得到了越来越多的关注。

Docker 使得在一个单一主机上创建多个容器变得简单。但是很多应用需要大量的资源，一个主机无法承担，此外真实世界的应用部署中通常需要多个主机以达到弹性计算、容错以及扩容的目的。

Clocker 所搭建的 Docker 云可以让 Docker 容器拥有跨越多台主机的能力，就好像容器在一个虚拟云主机上运行一样。 Clocker 实现了容器的智能部署，按应用需求自动新增主机和容器的功能。这些都是通过深度遍历，和广度遍历等不同策略自动实现的。

Clocker 提高了在云计算世界中 Docker 集群管理的标准。

Clocker 主要特点：

- 在云设备中自动创建和管理多个 Docker 主机
- 智能部署，包括：
 - 弹性计算
 - 容错
 - 简单扩容
 - 最大主机资源利用率
 - 应用最佳性能
- 能够在任意公有云或者私有云上部署 Docker 主机
- 可以不经修改将现有的  Brooklyn/CAMP 部署到 Docker 主机上

## 开始使用 Docker Cloud 

（本文中的例子使用 IBM Softlayer Cloud, 你也可以使用其他的[云服务](http://brooklyncentral.github.io/use/guide/locations/index.html) 或者真实主机进行 Docker Cloud 的搭建。）

Clocker 利用 Brooklyn 进行云上的部署和管理，需要一个配置文件 ‘brooklyn.properties’ （[样例](http://brooklyncentral.github.io/use/guide/quickstart/brooklyn.properties)）以及云上的许可证书来申请云主机。

我们可以通过下面的命令来进行下载、解压以及运行 Brooklyn 服务器来配置 Docker 实例：

    % wget --no-check-certificate --quiet \
    -O brooklyn-clocker-examples-0.4.0-dist.tar.gz https://git.io/WOhfyw
    % tar zxf brooklyn-clocker-examples-0.4.0-dist.tar.gz
    % cd brooklyn-clocker-examples-0.4.0/
    % ./clocker.sh launc

你可以看到 Brooklyn 的输出：

    ...
    2014-06-09 22:12:51,536 INFO  Starting brooklyn web-console on loopback interface because no security config is set
    2014-06-09 22:13:04,564 INFO  Started Brooklyn console at http://127.0.0.1:8081/, running classpath://brooklyn.war and []
    2014-06-09 22:13:04,582 INFO  Persistence disabled
    2014-06-09 22:13:04,588 INFO  High availability disabled
    2014-06-09 22:13:21,767 INFO  Launched Brooklyn; will now block until shutdown issued. Shutdown via GUI or API or process interrupt.

当 Brooklyn 服务器运行起来后，可以通过浏览器访问管理主页（127.0.0.1:8081）启动 Docker Cloud 目录.

{<1>}![](http://docker.u.qiniudn.com/create-application.png)

你将会看到有如下配置选项的页面，选择你的 Docker 主机将要部署的位置。 部署位置的名称可以设置为 my-docker-cloud 这样我们就能再配置选项中通过这个名字引用到这台主机，其余的配置可以保持默认配置。  Container Cluster Maximum Size 选项设置为4，控制每台主机上的容器数目，host and Host Cluster Minimum Size 设置为2，控制最初启动的 Docker 主机数目。

{<2>}![](http://docker.u.qiniudn.com/softlayer-docker-cloud-config.png)

选择 Finish Docker Cloud 就可以运行 。当云虚拟机和 Docker 主机软件被下载和安装完成后，将会收到一个启动成功的报告，如下图所示：

{<3>}![](http://docker.u.qiniudn.com/softlayer-docker-cloud-running.png)

"docker.machine.* sensor" 数据展示了 Docker 主机的使用情况，包括 CPU 和内存的使用率。

## 部署一个简单应用 ##

下面是一个 Tomcat 服务的部署 YAML 文件，通过一个 War 文件来进行配置。这个文件可以复制到 Brooklyn 的 ‘Add Application’ 对话框中。

    name: "Tomcat Web Application"
    location: my-docker-cloud
    services:
    - serviceType: brooklyn.entity.webapp.tomcat.TomcatServer
      brooklyn.config:
    docker.dockerfile.url:
      "https://s3-eu-west-1.amazonaws.com/brooklyn-clocker/UsesJavaDockerfile"
    wars.root:
      "https://s3-eu-west-1.amazonaws.com/brooklyn-clocker/hello-world.war"
    jmx.agent.mode: "JMXMP"
    
这个 docker.dockerfile.url 配置项指向了一个为 Java 应用预先加载 OpenJdk 命令的 Dockerfile。（这通常是部署中最慢的一个环节，未来有可能会通过把 Tomcat 归档文件包含到 Dockerfile 来优化这一部分性能。）

默认的 Docker 主机填充策略是最大化利用当前主机的资源，直到资源不足再从部署蓝图中分配下一个主机。其他可选的填充策略还有广度优先（将容器平均填充在各个主机）以及 CPU 优先（优先填充 CPU 利用率最低的主机）。

当 Docker 容器启动后（通常只需几秒，而云虚拟机通常要数分钟），由 Brooklyn 进程创建的 Tomcat 服务将会接管下面的事情。（从 Apache 镜像下载二进制文件， 解压并运行 catalina.sh 运行脚本）。

这个过程时间小于一分钟，完成后 Tomcat 服务将可以为配置好的网络应用提供服务。

{<4>}![](http://docker.u.qiniudn.com/softlayer-tomcat-server-running.png)

这个 Tomcat 进程绑定到了8080端口。那些熟悉 Brooklyn 的人可能会注意到 mapped.http.port 这个选项，为一个在 49000-65535之间的端口号。这些是 Docker 主机的共有 IP 所映射的端口号，他们可以用于进行容器中不同进程之间的通信以及不同容器间的通信。

JMX 数据通过类似的方式获得， Brooklun 通过 mapped.jmx.direct.port 进行连接。这些映射都是透明的，在部署简单应用时我们无须更改现有的 Brooklyn/CAMP 蓝图。

## 部署复杂应用 ##

下面这个例子展示了一个包含 SQL 数据库、负载均衡器以及一个集群应用服务器的弹性网络应用。相比上个例子更加复杂，需要用到多个实例类型，并展示了 Docker Cloud 是如何提供相对手工灵活的 Docker 主机配置。

该应用的蓝图定义在 Java 类 brooklyn.clocker.example 中。 [TomcatClusterWithMySql](https://github.com/brooklyncentral/clocker/blob/master/examples/src/main/java/brooklyn/clocker/example/TomcatClusterWithMySql.java) 在应用目录中被配置。

在启动中，从 Brooklyn 中断中选择 Add Application，并从蓝图列表中选择 Elastic Web Application。 

选择 my-docker-cloud 为 部署位置，并选择 Finish 来启动应用。

{<5>}![](http://docker.u.qiniudn.com/tomcat-cluster-config.png)

Docker 主机部署每一个容器，并且把每一个实例展示在管理树中。每个容器拥有一个 docker.container.entity 选项，指向部署的容器实例。

{<6>}![](http://docker.u.qiniudn.com/softlayer-tomcat-cluster-running.png)

运行正的应用提供一个 URL 指向 Nginx 的负载均衡器，并且可以通过这个 URL 来访问应用。这个运行中的容器 URL 被映射到了公有 URL，相关信息可以在 mapped.webapp.url 中看到。

{<7>}![](http://docker.u.qiniudn.com/softlayer-tomcat-cluster-nginx.png)

## 工作原理 ##

Brooklyn 利用一个云 API 库 [Apache jclouds](http://jclouds.apache.org/) 来提供云虚拟主机之间的安全传输（SSH）。

Docker 的架构是提供一种在主机上建立容器的机制。Brooklyn 利用 jclouds 来配置云机器，使其成为 Docker 主机。

{<8>}![](http://docker.u.qiniudn.com/simple-docker-architecture-diagram-cropped.png)

Brooklyn 通过一个 Dockerfile 来使得不同 Docker 容器之间可以进行 SSH 服务。这样每个容器可以被当做一台独立的虚拟主机。为了能够完全透明的做到这一点，需要对不同的云之间使用相同的 API 并为部署 Docker 容器开发对应的 jclouds 驱动。尽管这样做很繁琐，但在实际中有很好的效果。

Brooklyn 可以从下述层面获取数据并进行管理和配置：

- 整个应用
- 每个云机器（Docker 主机）
- 每个 Docker 容器
- 组成应用的每个软件

这些都使得自动在 Docker Cloud 中分发应用变得可能。

## 阅读更多 ##

[AMP for Docker](http://www.cloudsoftcorp.com/blog/2014/03/amp-for-docker/) 讲述了 jclouds 驱动， [Implementing a Docker Cloud with Apache Brooklyn](http://abstractvisitorpattern.blogspot.com/2014/06/clocker-implementing-docker-cloud-with.html)  对 Brooklyn 的整体架构做了更进一步的探索。

## 路线图 ##

这只是 Clocker 的第一个发行版，开发还在进行中。我们欢迎您的各种想法和贡献（[GitHub](https://github.com/brooklyncentral/clocker)）。

下一个版本中将会出现的新功能有：

- 更多的容器管理部署策略。当前版本包括广度优先，深度优先以及 CPU 自适应策略。之后将会有基于内存和 IO 的策略。
- 实现关联和反关联 API 以及简单的 DSL 来根据应用实例来控制容器的部署。并且可以在每个实例的基础上利用蓝图来指定关联选项。
- 改进 jclouds-docker 驱动，可以更多的控制 Docker 容器的配置，比如 CPU 共享以及内存分配。
- Docker 镜像库的继承。可以从一个集中控制的地方获取镜像，并且在不同的 Docker Cloud 主机间共享。
- 增加对诸如 [Open vSwitch](http://openvswitch.org/) 和  [OpenContrail](http://opencontrail.org/about/) 这类 SDN 的支持，可以隔离和控制容器的网络流量。并且可以通过引入一个共享的 Docker Vlan 来进行容器间的通信。
- 将现有的 Brooklyn 蓝图和 Docker 进行融合。

## 总结 ##

Clocker 可以容应用蓝图部署在云上的 Docker 容器中，在云虚拟机上建立 Docker 主机来传播智慧。

应用的蓝图可以将 Docker 看做一个 Docker Cloud 就好像他是一组在同一位置的同构的虚拟机器。Docker Cloud 可以使得关于容器部署和配置智能决策更有效的利用资源并提升性能。

[源代码](https://github.com/cloudsoft/brooklyn-docker/) 遵循 Apache 2.0 协议，可以在 Github 上下载到。欢迎 [fork](https://github.com/cloudsoft/brooklyn-docker/fork) 并通过 pull request 来贡献代码或者发起一个新的 issue 。

想要更多的了解 Clocker，请 [联系我们](http://www.cloudsoftcorp.com/contact/) 并在 IRC 频道上寻找 #brooklyncentral 。

## 致谢 ##

感谢 Andrew Kennedy（ [@grkvlt](https://github.com/grkvlt/) ）以及 Andrea Turli （ [@andreaturli](https://github.com/andreaturli/) ）的帮助。

***
#####这篇文章由 [Andrew Kennedy](https://twitter.com/cloudsoft) 撰写， [刘梦馨](http://weibo.com/oilbeater) 翻译。点击 [这里](http://www.cloudsoftcorp.com/blog/2014/06/clocker-creating-a-docker-cloud-with-apache-brooklyn/) 阅读原文。

#####The article was contributed by [Andrew Kennedy](https://twitter.com/cloudsoft), click [here](http://www.cloudsoftcorp.com/blog/2014/06/clocker-creating-a-docker-cloud-with-apache-brooklyn/) to read the original publication.



