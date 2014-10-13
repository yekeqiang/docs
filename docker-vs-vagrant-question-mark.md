# Docker vs Vagrant? 为何这两种技术不存在竞争

![alt](http://resource.docker.cn/vagrant-vs-docker.png)


##### 作者：[Marcos Otero](https://medium.com/@_marcos_otero)
##### 译者：[Mike Ling](https://twitter.com/tiramisu1993)
***

在阅读过现有的一些关于“Docker vs Vagrant”的博客（比如 [这篇](http://www.dockboard.org/docker-vs-vagrant-cloud/) 和 [这篇](https://phunehehe.net/docker-vs-chef-vagrant/) ）后，我决定写一篇文章来阐明这两种技术并非彼此竞争而是相互补充。

首先我会参照这两项技术在各自主页上的介绍，进行简单的总结。然后，我会简略地介绍如何将这两种技术同时应用到生产环境中。

***

## Vagrant

>Vagrant 基于行业技术标准提供了易于配置、可量产、并且可移植的工作环境，并且由一个统一的工作流程进行控制，从而使你和团队的工作效率和灵活性得以最大化。

>为了实现这些特性， Vagrant 站在巨人的肩膀上，诸如 VirtualBox 、 VMware 、 AWS 等顶级供应商以及其他供应商。同时业界标准的供应工具，诸如 shell scripts 、 Chef 或者 Puppet 也都能在机器上自动安装和配置软件。

简言之， vagrant 能够将一个文件（ vagrantfile ）安装进操作系统中（任意操作系统，一般来说是用来运行服务和云终端的系统），同时也能将最终应用中运行程序和处理进程所需要用到的库和软件一起装上。换句话来说， vagrant 能够安装操作系统和 Docker 应用，然后 Docker 容器就可以在你电脑和服务器上运行。

所有的这些特性允许你创造相近的环境（不是完全相同而是指他们相互之间十分接近）用来测试你的应用。

因此，可以这样认为：

***Vagrant 技术能够让你以一种方便简单的方式安装开发应用所需的一切。***


## Docker

> Docker 是一个能让你在任何应用中都能轻松创建轻量级、可移植、自足容器的开源项目。一个在笔记本上编译和调试的容易能够大规模应用在虚拟机、 bare metal 、 OpenStack 集群、公有云等不同的生产环境中。

> ##### 译者注：译者认为此处的 [bare metal](http://en.wikipedia.org/wiki/Bare_metal) 又称为bare machine ， 是指没有安装任何操作系统的计算机。


简单来说，Docker 技术允许你在容器中运行你的应用（这有点类似于虚拟机，但是事实上还是有很大的区别），从而确保该应用所需要的所有库和包能够在任何情况下都有效运行，而不管你将他们跑在什么样的机器上。这就意味着你给 Memcache 和 Redis 分别定制的容器，在任何一个 Linux 的发行版上（包括你电脑和服务器上那些安装了 Vagrant 的系统），它们都会有相同的运行效果。

> ##### 译者注：这里的 [Memcache](http://memcached.org/) 是指一种分布式缓存系统；而 [Redis](http://redis.io/) 是指键值型数据存储数据库，一般也可指数据服务器。这两项技术的共同点都是使用键值存储的方式存储数据。

总结：

***Docker 技术可以允许你通过将应用程序运行在一个容器中的方式来消除对于各种依赖和库的担忧。***

***

## 如何将这两种技术同时应用到你的生产环境中


如果此刻你还不能相信可以同时将两种技术整合到一起，那么让我给你举个例子：

1. 在你的电脑上安装与服务器有相同操作系统的 Vagrant 虚拟机（一般来说是 Ubuntu Linux 12.04 LTS 64 位版）。这意味着你可以在任何你想要的操作系统上编程，并且期待这些程序也可以在服务器上运行。

2. 通过在你虚拟机上安装 Docker 从而为 Vagrant 创造一个 Docker 容器。如果你能通过脚本来实现安装的话会更好。

3. 在你的容器中安装应用程序（ Nginx 、 Memcached 、 MongoDB 等 ）

4. 配置一个 shell 脚本、 Puppet 或者 Chef 脚本并安装 Docker 。每次 Vagrant 启动时运行 Docker 容器。

5. 在你安装了 Vagrant 的虚拟机上测试你的容器。

6. 现在你只需要怀着对提供者感激的心情使用同一个文件（你的 Vagrant 文件）并且敲下  `vagrant up -provider = “ provider ” `（此处的 provider 是指你的镜像源），之后 Vagrant 会帮你完成一切的。比如你选择 AWS 作为你的镜像源，之后 Vagrant 将会连接到你的 AWS 中的 AMI，安装一个和你电脑上使用的操作系统相同的系统，安装上 Docker 容器并启动它，之后给你返回一个 ssh 会话。 

7. 测试你在 AWS 中的容器并且观察他们是否得到你预想的运行效果。

> ##### 译者注：上面提到的 AWS 是指 Amazon Web Services ，是一个 Amazon 提供的云主机服务。


***
你可以看出， Vagrant 和 Docker 并不是相互竞争，而是互补的关系。因此，如果你不知道应该学哪一个的话，我的建议是两个都学，这样的话你可以同时从两个技术中受益。
***

P.S 如果你发现任何语法错误的话请让我知道并且修复它。谢谢！

---
##### 这篇文章由 [Marcos Otero](https://medium.com/@_marcos_otero) 发表，[Mike Ling](https://twitter.com/tiramisu1993) 翻译。点击 [这里](https://medium.com/devops-programming/582135beb623) 可查阅原文。

##### The article was contributed by [Marcos Otero](https://medium.com/@_marcos_otero), translated by [Mike Ling](https://twitter.com/tiramisu1993). Please click [here](https://medium.com/devops-programming/582135beb623) to read the original publication.
