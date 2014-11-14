# 揭秘微软的 Container 技术之 ——  Drawbridge

---

##### 作者：周晓璐

---


> 摘要：有人说 Windows Server 只有拥有了自己的 Container 技术才能继续保持与 Linux 的抗衡。但微软对于其 Container 技术一直没有明确的发布，只能从一些公开的演讲中，搜集到一些信息。

在构建软件定义的数据中心方面，VM 技术已逐渐显露出力不从心，Container 技术作为后继者慢慢崭露头角。 Container 技术由来已久，但开源技术 Docker 的出现，为开发者以微服务的形式构建可移植的应用，提供了标准。随着 Docker 的成熟，容器已经能够满足应用的可移植性、自动性、编排和扩展性。

作为一种开源的 Linux Container 技术，Docker 已经得到IBM、Google、RedHat、VMware和微软等多个公司的支持，这些公司纷纷宣布在自己的操作系统、虚拟机或云平台中支持 Docker 。微软更是在5月份宣布，在其 Azure 云服务的 IaaS 组件中可运行基于 Linux 的 Docker 容器。至于在 PaaS 服务中可用，应该只是时间问题了。 Azure 云服务部门 CTO Mark Russinovich 在9月份公开表示：“我们也在考虑这个问题，许多 Azure 云的 PaaS 用户已经有了这个需求。”

Russinovich 确认了正在将其代号为“ Drawbridge ”的 Container 技术商业化的计划， Drwbridge 基于库操作系统（library OS），library OS 由微软研究院 Galen Hunt 在2008年发起。

“基于VM的虚拟化技术其效率不高，而传统的OS虚拟化技术，如 Linux Container 技术，安全性又不足，所以 Drawbridge 选择了一条两者兼顾的道路，把 Kernel 的内存状态放到了 Container 中（这里指 Windows 的 Container ），用户间的隔离更彻底，而各个 Container 之间依然共享一部分资源，所以相较 VM 虚拟化效率要高。”曾经负责过 Drawbridge 项目的前美国微软首席开发经理左玥告诉笔者。

## Drawbridge的基石：库操作系统（library OS）

