# Docker vs Vagrant Cloud

什么是 vagrant ? Vagrant 是一个跨平台的虚拟机构造工具，能够通过 [vagrantfile](https://github.com/patrickdlee/vagrant-examples/blob/master/example2/Vagrantfile) 
描述虚拟机并将其部署到 hypervisor 上。

什么是 docker ? Docker 是一个 linux 上的 linux container 构造工具，能够通过 [dockerfile](https://github.com/CenturyLinkLabs/ctlc-docker-wordpress/blob/master/Dockerfile) 来定义一个 container ，并将其部署到任何运行 docker 的主机上。

Vagrant 和 docker 都能够通过一个配置描述文件来构造一个运行环境，然而区别在于 vagrant 构造一个虚拟机需要几分钟而 docker 构造
一个 linux container 只需要不到一秒钟。

事实上， container 和虚拟机之间的界限并不清晰，另外由于 docker 基于 linux 内核，所以如果你想在 OSX 上运行 docker，
通常需要利用 vagrant 构造一个虚拟机并在其中运行 docker。这个步骤只需要几行命令

```
$ mkdir ubuntu
$ cd ubuntu
$ vagrant init ubuntu
$ vagrant up
```

近期发布的 vagrant 1.5 版本加入了很多[新的特性](http://www.vagrantup.com/blog/vagrant-1-5-and-vagrant-cloud.html)，
其中包含 vagrant cloud。[Vagrant cloud](https://vagrantcloud.com/) 是一个 vagrant 的托管服务，
能够用于发现、分享 vagrant box，并且能够管理 vagrant shares （反向代理你的 vagrant box 的所有端口）

> 译者注：vagrant share 能够让任何人从任何地方访问将本地的 vagrant 虚拟机。基本原理是在你的 vgrant 主机上(非虚拟机)开启一个本地代理，
本地代理会连接到 vagrant cloud 上的远程代理，后者与一个在 vagrant cloud 上的拥有公共 IP 的超小虚拟机关联，从而任何对公共 IP 的访问最终
都被代理到本地的虚拟机。

对于 docker 来说最酷的地方莫过于使得构造和封装 linux container 环境变得可协作，用户可以通过
在一台笔记本上执行 `docker commit` 和 `docker push` 来分享本地的 container，任何人都可以通过 `docker pull` 和
`docker run` 构造出完全一样的 linux container 环境。这是一次革命性的变化，就如同将 linux 环境加入了 git 一样的版本控制系统。

这也是为什么 vagrant cloud 的意义非凡。Vagrant cloud 引入类似的协作机制，将虚拟机变的可以更方便的分享。和`docker pull`一样，
通过`vagrant box update`，你可以升级你的虚拟机，vagrant cloud 甚至可以托管你的私有虚拟机镜像。

现在再来看 vagrant 和 docker 之前的一些差异：

* Docker (或许)永远不会支持运行 windows，因为其基础是建立在 linux kernel 上的
* Vagrant 支持更多的操作系统
* 虚拟机较之 linux container 拥有更好的安全性和隔离性，尽管虚拟机比 linux container 更慢一些。

但是这意味着 docker 没有市场了吗？

不，docker 除了`docker commit`来分享之外还有更多的特性：

* 轻量级的隔离环境比虚拟机能够更方便和快捷地启动和停止
* 可以在一个虚拟机中运行多个 container，从而节省开销
* Docker 的 container 机制更适合一些[持续集成/持续发布和微型 PaaS 场景](http://www.centurylinklabs.com/top-10-open-source-docker-projects/)

然而有明显的迹象表明虚拟机将会在这些方面有所改善。

结语

虚拟机和 linux container 的竞争十分激烈，二者在竞争中相互启发并走向技术的进步。比如 vagrant 作者的 [serf](http://serfdom.io/) 
项目可以[很方便地在 linux container 环境中使用](http://www.centurylinklabs.com/decentralizing-docker-how-to-use-serf-with-docker/)。

从总体趋势来看 linux container 和虚拟机应该是会同时存在并协同工作的。一些[创业公司](http://www.centurylinklabs.com/top-10-startups-built-on-docker/)已经支持原生的 linux container 托管了，openstack 也如同支持虚拟机一样支持 docker container 了。

我们拭目以待 vagrant 和 docker 能够更多的一起工作并且更好地发挥各自优势。

> 相比正文，[Solomon Hykes](http://blog.docker.io/author/solomon/)在原文的评论更加有趣，以下为节选

* docker 可以通过 boot2docker 在 mac 中使用 (译者吐槽，但仍然是虚拟机)
* docker 的 windows 支持社区正在努力
* docker 正向通用的 container 引擎发展(而非 linux container)，通过插件的机制，抽象出执行环境和存储，如同 0.7 中引入的 storage driver 和 0.9 中的 execution driver。通过这种机制可以向上屏蔽虚拟化技术和操作系统的区别. (具体的虚拟化方法可以由相应的插件实现，如 hyper-v driver 来解决 windows 的问题)

> 译者十分认同 Solomon Hykes 的观点，目前的 docker 的发展趋势更多的是构造 app runtime，而非 vagrant 强调的 build time。尽管 docker 一开始
是以 linux container 为切入点的，但是从 dotcloud 的角度来讲，更希望其成为一个能够提供更多 PaaS 功能(更关注 runtime)的平台。

---
这篇文章由[ Lucas Carlson ](http://www.centurylinklabs.com/author/cardmagic/)发表，点击[此处](http://www.centurylinklabs.com/docker-vs-vagrant-cloud/)可查阅原文。

The article was contributed by [Lucas Carlson](http://www.centurylinklabs.com/author/cardmagic/), click [here](http://www.centurylinklabs.com/docker-vs-vagrant-cloud/) to read the original publication.