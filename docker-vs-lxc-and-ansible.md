# Docker vs LXC/Ansible?
---

### 为什么有这个问题？

[@GrzegorzNosek](https://twitter.com/GrzegorzNosek)问了这样一个好问题: 为什么要选择Docker来替代LXC/Ansible?
在最近一期DevOPS交流会上.

老实说我已经尝试回答这个问题有一段时了，我确实回答一些(包含在我的一次演讲在交流会上[http://www.slideshare.net/d0cent/docker-rhel](http://www.slideshare.net/d0cent/docker-rhel)); 它是关于开发者在docker上开发是如此的容易。

但是针对我自己，为什么会用docker呢? 因为我是一个sysadmin(系统管理人员)，我喜欢更底层的东西，因此LXC对于我来说
更自然的做事方式。

### 你的脸，你的屁股？有什么区别?

(如果你对于这个标题感到感概或者厌恶，请回到18年前，想一想[毁灭公爵](https://en.wikiquote.org/wiki/Duke_Nukem))

有一件关于我的事情，你应该知道，我正在为FedoraProject做贡献；最近我在捣鼓一个项目 Fedora-Dockerfiles project(https://git.fedorahosted.org/cgit/dockerfiles.git/). 我做它是为好玩，但是我也想学习更多的关于Dokcer, 当我和一些朋友
开启一些开源项目时，必须找到一种简单的方式去管理我们的开发环境。在这种情况下，答案就是Docker。

当我用docker为那些根本不懂DevOPS/SysOPping准备开发环境时，写Dockerfiles是一件如此有趣的事(有些时候有些大坑的)。LXC如何呢？
和Ansible一起用，我可以管理多个服务器的资源(像VPN, DNS, 一些webservice等). 它也很有趣，也很快，并且如此稳定，让是做起事情来
如此容易。

### 赢家是哪位呢？
对于我一个用fdisk而不是gparted(或者virsh而不是virt-manager;)来说，Docker不是一个管理服务的好例子。老实来讲，我依然在找寻这个
标题的答案。对于现在，经过了几个星期对于Dokcer研究(几个月对于LXC的研究)，我可以告诉你一件明显的事，当你知道LXC, 了解Docker就是非常简单的一件事(跑一些精灵进程在斯巴达样的Dokcer镜像里面可能是件艰苦的战争，特别是有些库或者是依赖缺失的情况), 创建和跑起Dockerfiles也是一件容易的事，就像创建Ansible palybooks一样。

我想我要做一件事情，就像几年前当XEN 和 KVM 都出现在免费开源项目全虚拟化竞赛(FOSS full-virt race)里的一样. 我会两个都用，并且
观察Docker和LXC的后续发展。Docker非常容易和完美地管理应用(因此持续开发对于Docker是一个杀手级应用), 我也会用LXC/Ansible在一些基础服务上(GitLab, DNS, VPN 等)。但是为了好玩，我两个都会去尝试的。当用LXC开发GitLab时，我也会创建
Dockerfile为它。

就像我想的那样，我会有一个更好的答案在未来的一些日子里，这可能是个好的议题在一些会议演讲上吗？

---
这篇文章由[ Maciej Lasyk ](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981)发表，点击[此处](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981)可查阅原文。

The article was contributed by [Maciej Lasyk](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981) to read the original publication.