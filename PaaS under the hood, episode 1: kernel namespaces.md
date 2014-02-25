# PaaS的底层实现1：内核命名空间
Posted on November 28, 2012 by Jérôme Petazzoni

把事情变得简单是一件繁重的工作。在dotCloud，我们把像部署，扩展Web应用程序这样超级复杂的事情，打包成对于开发者来说最简单的体验。但我们是如何做到这点的呢？从内核层级的虚拟化到监控，从高吞吐量的网络路由到分布式锁，从处理EBS问题到每分钟收集无数的系统指标……有人说，扩展一个PaaS是一个不可能的任务，对我们来说是这样么？继续往下看吧！

这个系列的文章探索了PaaS的架构和内部实现，包括了广义的PaaS和狭义的dotCloud。这是第一篇文章，我们会介绍名称空间（namespaces），这是一个dotCloud平台用来隔离应用程序的Linux内核的特性。

## Part 1: Namespaces

当我第一次认识 Linux Containers (LXC) 时，我有一个（非常错误的）印象——他们主要依赖于control groups（cgroups）。这是一个很容易犯的错误：当你每次创建一个新的container，比如叫“jose“，就会出现一个同样名字的cgroup，"/cgroup/jose"。但事实上，虽然cgroups对于Linux Container来说非常有用，但真正重要的底层支持是来自于namespaces。
