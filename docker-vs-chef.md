#Docker VS Chef

![alt](http://resource.docker.cn/docker-vs-chef.jpg)

#####作者：[Steve Hall](https://twitter.com/stevehalltw)

#####译者：[Mike Ling](https://twitter.com/tiramisu1993)

***
从来没有人说：“我喜欢通过手动开启服务来浪费时间”。长期以来，系统管理员和开发人员在等待新服务被创建时只能无聊地摆弄指头来打发时间，这体验非常糟糕却也让人无奈。尽管虚拟化和云计算以及大规模运算已经取得快速发展，但新服务的创建效率却未与时俱进。



使用 [Docker](www.docker.com) 和 [Chef](http://www.getchef.com/) 吧。这些轻量级的工具非常易于使用，而且能给你的多服务器环境带来结构性改善。不过 Docker 和 Chef 的实现方法大不相同。接下来会一一介绍。

***

###它们是什么


Docker 发端于一个名为 dotcloud 的开源项目；随着编写者不断挖掘它的潜力，它迅速变成了一个炙手可热的项目。它由 GO 语言编写的，并且只支持 Linux。它基于 Linux 容器（LxC）来创建一个虚拟环境。请注意：虚拟环境 VE ( Virtual Environment ) 和 虚拟机（ VM ）很不一样。虚拟机是由虚拟工具或者模拟器（ HyperV 、 VMWare 等）创建的，是一个全镜像的主机源，其中包括操作系统、硬盘调整、网络和虚拟进程。过于臃肿的结构吃掉了大量的硬盘空间同时拖慢了运行和开机速度。

#####>译者注：[LXC](https://linuxcontainers.org/) 是一个 Linux 提供的收容功能接口，通过 LXC 提供的 API 和简单的工具，使得 Linux 用户可以简单的创建和管理系统或者应用的空间。


Docker 通过自己与操作系统内核相分离的特点很好的解决了这个问题。它会首先创建一个与机器内核相分离的 “等级0” 包裹。任何之后的改变都被保存为 namespace 的快照，并且像虚拟系统那样在单独的容器内运行。“快照”只捕捉“基本模版”的改动，而不是整个机器上面的设置（不像是VM的快照功能）。从这一点我们可以看出 namespace 能够储存并在部署了 Docker 的主机中创建完整环境。

Chef 是一个 CM 工具。它和 [Puppet](http://puppetlabs.com/) 并列为 CM 工具系列中最常用组件。它最初在2009年发布，能够非常简便地对改变进行回滚、配置新机器，因此管理员和开发者使用它来管理机器环境。 Chef 完全支持 Windows 和 Linux 以及 Unix， 并且也有值得夸奖的 GUI （虽然不像 Puppet 的那么平滑）。不过也有人抱怨说 Chef 对于新手来说并不好理解，同时文档让使用者、特别是新手非常头疼。


#####>译者注：前文指的 CM 指的是 [Configuration management](http://en.wikipedia.org/wiki/Configuration_management) ，本意指是一个建立和保持生产连续性的生产模型；笔者认为此处是指[Comparison of open-source configuration management software](http://en.wikipedia.org/wiki/Comparison_of_open-source_configuration_management_software)---一系列开源的 Congratulation management 组件。
***

###它们是如何工作的

#### DOCKER

Docker 不会通过建立独有的操作系统、进程和对硬件进行模拟来创建属于自己的虚拟机。一台 VE 就像是轻量级的 VM ；**它在已有的内核关于底层硬件的镜像上**建立一个可以用来运行应用的‘容器’。它也可以用来创建操作系统，因为所谓的操作系统也不过是一个跑在内核上的应用而已。可以把 Docker 想象成 LxC 的一个强化版，只是了以下 LxC 所不具有的特性：

- 强大的可移植性：你可以使用 Docker 创造一个绑定了你所有你所需要的应用的对象。这个对象可以被转移并被安装在任何一个安装了 Docker 的 Linux 主机上。
- 版本控制： Docker 自带 git 功能，能够跟踪一个容器的成功版本并记录下来，并且可以对不同的版本进行检测，提交新版本，回滚到任意的一个版本等功能等等。
- 组件的可重用性： Docker 允许创建或是套用一个已经存在的包。举个例子，如果你有许多台机器都需要安装 Apache 和 MySQL 数据库，你可以创建一个包含了这两个组件的‘基础镜像’。然后在创建新机器的时候使用这个镜像进行安装就行了。
- 可分享的类库：已经有上千个可用的容器被上传并被分享到一个共有仓库中（[http://index.docker.io/](http://index.docker.io/)） 。考虑到 AWS 对于不同环境下的调试和发布，这一做法是十分聪明的。

如果希望更加详细地了解 Docker 的功能，请点击 [这里](http://stackoverflow.com/questions/17989306/what-does-docker-add-to-just-plain-lxc) 。

####CHEF

Chef 是使用“主机-代理”结构，通过中心的主机服务器对于其他的数台安装在其他服务器节点上的代理进行协调和发送指令。这个代理（ agent ）可以通过使用轻量级工具和 SSH 在服务器端进行简易的部署。通过使用基于 Ruby 的 DSL ，所有的管理可通过 GUI 或 CLI 实现。这个 GUI 非常有用，但是当你遇到很复杂的任务的时不可避免的需要学习 Ruby 。Chef 和其它的 CM 工具一样，使用 **结构-等于-代码** 的模式进行工作。这也就意味着所以的基础节点的管理必须通过 Ruby 代码来实现。首先定义你需要的修改，之后选定将这些修改应用在哪些机器上，剩下的事情交给 Chef 的服务器做就好了。举个例子，如果你需要将所有 Windows 2008 服务器中的 .Net 2.0 升级到 .Net 3.0 ，你就可以把这个修改定义到你的 Chef 服务器上，之后 Chef 服务器将会进行回滚。有一点需要注意： Chef 对于主机到代理的推送的实现还是属于比较初级的阶段，因此建议你周期性的进行检查看看主机节点是否在代理节点上实施了更改。毫无疑问，这个缺点一直困扰着 Chef 开发者，并且悬而未决。


Chef 的配置被打包在 JSON 文件中并被称作“ cookbook ”。这个软件运行在“客户端-服务器”模式时叫做 Chef-server ，而当它运行在单一模式的时候被称作“ Chef-solo ”。 Chef 的部分吸引力来自于大量的 cookbook 已经被创建并且被免费分享在网络上。


###结论

那么 Docker 和 Chef 到底哪个更好呢？面对这种复杂的选项，答案是“看情况”——根据你所想要得到的效果和对工具的信任程度进行选择。当 Chef 这个项目经历了长期测试和考验的时候， Docker 只是一个新兴项目，并且创始人还在犹豫是将其应用于生产环境中的时候。但是 Docker 和 Chef 可以相互补充，Docker 的优势在于快速配置新服务器，而 Chef 则能恢复一些机器上已经存在的改变，后者是 Docker 无法做到的。另外，Docker 是创造新的只读服务器实例时的完美选择，这其中包括创建一个全新的服务器的。

对于想要全面、详细分析 Docker 和 Chef 之间不同的读者，请阅读[这篇文章](http://blog.relateiq.com/why-docker-why-not-chef/) 。

###参考资料
- [https://phunehehe.net/docker-vs-chef-vagrant/](https://phunehehe.net/docker-vs-chef-vagrant/)
- [http://blog.relateiq.com/why-docker-why-not-chef/](http://blog.relateiq.com/why-docker-why-not-chef/)
- [http://stackoverflow.com/questions/16047306/how-is-docker-io-different-from-a-normal-virtual-machine](http://stackoverflow.com/questions/16047306/how-is-docker-io-different-from-a-normal-virtual-machine)
- [https://groups.google.com/forum/#!topic/docker-user/NF86nnPMZ6k](https://groups.google.com/forum/#!topic/docker-user/NF86nnPMZ6k)

***

#####这篇文章由 [Steve Hall](https://twitter.com/stevehalltw) 发表，[Mike Ling](https://twitter.com/tiramisu1993) 翻译。点击 [这里](https://www.scriptrock.com/articles/docker-chef/) 可查阅原文。

#####The article was contributed [Steve Hall](https://twitter.com/stevehalltw), translated by [Mike Ling](https://twitter.com/tiramisu1993). Please click [here](https://www.scriptrock.com/articles/docker-chef/) to read the original publication.
