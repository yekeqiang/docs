#微软如何支持开源的 Docker 容器模型

#####作者：[Mary Jo Foley](http://www.zdnet.com/meet-the-team/us/mary-jo-foley/)

#####译者：[Bo Wen](https://github.com/iambowen)

***
Docker，这个自动化应用部署的开源引擎，在本周发布了 1.0 的里程碑版本。

云服务提供商，包括微软、 IBM 、 Rackspace 、 Google 以及其他主要的 Linux 提供商如 Canonical 和 Red Hat ，都开始支持 Docker 。

![alt](http://resource.docker.cn/cloud-linux.png)

[Docker 使用容器来替代虚拟机](http://www.zdnet.com/docker-1-0-brings-container-technology-to-the-enterprise-7000030333/) ，从而可以使多个应用同时在同一个服务器上运行，我在 ZDNet 的同事 Steven J. Vaughan-Nichols 就是这样给我解释的。 Docker 给开发人员以及系统管理员提供了一个平台，让配置和部署分布式应用变得更加容易。

Docker 的测试者已经可以 [在 Azure 的 Linux 虚拟机上尝试预览版的 Docker](http://azure.microsoft.com/blog/2014/06/09/docker-and-azure-coolness/) 。在的 Dockercon 的展示中，微软的代表演示了使用 [Azure虚拟机扩展](http://www.zdnet.com/microsofts-azure-cloud-team-moves-toward-blurring-the-iaaspaas-lines-7000026708/) ，用 Docker 向 Azure 的 Linux 虚拟机做部署。

在 Azure 上运行时， Docker 和微软的跨平台 Azure 命令行工具集成在一起，让用户能更加容易的在 Azure 上启动 Docker 。用户不需要单独的登陆 Azure 上每个 Docker 主机；相反，他们在自己的桌面或者笔记本上为每个使用 Docker 客户端的主机运行配置命令。

微软官方已经将源码开放，并承诺 [将 Azure 命令行工具（ CLI ）合并回主工程](https://github.com/MSOpenTech/azure-sdk-tools-xplat/tree/docker) 。他们同时还致力于为 Docker 提供更多的教程和信息，推动项目前进。

有消息说下一个版本的 ASP.NET 框架，工程代号 Project K ，最终会被微软变成构建“ .Net的Docker ”，下一版的 ASP.NET 将允许用户通过基于应用-应用部署自选的 .Net 框架的版本。我得到的消息是， ASP.NET v.Next 团队正专注于让下一个版本的 ASP.NET 更容易在 Docker 容器中打包。


在未来更长的时间里，微软会持续的研究应用交付的新方法，包括使用库 OS 模型。 [微软研究院的 Drawbridge 项目](http://www.zdnet.com/microsoft-to-offer-its-drawbridge-virtualization-technology-on-top-of-its-windows-azure-cloud-7000009774/) 正在推进不需要虚拟机来进行虚拟化的方法。

微软是否或者何时产品化 Drawbridge 还没有定论，但是去年在微软的招聘网站上发布了一则招聘信息中提到 [Drawbridge 已经部署在 Windows Azure 上](http://www.zdnet.com/microsoft-to-offer-its-drawbridge-virtualization-technology-on-top-of-its-windows-azure-cloud-7000009774/) 了。

***

#####这篇文章由 [Mary Jo Foley](http://www.zdnet.com/meet-the-team/us/mary-jo-foley/) 撰写， [BOWEN](https://github.com/iambowen) 翻译。点击 [这里](http://www.zdnet.com/heres-how-microsoft-is-supporting-the-open-source-docker-container-model-7000030393/) 阅读原文。

#####The article was contributed by [Mary Jo Foley](http://www.zdnet.com/meet-the-team/us/mary-jo-foley/), click [here](http://www.zdnet.com/heres-how-microsoft-is-supporting-the-open-source-docker-container-model-7000030393/) to read the original publication.
