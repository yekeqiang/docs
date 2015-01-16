# 对于 OpenStack ，Docker 1.0 意味着什么

![alt](http://resource.docker.cn/business-org-chart-1.png)

##### 作者：[Jason Baker](https://twitter.com/jehb)

##### 译者：[Peter Zhang](https://github.com/duobei)

***
门票售罄的 [Docker大会](http://www.dockercon.com/) 发布了许多重大公告，其中最引人注目的是发布 Docker 1.0 。


尽管对于它现在能否满足每个产品工作量的需求存在争议，然而毫无疑问的是这个里程碑版本的发布，是 Docker 进入数据中心的重要一步。

Docker 究竟是什么呢？Docker 是一个 Linux 容器平台，为开发者和系统管理人员设计，能使开发和部署分布式应用变得简单。 Docker 打包一个应用的所有部分，工具、配置文件、库等等，使之成为一个更简单的任务。概念上讲，它有点像个虚拟机，允许多个应用使用单个强劲机器，同时保持每个应用各自不同的具体配置，不会干扰其他应用。与虚拟机不同的是，应用原生地运行在 Linux 内核下，每个应用与其他应用隔离，在操作系统下面也隔离。想了解更多关于 Docker 的知识，可以点击下面的视频学习。

<embed src="http://player.youku.com/player.php/sid/XNzI3ODg2NTM2/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

容器超级赞。他们快速、高效、易用、轻量。容器会替代传统的虚拟化么？有些会，有些不会。容器是开发新应用和移植老应用的一个很棒的选择。但是这个世界依旧会运行许多传统应用，这些应用永远不会被运行在一个 Linux 容器里，或是因为应用的特定要求，或是因为维护现存支持协议的需要。与容器不同，虚拟机提供了运行非  Linux 宿主的能力，这可能是某个应用的必要条件。不过这应该不会打消你的热情，在不远的将来， Docker 和 Linux 容器会成为应用大规模部署的重要部分。

Docker 1.0 版本的发布带来了许多改进，为开发者和系统管理人员能够平滑过渡做好了准备。例如，极大改进了网络部分，在不需要桥接主操作系统的情况下，容器就可以直接连接到主网络界面。能够与 [SELinux](http://opensource.com/business/13/11/selinux-policy-guide) 很好地协作，允许更好的安全实现。当然了，随着新版本的发布，许多 bug 已经被修正。

Docker 即将成为 OpenStack 管理员的重要工具，与传统的虚拟机一起在 OpenStack 集群中工作。 Linux 容器要么通过 Heat 独立的启动，进行配置和编配的本地开发；要么通过 Nova 启动，借助一种专门的驱动，把容器作为另一种类型的管理程序来处理。哪种方法最优，取决你的实际用例。

想了解更多 OpenStack 和 Docker 如何合作的信息吗？请观看来自上个月亚特兰大 OpenStack 峰会的会议视频，包含了一个简短的概念介绍和一些部署的最佳实践。

<embed src="http://player.youku.com/player.php/sid/XNzI3ODg2NzY0/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

***

##### 这篇文章由 [Jason Baker](https://twitter.com/jehb) 撰写， [Peter Zhang](https://github.com/duobei) 翻译。点击 [这里](http://opensource.com/business/14/6/docker-and-openstack) 阅读原文。

##### The article was contributed by [Jason Baker](https://twitter.com/jehb) , click [here](http://opensource.com/business/14/6/docker-and-openstack) to read the original publication.