在 [Galen 2011年发布的一篇论文](http://research.microsoft.com/pubs/141071/asplos2011-drawbridge.pdf)中，详细说明了win7库操作系统中运行 Excel 、 PowerPoint 和 Internet 的工作原型。 library OS 的想法是，一个应用所依赖的操作系统的特性，会体现在应用的地址空间上。一个连接宿主机操作系统内核和库操作系统的小抽象集合，提高了系统安全性，也使得系统各组件能够更快地改进。

在论文中，详细描述了一个 win7 library OS 的工作原型，其中运行了诸如 Microsoft Excel 、 PowerPoint 和 Internet Explorer 等常用应用。证明了通过对网络协议的重用，可以实现在各个独立、安全隔离的库操作系统实例的桌面共享。每个实例的开销相比全虚拟化要小得多，一个典型的应用只会增加16M的工作集和64M的磁盘空间。在库操作系统下面提供了一个新的ABI（应用二进制接口），保证了应用的移动性。我们也证明了只花费很小的开销，就可以达到当前很多硬件虚拟化的功能。

库操作系统会精简操作系统到固件层，将重点放在 API 和应用交付层面，而不是低层次的服务。定义了3种 OS 服务，包括：硬件服务、用户服务和应用服务。硬件服务包括了操作系统内核和硬件驱动；用户服务包括了 GUI shell 和桌面、剪切板、索引器等；应用服务包含了 API 实现，包括框架、渲染引擎、通用的UI控制等。

在 Drawbridge 中运行的应用可以访问Windows的核心特性和增强版的API，包括 .NET CLR和 DirectX 。虽然被严格地隔离开来，但 Drawbridge 中的应用依然可以共享资源，包括屏幕，键盘，鼠标和用户剪切板。

![alt](http://resource.docker.cn/windows-7-os-architecture.jpg) 

![alt](http://resource.docker.cn/drawbridge-architecture.jpg) 
  
在八月份的 TechMentor 大会 Keynote 上，Redmond 的专栏作家 Don Jones 曾发表演讲讨论过库操作系统的话题。 Jones 说：“我们通常都将开发人员的开发理解为针对某一系统的开发，比如 iOS 开发、 Android 开发、 Windows 开发等，但这是不准确的，他们应该是在针对一种运行环境或一组 API 做开发。而这组 API 再关联相应的操作系统。”

## Drawbridge VS Docker

在纽约举行的 Interop 大会上， Russinovich 宣告了 Drawbridge 依然在使用。虽然他没说 Windows 的计划，也没有明确表示 Drawbridge 会被加到 Windows Server 和 Hyper-V 中。但可以肯定的是 Drawbridge 在Windows Server 和 Azure 的工作已经在进展中了。 Russinovich 说在微软新的基于 Azure 的机器学习技术中，已经使用了 Drawbridge 容器技术。

“显然对虚拟化技术的加速已经不能满足我们的需求，所以我们借助了微软研究院的 Container 技术 Drawbridge ，这是一项我们内部一直在用的技术，我们正在试图将其公开化。”

虽然微软 Azure 也高调宣称了其对 Docker 的支持，但从 Russinovich 的态度看来，其将 Drawbridge 作为容器技术的优先选择，不断强调 Drawbridge 在部署微服务方面更安全。

Russinovich 说：“在一个多租户的环境中，必然会有很多未知来源的第三方代码运行在同一个平台上，你需要为他们设立安全屏障。大多数云平台使用虚拟化技术来实现，而通过一个更小粒度的安全容器，能够更高效地实现，这就是 Drawbridge 的设计初衷。”

左玥告诉笔者，其实微软自身的 Container 技术与 Docker 并不矛盾，Docker 是将 Linux 的 Container 技术标准化的工具，同样 Windows 的 Container 技术也可以使用 Docker 来管理。 

## Windows Server的Container技术？

Sam Ramji ，是 Apigee（一家提供基于云的 API 服务提供商）的 VP ，5年前离开微软时，是新兴开源与 Linux 策略部门的头。在采访中，他认为 Windows Server 只有拥有了自己的 Container 技术才能继续保持与Linux的抗衡。 

虽然不知道 Server 团队的头是怎么想的，但可以肯定的是他们也已经开始了 Container 技术方面的投入，预计在下一个 Server 版本中就会添加对 Container 的支持。左玥告诉笔者， Drawbridge 只是微软的一种 Container 技术，至于未来 Windows Server 的 Container support 是否基于 Drawbridge 还有待观察。

## 老大开源，老二怎么玩？

以前的IT界，多是老大闭源，老二开源，现在老大开源了，闭源怎么玩呢？先不说去抢 Linux 的生意，首先 Windows 目前的用户是否对 Container 技术有这么强的需求呢？

[Ubuntu也刚刚推出了自己的LXD](http://www.zdnet.com/ubuntu-lxd-not-a-docker-replacement-a-docker-enhancement-7000035463/)，并且还强调并不是要替代 Docker ，而是作为 Docker 的补充。有幸在微信群中看到了大牛们讨论：有人说 LXD 就是要替代Docker，因为 Docker 的初衷本来就是 LXC+RESTful API ；还有人说 Docker 让容器更像进程，而 LXD 是让容器更像虚机。老外不是一直反对“重复造车轮”的吗？你怎么看？

（感谢前美国微软首席开发经理左玥对本文的审校和帮助！）

相关链接：

- [Windows 'Drawbridge' Container Tech Sets Stage for Docker Battle](http://redmondmag.com/blogs/the-schwartz-report/2014/10/windows-drawbridge-container.aspx)

- [Why IT Pros Should Prepare for Microsoft's Stealth Library OS](http://redmondmag.com/blogs/the-schwartz-report/2014/08/microsoft-stealth-library-os.aspx)

---

原文地址；[揭秘微软的 Container 技术之 —— Drawbridge](http://www.csdn.net/article/2014-11-13/2822624-Drawbridge)
