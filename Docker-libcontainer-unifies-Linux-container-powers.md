## Docker 的 libcontainer 正在联合各种 Linux 容器的力量 ##

> 由 [Steven J. Vaughan-Nichols](http://www.zdnet.com/meet-the-team/us/steven-j-vaughan-nichols/) 发布在 [zndnet](http://www.zdnet.com/docker-libcontainer-unifies-linux-container-powers-7000030397/)

在旧金山举行的 [DockerCon](http://www.dockercon.com/) 中， [Docker](http://www.docker.com/) 的 CTO 及联合创始人 Solomon Hykes 宣布他们将会和其他之前在容器技术领域的竞争对手通力合作，共同开发 Docker 的关键开源组件 [libcontainer](https://github.com/docker/libcontainer)。

![](http://cdn-static.zdnet.com/i/r/story/70/00/030397/libcontainer-diagram-620x465.png?hash=AGV3ZTD3Lw&upscale=1)
libcontainer 与 Linux 服务交互图

Google、 [Red Hat](http://www.redhat.com/) 和 [Paraalels](http://www.parallels.com/) 现在都成为了这个项目的参与者， 他们将会和 Docker 一起作为这个项目的核心维护者，这对系统管理员、数据中心管理者和云架构师来说是一件十分重要的事情。 此外 Canonical Ubuntu 的容器工程师也将会加入这个项目中来。

换句话说， libcontainer 极有可能在将来成为标准 Linux 内核中的默认容器库。更有甚者，还有传言说[微软正在研发将依托于 Docker 的程序运行在 Azure 上的机制](http://www.zdnet.com/heres-how-microsoft-is-supporting-the-open-source-docker-container-model-7000030393/)。

Libcontainer 可以让容器通过一种一致可预测的方式使用 Linux 的 namespaces、 cgroups、 capabilities、 AppAromor 安全配置、 网络接口以及防火墙规则。并且 libcontainer 并不依赖于 [LXC](https://linuxcontainers.org/)、 [libvirt](http://libvirt.org/) 和 [systemd-nspawn](http://www.freedesktop.org/software/systemd/man/systemd-nspawn.html) 这些 Linux 用户空间组件。 Docker 声称[这样可以减少不可控的部分](http://blog.docker.com/2014/03/docker-0-9-introducing-execution-drivers-and-libcontainer)，将 Docker 从不同版本的 LXC 所带来的副作用中解放出来。

在一次邮件采访中 Parallels 的服务器虚拟化 CTO 以及 Linux 基金会的技术咨询主席  James Bottomley 说他们最终达成了一致，共同开发 libcontainer。 Libcontainer 将会向应用提供他们所需要的更细粒度的容器特征（希望能够帮助诞生下一代类似 Docker 的应用），并且可以使我们的工具有更好的兼容性，在不同的产品间无缝的迁移。举例来说，我们可以让 Docker 和 LXC 在 [OpenVZ](http://openvz.org/Main_Page) 甚至在云服务器产品上进行部署。

Libcontainer 是由 [Google Go](http://golang.org/) 语言编写的，同时他也有其他不同语言版本的实现。微软将可能会用 ASP.NET 进行实现。 Parallels 的 [libct](https://github.com/cyrillos/xemul-libct) 其中包含了 libcontainer 的功能，主要是由 C/C++ 以及 Python进行实现。

Bottomley 还声称他们（Parallels）将会重构代码，以便 Docker 的 Go 代码可以在底层调用 libct 代码。这样将会提供对 Go 和 C/C++/Python 共同的 API 以及执行路径。对于 Docker 来说他将会从 libct 中获得 checkpoint/restore 以及 动态迁移的能力。对于 Parallels 来说他们将能够在 OpenVZ 和 Parallels 云服务器上直接运行基于 GO 的 Docker。

Libcontainer 在不久可能会成为 Linux 的标准容器库。因为很多强大的公司正在开发并使用它。[ Red Hat 已经将它集成到刚刚发布的RHEL7中](http://www.zdnet.com/a-big-step-forward-in-business-linux-red-hat-enterprise-linux-7-arrives-7000030385/)， Google 也已经在将自己内部的容器迁移到 Docker。

Google 基础设施的 VP ‎Eric Brewer 在 DockerCon 上说[几乎所有的 Google 应用都在容器中运行](http://www.itworld.com/virtualization/416864/containers-bring-skinny-new-world-virtualization-linux)，并且将会在今后利用 Docker 和 libcontainer 作为 [Google Compute Engine](https://cloud.google.com/products/compute-engine/) 的容器。当企业将一项技术应用到最终客户而不只是程序员时，你就知道他们正在认真的考虑这件事情。
