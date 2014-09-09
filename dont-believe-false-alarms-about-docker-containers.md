#别轻信与 Docker 容器有关的虚假警告

#####作者：[David Linthicum](https://twitter.com/DavidLinthicum)

#####译者：[qwe12qwe19](http://blog.csdn.net/qwe12qwe19/article/details/38303665)

***

不管别人怎么说， Docker 容器仍然是必需的选择。

随着Docker逐渐重新升温，一些人表示，这个实现了云计算工作负荷的可移植性和管理的方案可能有一个致命弱点。

由于 Docker 容器使用与 Linux 一致的实现方法，这也导致大规模漏洞的出现，影响运行在服务器上的每个容器里的操作。特别是当底层操作系统出现故障时，所有的容器化的工作也会随之沉没。

有人在把它与使用管理程序相比较，并且认为 VMs 可能是管理云工作量的更好的方法。这简直就是对 Docker 引以为傲的优势——用轻便的容器来代替臃肿的 VMs ——打脸。


不要认为 Linux 操作系统级别的失败会威胁到 Docker 容器的可靠性。事实上 Docker 采取了很多防护措施来应对失败：

- Namespace 能够隔离容器的所有进程，包括别的容器和主机操作系统。每个容器都有自己的网络栈区。如果有个容器失败了，别的容器仍然能与其它容器和主机操作系统对话。

- Docker Cgroups 控制资源，为隔离做准备。 Cgroup 不仅保证容器分享底层资源，同时还通过观察确保单独的容器不会耗尽所有资源，拖垮其它的。

- 如果发生问题， Docker 容器会自动重启，也能提供额外的选择。你也能设计系统以分散对系统依赖，这能进一步减少单个操作系统失败导致容器被杀掉的风险。

核心观点就是容器比虚拟机更快，消耗资源也更少，前提是用户愿意专一使用单个平台，为所有容器提供共享的操作系统。下面列出了容器的一些真正的优势：

- 完整的虚拟机需要数分钟来创建和启动，而容器只需几秒就可以完成初始化。
- 与运行在虚拟机上应用相比，容器能为容器内的应用提供更好性能。前者还需要额外通过系统管理程序来运行。

正是轻量级的架构优势和更出色的性能表现让 Docker 大受欢迎。

虽然仍有瑕疵，但是 Docker 也才不过是 1.0 版本，我们无需杞人忧天。

*我的同事 Jonathan Baier 对本文有贡献。*


这篇文章最初发表在 [InfoWorld.com](http://www.infoworld.com/) 上。欢迎阅读 David Linthicum 的 [云计算博客](http://www.infoworld.com/blogs/david-linthicum?source=footer) 了解云计算的最新发展情况。你也可以在 Twitter 上关注 [InfoWorld](https://twitter.com/infoworld) 。

***

#####这篇文章由 [David Linthicum](https://twitter.com/DavidLinthicum) 撰写，[qwe12qwe19](http://blog.csdn.net/qwe12qwe19/article/details/38303665) 翻译。点击 [这里](http://www.infoworld.com/d/cloud-computing/dont-believe-false-alarms-about-docker-containers-246692) 阅读原文。

#####The article was contributed by [David Linthicum](https://twitter.com/DavidLinthicum) , click [here](http://www.infoworld.com/d/cloud-computing/dont-believe-false-alarms-about-docker-containers-246692) to read the original publication.
