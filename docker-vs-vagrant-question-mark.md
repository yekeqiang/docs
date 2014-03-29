#Docker vs Vagrant?
##Why this two technologies aren’t competing
#Docker vs Vagrant?
##为什么这两种技术之间不存在竞争

***
After reading some blogs with posts about “Docker vs Vagrant” ( like [this one](http://www.centurylinklabs.com/docker-vs-vagrant-cloud/) and [this one](https://phunehehe.net/docker-vs-chef-vagrant/)) I decided to write about this two technologies and how they are not competing against each other but they are complementing each other.

在阅读过现有的一些关于 “Docker vs Vagrant” 的博客后（比如[这一个](http://www.centurylinklabs.com/docker-vs-vagrant-cloud/)和[这一个](https://phunehehe.net/docker-vs-chef-vagrant/)我决定写一篇关于这两种技术的文章，并且表示这两项技术相互之间并不存在相互竞争的关系。）
***
I’m going to make a little summary about each technology as is established in their respective web pages with a brief explanation. After that, a little explanation in how you can use both technologies to improve your productivity.

首先我会参照对这两项技术各自的主页上已有的介绍进行简单的总结。在那之后，我会简略的介绍如何将这两种技术同时应用到你的生产环境中。

***
###Vagrant

Vagrant provides easy to configure, reproducible, and portable work environments built on top of industry-standard technology and controlled by a single consistent workflow to help maximize the productivity and flexibility of you and your team.

To achieve its magic, Vagrant stands on the shoulders of giants. Machines are provisioned on top of VirtualBox, VMware, AWS, or any other provider. Then, industry-standard provisioning tools such as shell scripts, Chef, or Puppet, can be used to automatically install and configure software on the machine.

###Vagrant

Vagrant 基于行业技术标准提供了易于配置，可量产，并且可移植的工作环境，并且由一个统一的工作流程进行控制，从而使得你和你的队伍的工作效率和灵活性得以最大化。

为了实现这些特性， vagrant 将一个文件（ vagrantfile ）安装进操作系统中（ 这个操作系统可以是任意，一般来说是你用来运行服务和云终端的系统 ）同时将你最终的应用中用来运行程序和处理进程所需要用到的库和软件一起装上。换句话来说， vagrant 安装在你的操作系统以后，你的 Docker 应用可以运行在你电脑和服务的 Docker容器中。

***
All this allows you to create similar environments ( no exactly the same but really close to each other ) where you can test your applications.

So, in summary:

Vagrant is a technology that will allow you to install all you need to develop your application in an easy and simple way.

所有的这些特性允许你创造相近的环境（不是完全相同而是指他们相互之间十分接近）用来测试你的应用。

因此，可以这样认为：

Vagrant技术可以允许你通过一种方便而又简单的办法将它融入你所开发的应用中。
***

###Docker

Docker is an open-source project to easily create lightweight, portable, self-sufficient containers from any application. The same container that a developer builds and tests on a laptop can run at scale, in production, on VMs, bare metal, OpenStack clusters, public clouds and more.

Docker是一个可以运行在任何一个应用中的轻量级，可移植，独立的容器的开源项目。一个在笔记本上编译和调试的容器可以按照规模应用在 虚拟机，bare metal ，openstack 集群，公有云等不同的生产环境中。

***
>译者注：译者认为此处的 [bare metal](http://en.wikipedia.org/wiki/Bare_metal) 又称为bare machine ， 是指没有安装任何操作系统的计算机。
***

Again in simple words, Docker is a technology that allows you to run applications inside containers ( that resemble virtual machines but are really different ) which assure you that all the libraries and packages needed by an specific application are always available and are always the same without mattering where you run them. That means that you can make a container for Memcache and another for Redis and you can be sure that in any Linux OS where you run your containers ( including the operating system that Vagrant just installed in your computer or server ), they will have the same behavior.

简单来说，Docker 技术允许你在容器中运行你的应用（ 这有点类似于虚拟机，但是事实上还是有很大的区别的 ）从而确保该应用所需要的所有函数库和包能够在任何情况下都有效运行，而不管你将他们跑在什么样的机器上。这就意味着你给 Memcache 和 Redis 分别定制的容器，可以任何一个 Linux 的发行版上（包括你电脑和服务器上那些安装了 Vagrant 的系统），它们都会有相同的运行效果。

***
>译者注：这里的 [Memcache](http://memcached.org/) 是指一种分布式缓存系统； 而[Redis](http://redis.io/) 是指键值型数据存储数据库，一般也可指数据服务器。这两项技术的共同点都是使用键值存储的方式存储数据。
***

***
So, in summary:

Docker is a technology that allows you to run self-contained applications eliminating worries about dependencies and libraries.

总结：

Docker 技术可以允许你通过将应用程序运行在一个容器中的方式来消除对于各种依赖和函数库的担忧。
***
##How to use both to improve your productivity

##如何将这两种技术同时应用到你的生产环境中

***

If at this point you are still not convinced that you can use both at the same time for your benefit let me give you an example:

如果此刻你仍然无法想象可以同时将两种技术整合到一起，那么让我为你举几个例子：

***


1. Install a Vagrant virtual machine in your computer containing the same OS you will have in your server ( normally Ubuntu Linux 12.04 LTS 64 bits). This means that you can program in any OS you want and still expect your program will run in your server.


    在你的电脑上安装和你的服务器有相同操作系统的 Vagrant 虚拟机（一般来说是 Ubuntu Linux 12.04 LTS 64 位版）。这意味着你可以在任何你想要的操作系统上编程并且借此来猜测你的程序将在服务器上运行。

2. Install your Docker packages to create Docker containers inside your virtual machine created for Vagrant. This step is better if you can install them through an script.
   
    通过在你虚拟机上安装 Docker 从而为 Vagrant 创造一个 Docker 容器。如果你能用脚本来实现这一步的安装的话会更好。

3. Inside your containers put your applications ( Nginx, Memcached, MongoDB, etc)

    在你的容器中安装应用程序（  Nginx, Memcached, MongoDB, 等 ）

4. Configure a shell script, Puppet or Chef script to install Docker and run your Docker containers each time Vagrant begins.

    配置一个 shell 脚本，Puppet 或者 Chef 脚本，每次当 Vagrant 运行时通过脚本去安装 Docker 和 运行你的 Docker 容器

5. Test your containers in your Vagrant VM inside your computer.

    在你安装了 Vagrant 的虚拟机上测试你的容器

6. Thanks to providers now you can take the same file ( your Vagrant file ) and just type vagrant up —provider=“provider” where the provider is your next host and Vagrant will take care of everything. For example, if you choose AWS then Vagrant will: Connect to your AMI in AWS, install the same OS you used in your computer, install Docker, launch your Docker containers and give you a ssh session.

    现在你只需要怀着对提供者感激的心情使用同一个文件（你的 Vagrant 文件）并且敲下  `vagrant up -provider = “ provider ” `（此处的 provider 是指你的镜像源），之后 Vagrant 会帮你完成一切的。比如你选择 AWS 作为你的镜像源，之后 Vagrant会将连接到你的 AWS中的 AMI，安装一个和你电脑上使用的操作系统相同的系统，安装上 Docker 容器并启动它，之后给你返回一个 ssh 会话。 

7. Test your containers in AWS and look that they behave exactly as you expect
   
    测试你在 AWS 中的容器并且观察他们是否得到你预想的运行效果。


***
>译者注：上面提到的 AWS 是指 Amazon Web Services ，是一个 Amazon 提供的云主机服务
***

As you can see Vagrant and Docker are not competing, they are complementing each other. So, if you are wondering which one you should learn my advice is to learn both, you could benefit a lot from both technologies.

你可以看出 Vagrant 和 Docker 并不是相互竞争，而是互补的关系。因此，如果你不知道应该学哪一个的话，我的建议是两个都学，这样的话你可以从两个技术中同时受益。
***

P.S If you found any grammar mistake please let me know to fix it. Thanks!

P.S 如果你发现任何语法错误的话请让我知道并且修复它。谢谢！

---
这篇文章由[ marcos_otero ](https://medium.com/@_marcos_otero)发表，点击[此处](https://medium.com/devops-programming/582135beb623)可查阅原文。

The article was contributed by [ marcos_otero ](https://medium.com/@_marcos_otero), click [此处](https://medium.com/devops-programming/582135beb623) to read the original publication.
