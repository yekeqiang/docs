# What Docker 1.0 means for OpenStack (对于 OpenStack ，Docker 1.0 意味着什么) #
>注: 原文地址[What Docker 1.0 means for OpenStack](http://opensource.com/business/14/6/docker-and-openstack), 作者[Jason Baker](http://opensource.com/users/jason-baker)(Red Hat)

![ ](http://opensource.com/sites/default/files/styles/image-full-size/public/images/business/BUSINESS_orgchart1.png?itok=yKJbIJpA)

本周的周一和周二门票都卖光的[ Docker 大会](http://www.dockercon.com/)上，有许多大的公告。但是，其中最大的公告是 Docker 1.0 的发布。它现在是否满足每一个产品工作量的需求是一个倍受热议的话题，然而毫无疑问的是这个里程碑是使 Docker 进入数据中心重要的一步。

Docker 究竟是什么呢？Docker 是一个 Linux 容器平台，为了使开发者和系统管理人员开发和部署分布式应用变得简单而设计的。Docker 打包一个应用的所有部分，工具、配置文件、库、等等，使之成为一个更简单的任务。概念上讲，它有点像个虚拟机，允许分割一个单一的强劲机器使多个应用来共享这一机器，在各自不同的具体配置要求下，而且不允许这些应用干扰其他应用。不像虚拟机的部分是，应用原生的运行在 Linux 内核下，只不过小心的把每个应用同其他应用分离开，在操作系统下面也分离开。想了解更多么？下面这个简短的视频会给你一些额外的背景。

[What is Docker?](https://www.youtube.com/watch?v=ZzQfxoMFH0U)

容器超赞的，他们快速、高效、易用、轻量。容器会替代传统的虚拟化么？好吧，有些会，有些不会。容器是开发新应用和移植老应用的一个很棒的选择。但是这个世界依旧会运行许多传统应用，这些应用永远不会被运行在一个 Linux 容器里，因为具体的应用要求，或者因为维护现存支持协议的需要。不像容器，虚拟机提供了运行非 Linux 宿主的能力，这应当是这种应用的必要条件。但这些应该不会打消你的热情，认为 Docker 和 Linux 容器不久将来会成为衡量一个应用的重要组成部分的热情。

Docker 1.0 版本的发布带来了许多改进，为开发者和系统管理人员能够平滑过渡做好了准备。例如，极大改进了网络部分，不需要桥接主操作系统的情况下，容器现在就可以直接连接到主网络界面。能够与[SELinux](http://opensource.com/business/13/11/selinux-policy-guide)很好的协作，允许更好的安全实现。当然了，随着新版本的发布，许多 bug 已经被修正。

Docker 即将成为 OpenStack 管理员使他们自己熟悉 OpenStack 的重要工具，随着在 OpenStack 集群中它逐渐开始支持传统的虚拟机。Linux 容器要么通过 Heat 独立的启动，Heat 是用来配置和编排开发选项的，要么通过 Nova 启动，利用一种专门的驱动，把容器作为另一种类型的管理程序来处理。对你来说那种方式运行的最好取决于你的具体用例。

想了解更多关于 OpenStack 和 Docker 如何能一起合作么？请观看来自上个月亚特兰大 OpenStack 峰会的会议视频，包含了一个简短的概念介绍和一些部署的最佳实践。

[Practical Docker for OpenStack](https://www.youtube.com/watch?v=vSukPe5O7HM)
