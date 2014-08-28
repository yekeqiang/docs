#Docker Vs Chef

#Docker Vs Chef

***
“I love the long time it takes to manually provision a brand new server from scratch!” said no one ever. One of the things that sysadmins and devops have long regarded as a necessary evil is the overhead time wasted twiddling one’s thumbs while waiting for a new server’s creation. With the advent of virtualization and cloud computing and the resultant massive increase in need for computing, the frequency of new-server creation has only increased as well.

从没有人会说：“我喜欢通过手动开启服务来浪费的时间”。长久以来，系统管理员和开发人员在等待一个新的服务被创建时只能靠玩弄自己的大拇指来打发时间被认为是很多糟糕但无可奈何的事情之一。尽管随着虚拟化和云计算以及对于大规模运算的快速发展，在新服务创建所需要的时间方面的发展却收效甚微。

***
Enter tools like Docker and Chef. **In a nutshell**, they both make it much easier to define and roll out machine-level changes quickly in a multi-server environment. But the roads they take to arrive to the location of this holy grail are very different indeed, as we shall now see.

 安装[Docker](www.docker.com) 和 [Chef](http://www.getchef.com/) 这样的工具只需要占用和**坚果壳一样大小的空间**，但是他们却非常容易使用且能给你的多服务器环境带来机械级的改善。只不过他们达到这种效果的方式却是非常不一样的，接下来会一一介绍。

***
###What They Are
###他们是什么

Docker started life as an open-source side project called dotCloud. But when its creators realized its potential, it quickly became a top priority. It is a Linux-only tool developed in the Go language, and it builds on LinuX Containers (LxC) to create Virtual Environments (VE’s). Now pay attention kids, because a VE is radically different from a virtual machine (VM). A VM is created by a virtualizer tool or emulator (HyperV, VMWare, etc), and it is a full image of a source machine, including the OS, disk config, network and even virtual processors. The resulting bloated mass eats up quite a bit of disk space and is slow to deploy and start up.

Docker 是来着/与一个名叫 dotcloud 开源项目。但是当它的编写者意识到它的潜力的时候，它立刻变成了一个炙手可热的项目。它由 GO 语言编写的，并且只支持 Linux。他是基于 Linux 容器（LxC）来创建一个虚拟环境。现在请注意喽，因为虚拟环境VE ( Virtual Environment ) 和 一台虚拟机（ VM ）是很不一样的。一台虚拟机是由虚拟工具或者模拟器( HyperV, VMWare, 等等 )创建的，并且它是一个全镜像的主机源，其中包括操作系统，硬盘调整，网络和虚拟进程。所以过于臃肿的结构吃掉了大量的硬盘空间同时拖慢了运行和开机速度。
***
>译者注：[LXC](https://linuxcontainers.org/)是一个 Linux 提供的收容功能接口，通过 LXC 提供的 API 和简单的工具，使得 Linux 用户可以简单的创建和管理系统或者应用的空间
***
Docker solves this problem by isolating the OS from the kernel. It then creates a ‘level-zero’ package from the kernel-machine baseline. Any subsequent changes are captured as snapshots called namespaces, and can be run in standalone containers as virtual systems. Snapshots capture only the changes from the ‘base template’, not the full machine setup (unlike VM’s). From this point, namespaces are stored and can be installed on their own to create full environments on any Docker-ready host.

Docker 通过它与操作系统的内核相分离的特点很好的解决了这个问题。它会首先通过创建一个与机器内核相分离的 “等级0” 包裹。任何之后的改变都说对于命名空间的调用进行捕捉并快速的将它记录下来并且可以被当做虚拟机上单独运行容器中。“快照”只能捕捉到“基本模版”的改动，而不是整个机器上面的设置（不像是VM的快照功能）。从这一点我们可以看出命名空间可以储存并且安装在任何一个他们自己创建的安装了 Docker 的环境中。

***

Chef is a CM tool. Together with Puppet, they are the big boys of the CM marketspace. It was first released in 2009, and it helps admins and devops professionals manage their environments by making it easy to roll out changes and even provision new machines. Chef fully supports Windows as well as Linux and Unix, and it also boasts a proper GUI (though still not as slick as Puppet’s). However, there are complaints that Chef is a muddle to understand for newbies, and the documentation is not easy to wrap your head around, especially for newbies.

Chef 是一个 CM 工具。它和 [Puppet](http://puppetlabs.com/)并称为 CM 工具系列中最常用组件。它最初是在2009年发布，由于它在机器上对于一些改变进行回滚和新机器的环境进行配置的操作简便，被管理员和开发者用来管理自己的机器环境。Chef 完全支持 Windows 和 Linux 以及 Unix， 并且它有令自己自夸的 GUI （虽然不像 Puppet 的那么平滑）。但是也有人抱怨说 Chef 对于新手来说并不是这么好理解，同时文档可能让使用者头疼特别是对于新手而言。

***
>译者注：前文指的 CM 指的是 [Configuration management](http://en.wikipedia.org/wiki/Configuration_management),本意指是一个建立和保持生产连续性的生产模型，但是笔者认为此处是指[Comparison of open-source configuration management software](http://en.wikipedia.org/wiki/Comparison_of_open-source_configuration_management_software)---一系列开源的 Congratulation management 组件。
***

###How They Work

###他们是如何工作的

***
Docker doesn’t create its own virtual computer with a distinct OS and processors and hardware emulation. A VE is a lite VM;** it rides on the already existing kernel’s image of the underlying hardware**, and only creates a ‘container’ in which to run your apps. It can even recreate the OS, since the OS is just another app running on the kernel. Think of Docker as a turbocharged version of LxC; it offers the following additional features that LxC doesn’t:



- Portable deployment across machines: you can use Docker to create a single object containing all your bundled applications. This object can then be transferred and quickly installed onto any other Docker-enabled Linux host.


- Versioning: Docker includes git-like capabilities for tracking successive versions of a container, inspecting the diff between versions, committing new versions, rolling back etc.


- Component reuse: Docker allows building or stacking of already created packages. For instance,  if you need to create several machines that all require Apache and MySQL database, you can create a ‘base image’ containing these two items, then build and create new machines using these already installed.


- Shared libraries: There is already a public registry (http://index.docker.io/) where thousands have already uploaded the useful containers they have created. Again, think of the AWS common pool of different configs and distros – this is very similar

For an exhaustive list of Docker’s capabilities, [follow this link](http://stackoverflow.com/questions/17989306/what-does-docker-add-to-just-plain-lxc).

Docker 不会通过建立独有的操作系统，进程和对硬件进行模拟来创建属于自己的虚拟机。一台 VE 就像是轻量级的 VM ；**它在已有的内核关于底层硬件的镜像上**建立一个可以用来运行你的应用的‘容器’。它也可以用来创建操作系统，因为所谓的操作系统也不过是一个跑在内核上的应用而已。可以把 Docker 想象成 LxC 的一个强化版；它提供了以下 LxC所不具有的特性：

- 强大的可移植性：你可以使用 Docker 创造一个绑定了你所有你所需要的应用的对象。这个对象可以被转移并被安装在任何一个安装了 Docker 的 Linux 主机上。
- 版本控制： Docker 自带 git 功能，能够跟踪一个容器的成功版本并记录下来，并且可以对不同的版本进行检测，提交新版本，回滚到任意的一个版本等功能等等。
- 组件的可重用性： Docker 允许创建或是套用一个已经存在的包。举个例子，如果你有许多台机器都需要安装 Apache 和 MySQL 数据库，你可以创建一个包含了这两个组件的‘基础镜像’。然后在创建新机器的时候使用这个镜像进行安装就行了。
- 可分享的类库：已经有上千个可用的容器被上传并被分享到一个共有仓库中（[http://index.docker.io/](http://index.docker.io/)）。考虑到 AWS 对于不同环境下的调试和发布，这一做法是十分聪明的。

如果对于 Docker 的功能想要有更加详尽的了解，请点击[这个链接](http://stackoverflow.com/questions/17989306/what-does-docker-add-to-just-plain-lxc)。

***

Chef makes use of a master-agent setup, in which a central master server coordinates and sends out instructions to a number of agents installed on all other client nodes. The agents can even be installed from the client side using the ‘knife’ tool and SSH for easy deployment. All this is managed by either a GUI or a CLI using base Ruby as the DSL. The GUI is useful, but for most advanced tasks it will be inevitable that one will have to learn Ruby. Chef and other CM tools utilize an** infrastructure-as-code model** – to work effectively all infrastructure nodes being managed must be expressible purely in code, in this case Ruby. You define the changes you want and the subset of machines you want those changes to be applied to, then the Chef server does the rest. For instance, if you need to upgrade all Windows 2008 servers with the .Net 2.0 framework to .Net 3.0, you will define this on the Chef server and it will be rolled out. One caveat though – Chef’s implementation of ‘push’ from master to agents is still sub-par; the crowdsourced recommendation is that you instead configure your agents to periodically ‘phone home’ to check if the master has any changes for them to deploy on their machines. Needless to say, this is a major annoying drawback that Chef’s creators are working to resolve.

Chef 是使用 主机-代理 结构，通过中心的主机服务器对于其他的数台安装在其他服务器节点上的代理进行协调和发送指令。这个代理（agent）可以通过使用轻量级工具和 SSH 在服务器端进行简易的部署。所有的管理都可以通过基于Ruby作为DSL而实现的 GUI 或者 CLI来进行。这个GUI是非常有用的，但是当你遇到很复杂的任务的时不可避免的需要学习 Ruby 。Chef 和其它的 CM 工具一样，使用 **结构-等于-代码** 的模式进行工作。这也就意味着所以的基础节点的管理必须通过 Ruby 代码来实现。你首先定义你需要的修改，之后选定你想要这些修改应用在哪些机器上，剩下的事情你只管交给 Chef 的服务器来做就好了。举个例子，如果你需要将所有 Windows 2008 服务器中的 .Net 2.0 升级到 .Net 3.0,你就可以把这个修改定义到你的 Chef 服务器上，之后 Chef 服务器将会进行回滚。有一点需要注意- Chef 对于主机到代理的推送的实现还是属于比较初级的阶段，许多人建议你周期性的进行检查看看主机节点是否在代理节点上实施了更改。毫无疑问，这个缺点是一直困扰着 Chef 开发者并且一直都想被解决的问题。

***

Chef configs are packaged into JSON files called ‘cookbooks’, and the software can run in either a client-server mode, called Chef-server, or standalone mode called ‘Chef-solo’. Part of Chef’s appeal is that a huge number of cookbooks have been created and shared freely on the net.

Chef 的配置被打包在 JSON 文件中并被称作 ‘cookbook’，并且这个软件运行在 ‘客户端-服务器’ 模式时叫做 Chef服务器，而当它运行在单一模式的时候被称作 ‘Chef-solo’。Chef 一部分吸引力是来自于大量的 cookbook 已经被创建并且被免费分享在网络上。

***
>译者觉得这段的 chef-server 和 chef-solo 翻译成 chef服务器和chef-solo不是很妥当，但是不知道怎么更改，请校队人员能给予意见，更改后这段注释会被删除
***

###Conclusion
###结论

So which is best, Docker or Chef? As with so many complex choices, the answer is “It depends.” Depends on what you want to achieve, and the level of trust you have in the tool – Chef is tried and tested, while Docker is still very new and even its creators hesitate to recommend it for production environments. Actually, Docker and Chef can even complement each other; Docker is used for quickly provisioning new servers, and Chef is used for rolling out the small, detailed changes to existing machines, a task for which Docker may be ill-suited. Alternatively, Docker would be ideal in a setup mainly requiring creation of new read-only server instances, in which the change consists of an entirely brand-new server. 

For a full, detailed analysis of Docker-Chef differences, see [this excellent article](http://blog.relateiq.com/why-docker-why-not-chef/).

那么 Docker 和 Chef 到底哪个更好呢？面对这么多复杂的选择，答案是 “看情况”。根据你所想要得到的效果和你对工具的信任程度，当 Docker 还是一个很新的项目且它的制作这也在犹豫是不是要让它加入到生产环境中的时候，Chef 这个项目就已经经历了很长时间的测试和考验。确实，Docker 和 Chef 可以相互补充，Docker 被用于快速配置新的服务器，而Chef被用于恢复一些机器上已经存在的改变，这一点是 Docker 无法做到的。另外，Docker 是创造新的只读服务器实例时的完美选择，其中包括创建一个完全全新的服务器的。

对于想要完全的，详细的分析 Docker 和 Chef 之间有什么不同的人，请看[这篇文章](http://blog.relateiq.com/why-docker-why-not-chef/)

###References
[https://phunehehe.net/docker-vs-chef-vagrant/](https://phunehehe.net/docker-vs-chef-vagrant/)
[http://blog.relateiq.com/why-docker-why-not-chef/](http://blog.relateiq.com/why-docker-why-not-chef/)
[http://stackoverflow.com/questions/16047306/how-is-docker-io-different-from-a-normal-virtual-machine](http://stackoverflow.com/questions/16047306/how-is-docker-io-different-from-a-normal-virtual-machine)
[https://groups.google.com/forum/#!topic/docker-user/NF86nnPMZ6k](https://groups.google.com/forum/#!topic/docker-user/NF86nnPMZ6k)
