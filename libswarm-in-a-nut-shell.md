#Libswarm 简明介绍

#####作者：[Brian Goff](https://twitter.com/cpuguy83)

#####译者：[刘梦馨](http://weibo.com/oilbeater)

***

Libswarm 提供了一组通用的 API 可以将原先相互分离的各种工具和服务组合起来。 Swarmd 可以将这些 libswarm 服务像 unix 下的管道那样相互连接。我个人更喜欢把他当做一种中间件链条。

如果你想更多了解 libswarm，可以查看 [github](https://github.com/docker/libswarm)，并且加入我们的 freenode 频道 #libswarm。感谢 @markwrenn 的图示。

----------

在 Dockercon 上，Docker 宣布了一项正在进行中的新项目 libswarm 。我想澄清一下 libswarm 是什么，它能做什么，又不能做什么。

首先，libswarm 自己并不是一个流程协调工具，而且也不会取代任何流程协调工具。

Libswarm 首先是一个库，不会是一个最终用户的工具。它能够帮助你把各个不同的分离工具，包括流程协调工具组合起来。

我看到很多 Docker 的核心功能被分解成一个个小的 libswarm 服务，他们组合起来就是 Docker 。我看到一些工具，能够 hook 了 libswarm 的 API 来扩展原生的 Docker 功能，并且不需bind-mounting Docker 的 socket（这个功能相当危险）。Libswarm 的 API 会帮助你通过一种非传统 REST API 的方式（可能是别的形式）与 Docker 交互。

Swrmd 是 Libswarm 实现的一个参考标准。

Swarmd 是一个概念验证的二进制文件，通过运行它，你可以像 unix 管道那样将多个服务连接起来。

**Swarmd 的语法还不完整，并且可能会改变。**

在 [Libswarm](https://www.github.com/docker/libswarm) 项目中，你将会看到一个叫做 “backends” 的文件夹。里面存放着各种文件，例如如何与 libswarm API 进行交互；如何与诸如 AWS 、 Rackspace 、 Orchard 、 shipyard 这样的外部服务交互；如何与二进制文件交互；甚至是如何拦截 libswarm 自身消息从而以某种方式进行控制。

最基础的 swarmd 设置是这样的：


    ./swarmd 'dockerserver unix:///var/run/docker.sock' 'dockerclient tcp://1.2.3.4:2375'

这将会开启一个 Docker REST API ，实现 “dockerserver” 服务。 dockerserver 接受 HTTP 请求并且从中生成 libswarm 消息。 “dockerclient” 接受 libswarm 消息并且将他们传送给 docker 的守护进程（目前用的是 REST API ）。这个操作不多，但能让你在本地运行一个 docker 客户端并与 /var/run/docker.sock 进行通信，并将这些请求发送到 1.2.3.4:2357 的 docker 守护进程。

{<1>}![alt](http://resource.docker.cn/techd-libswarm.png)

你也可以进行下面的操作：

    ./swarmd 'dockerserver unix:///var/run/docker.sock' 'aggregate "dockerclient tcp://1.2.3.4:2375" "dockerclient tcp://1.2.3.5:2375" "dockerclient tcp://1.2.3.6:2376"'

这和第一个例子很像，但是他利用了 “aggregate” 服务来和各个 “dockerclient” 服务通信，并将结果聚集起来。你可以在这里使用 “docker run” ， “aggregate” 将会选择一个 “dockerclient” 来生成一个新的容器。

你可以用 “docker ps” 来获得所有运行中容器的信息。目前它的功能主要是 demo 。

但是想象一下这个：

    ./swarmd 'dockerserver unix:///var/run/docker.sock' 'mesos "dockerclient tcp://1.2.3.4:2375" "dockerclient tcp://1.2.3.5:2375" "dockerclient tcp://1.2.3.6:2376"'

目前没有 mesos 的后端（请帮助我们）。如果有的话，我们就可以把 “docker run” 和 Mesos 连接起来，这样就可以让 “dockerclient” 遵循 Mesos 的标准来获得可用性。

你也可以将 Mesos 换成 Kubernetes 等其他调度器。

你可以连接任何你想要的服务：

    ./swarmd 'dockerserver unix://var/run/docker.sock' 'serviceA' 'serviceB' 'serviceC' 'dockerclient tcp://1.2.3.5:2375'

这之中的任何一个服务都可以截获请求，按一定规则完成操作并沿着链条将请求传送，或者调用外部服务进行操作再在沿着链条传递。

只要这些被调用的服务实现了 libswarm API ，同时能够正确地 发送/接收 libswarm 消息，就可以加入这个链条。

在上述例子中，我用 “dockerserver” 作为前端， “dockerclient” 作为后端，主要是为了便于说明，并且实际中也是这样使用的。

技术上来说， Libswarm 和 Docker 是完全独立的，它可以脱离 Docker 存在。

我必须提到， libswarm 利用 [libchan](https://www.github.com/docker/libchan) 来进行通信。

Libswarm 目前还在初步的开发中，很多 API 都还没有固定下来。

***

#####这篇文章由 [Brian Goff](https://twitter.com/cpuguy83) 撰写，[刘梦馨](http://weibo.com/oilbeater) 翻译。点击 [这里](http://www.tech-d.net/2014/07/03/libswarm/) 阅读原文。

#####The article was contributed by [Brian Goff](https://twitter.com/cpuguy83), click [here](http://www.tech-d.net/2014/07/03/libswarm/) to read the original publication.
