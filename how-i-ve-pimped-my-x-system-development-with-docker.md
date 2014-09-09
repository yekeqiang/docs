#我是如何使用 Docker 来协助 X 系统上的开发工作的

***

#####作者：[SEBASTIAN GĘBSKI](https://twitter.com/liveweird)

#####译者：[周亮](http://www.brightconan.com)

***
对应用进行设置可能会相当复杂，尤其是在 Linux 系统上。不同应用有不同的配置方法，它们会在不同的文件系统路径下（在不同的 Linux 发行版中，由于应用存在多种变种，这些路径也会有所不同）保存二进制文件和数据。一旦你把系统配置好了，就很难再恢复到之前的状态，尤其是你同时进行了一些其他修改的时候（比如，安装了一些其他应用程序）。这也是最近诸如 **Puppet**、 **Chef**、 **Ansible** 和 **Salt** 这样的部署工具流行的原因。但即使有了这些工具的帮助，创建 cookbook/recipe 也许也十分麻烦： Linux 系统并非以傻瓜化著称，系统本身也不能让你摆脱麻烦。

##业界出现了一个新的工具

很幸运的是，对于 Linux 运维人员来说，一个新的工具产生了，而该工具很有可能改变游戏规则：[Docker](http://www.docker.com/) ，一个开源的平台，能够以一种轻量级的方式打包应用程序以及它们的依赖。

###这到底意味着什么？

Docker 使你能够在 Linux 系统上对不同的应用程序进行隔离，在不同的上下文环境中运行这些程序（这些程序可能执行在一台物理机器上，也可能运行在不同的物理机器上），请记住，这一点非常重要：

- Docker 使用了底层的内核机制做到了资源隔离，而并不需要其他资源消耗型的虚拟化技术，[如果想了解更多细节，请点击这里。](http://www.infoq.com/news/2014/03/docker_0_9)

- Docker **镜像**（一些保存的快照）以及 Docker **容器**（运行时隔离应用程序的容器）与虚拟镜像相比，使用起来快多了。

- 容器十分灵活，你可以在容器里打包很多应用，你也可以只打包一个，同时你可以尽你所愿来运行容器。

- Docker 容器里运行着一些镜像，而这些镜像之上有一个层的概念，分层使得你能够很容易地构造你的应用程序（每次操作都可以很容易地回滚/前滚，你只需要简单地增加或是删除层，却不会对下面的层产生影响）。

- Docker 完美地支持了“**一次配置，到处运行**”的范式。

##在实践中 Docker 是如何工作的？

或者这么讲，至少对于我来说，它是这么工作的：

1. 我已经在我的本地 [Vagrant](http://www.vagrantup.com/) 环境中指定了 Docker 作为部署工具， Vagrant 从 **1.6** 版本就引入了这一功能。对于什么是 Vagrant ，以及它为什么是 X 平台开发人员必备的工具，我认为无需赘述。

2. Docker 已经为我自动下载了一些 Linux 发行版的镜像（这些镜像会被 Vagrant 使用，在 hypervisor 上运行）。

3. 现在我能够以至少两种方式创建我自己的容器（正在运行的，实现资源隔离的应用程序）：

 - 第一种方式是制作一个用命令配置好的 **Dockerfile** ，这个 Dockerfile 基于干净的 Linux 镜像来生成，同时这些命令也使用了非常简单的 DSL （领域特定语言）。这种方式是我比较喜欢的，而且确实实用。

 - 第二种方式是创建一个运行着终端的全新的容器，这样的话你就可以在终端上执行你自己的命令，来做到你自己想做的事情。

```
sudo docker build
...
or
...
sudo docker run -i -t <image_name> /bin/bash    
```

##有些重要的事情需要记住



1. 如果在 Dockerfile 里的命令执行完了，或者是通过 *run* 这个子命令运行的命令执行完了（因为它们并不是 daemon 程序），那么容器就会关闭并且消失！

2. 当容器正在运行时，你可以十分方便地：

 - 查看容器中命令的输出（*docker logs*）
 - 挂载到运行的容器（*docker attach*）
 - 列出容器内文件系统的实际变化（记得不同的容器并不能看到其他容器的变化！）（*docker diff*）
 - 暴露并且映射容器中的端口（比如，如果你正在搭建一个应用程序，而该应用程序在容器内已经有了相应的端口）（在 dockerfile 中的 *EXPOSE* 选项，Docker 命令的 *-p* 选项）

3. 如果你想要通过手动执行命令的方式创建你自己的容器（*run*），你需要存储你自己的镜像 - 首先你需要在正在运行的容器列表里找到你想要的容器，然后执行commit命令：

```
sudo docker ps
sudo commit <container name>
```

##通过使用以上所有的选项...

...我能够：

- 使用应用组件的**任何**组合来组成我的本地开发环境，添加或者删除一个运行时组件就如同开启或者关闭容器那么简单。一旦我关闭了某个应用组件，那么它也从文件系统中完全消失了。

- 非常方便地（回滚/前滚）创建出独立的，隔离的应用容器，同时又不像虚拟镜像那样会耗费许多时间，也没有不必要的操作系统开销。

- 实验一系列很有意思的事情，却几乎不会冒重头来过的风险（*有鉴于此，Chef recipe 已经过时了...*）。

...在我做到以上所有事情的同时，我的操作系统却坚如磐石：我不会破坏任何事情，我可以很简单地回滚我做的任何操作。安装其他的容器也不会互相影响。

总之，在几周的时间里，我已经把 Docker 作为了我的主要的软件开发工具。现在我已经无法想象缺少了 Docker 我该怎样进行基于 JVM 的开发工作了

***

#####这篇文章由 [SEBASTIAN GĘBSKI](https://twitter.com/liveweird) 撰写，[周亮](http://www.brightconan.com) 翻译。点击 [这里](http://no-kill-switch.ghost.io/how-i-ve-pimped-my-x-system-development-with-docker/) 阅读原文。

#####The article was contributed by [SEBASTIAN GĘBSKI](https://twitter.com/liveweird), translated by [周亮](http://www.brightconan.com), click [here](http://no-kill-switch.ghost.io/how-i-ve-pimped-my-x-system-development-with-docker/) to read the original publication.
 
 
