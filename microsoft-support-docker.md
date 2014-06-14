
# 微软如何支持开源的Docker容器模型

> 注: 原文作者[Mary Jo Foley](http://www.zdnet.com/meet-the-team/us/mary-jo-foley/)，发表于[All About Microsoft](http://www.zdnet.com/blog/microsoft/)

Docker，这个自动化应用部署的开源引擎，在本周到达了1.0的里程碑。

云服务提供商，包括微软，IBM，Rackspace，Google以及其他主要的Linux提供商如Canonical和Red Hat，都开始支持Docker。

![model](http://cdn-static.zdnet.com/i/r/story/70/00/030393/azuredocker-200x101.png?hash=ZwMyZmxkMQ&upscale=1)

[Docker使用容器来替代虚拟机](http://www.zdnet.com/docker-1-0-brings-container-technology-to-the-enterprise-7000030333/), 从而可以使多个应用同时在同一个服务器上运行，我在ZDNet的同事Steven J. Vaughan-Nichols就是这样给我解释的。Docker给想要配置和部署分布式应用变得更加容易的开发人员以及系统管理员提供了一个平台。

Docker的测试者已经可以在[Azure的Linux虚拟机上尝试预览版的Docker](http://azure.microsoft.com/blog/2014/06/09/docker-and-azure-coolness/)。但就在本周的Dockercon的展示中，微软的代表演示了使用[Azure虚拟机扩展](http://www.zdnet.com/microsofts-azure-cloud-team-moves-toward-blurring-the-iaaspaas-lines-7000026708/)用Docker向Azure的Linux虚拟机做部署。

在Azure上运行时，Docker和微软的跨平台Azure命令行工具集成在一起，让用户能更加容易的在Azure上启动Docker。用户不需要单独的登陆Azure上每个Docker主机;相反的，他们在自己的桌面或者笔记本上为每个使用Docker客户端的主机运行配置命令。

微软官方已经将源码开放，并承诺将[公司的Azure命令行工具(CLI)合并回主工程](https://github.com/MSOpenTech/azure-sdk-tools-xplat/tree/docker)。他们同时还致力于为Docker提供更多的教程和信息，推动项目前进。

有消息说下一个版本的ASP.NET框架，工程代号Project K，最终会被微软变成构建“.Net的Docker”，鉴于下一版的ASP.NET将允许用户通过基于应用-应用部署自选的.Net框架的版本。但是这不尽然。我得到的消息是，ASP.NET v.Next团队正专注于将下一个版本的ASP.NET更容易在Docker容器中打包。


在未来更长的时间里，微软会持续的研究应用交付的新方法，包括使用库OS模型。[微软研究院的Drawbridge项目](http://www.zdnet.com/microsoft-to-offer-its-drawbridge-virtualization-technology-on-top-of-its-windows-azure-cloud-7000009774/)正在推进不需要虚拟机来进行虚拟化的方法。

微软是否或者何时产品化Drawbridge还没有定论，但是去年在微软的招聘网站上发布了一则招聘信息中[提到了Drawbridge已经部署在Windows Azure上了](http://www.zdnet.com/microsoft-to-offer-its-drawbridge-virtualization-technology-on-top-of-its-windows-azure-cloud-7000009774/)。
