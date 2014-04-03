# Docker vs LXC/Ansible?
---

### 为什么提出这问题？

During last DevOPS meetup @GrzegorzNosek asked very good question – why should one use Docker instead of pure LXC/Ansible?

Honestly I’ve been trying to answer myself this question for a while. I did in some part (included this in my talk I gave during that meetup: http://www.slideshare.net/d0cent/docker-rhel); while it’s about developers running development envs Docker is just so much easier to use.

But how should I explain using Docker for myself? I’m sysadmin and I love low-level – so LXC for me is just natural way of doing things :)

在最近一期 DevOPS 交流会上， [@GrzegorzNosek](https://twitter.com/GrzegorzNosek) 问了这样一个好问题: 为什么要选择 Docker 来替代 LXC/Ansible ?

老实说我也一直尝试回答此问题，并且回答了部分（参见我在那次交流会上的 [演讲](http://www.slideshare.net/d0cent/docker-rhel](http://www.slideshare.net/d0cent/docker-rhel) )。演讲主题是 Docker 让开发者运行开发环境十分简单。

就我本人而言，为什么会用 docker 呢? 身为一个 sysadmin （系统管理人员），我喜欢更底层的东西，自然而然地就使用 LXC 了。

### 脸还是屁股？有何区别?

(If you feel embarassed / disgusted somehow with this header please rewind 18 years and remember that: https://en.wikiquote.org/wiki/Duke_Nukem)

One thing you should know about me – I’m contributing to FedoraProject; lately I’ve been poking around Fedora-Dockerfiles project (https://git.fedorahosted.org/cgit/dockerfiles.git/) – I’m doing it for fun and also I wanted to learn more about Docker as I’m running some Open-Source projects with friends and had to find a easy way for them to rollup own development envs. Docker is the answer in this case.

So – currently I’m using Docker to prepare dev-envs for guys who knows nothing about DevOPS / SysOPping; writing Dockerfiles is so much fun (and sometimes so big hell :) ). And LXC? Together with Ansible I’m managing some servers’ resources (like VPN, DNS, some webservices etc). It’s also fun, it’s fast, rather reliable and it makes things so much easy to live with.


(如果你觉得这个标题令人尴尬或者心生厌恶，不妨回想一下 18 年前的 [毁灭公爵](https://en.wikiquote.org/wiki/Duke_Nukem) 维基词条)

你们都知道我正在为 FedoraProject 做贡献；最近我在捣鼓一个项目 [Fedora-Dockerfiles project](https://git.fedorahosted.org/cgit/dockerfiles.git/) 。鼓捣这个项目纯粹是出于好玩，同时我也想对 Dokcer 了解更多。 当我和朋友们开始一些开源项目时，必须找到一种简单的方式去管理我们的开发环境。在这种情况下，答案就是 Docker 。

目前我在用 Docker 为哪些对 DevOPS/SysOPping 一无所知的人们准备开发环境，而编写 Dockerfiles 充满乐趣（不过偶尔也会有大坑） :)) 。LXC 表现如何呢？通过搭配 Ansible 使用，我能管理多个服务器资源（比如 VPN 、 DNS 、一些网页服务等）。有趣又快速，并且非常稳定，让工作和生活简单多了。

### 谁是赢家呢？

But still – for me as guy who use rather fdisk than gparted (or virsh than virt-manager ;) ) Docker is not the case for managing services. And honestly I’m still looking for an answer for the question from subject of this blogpost. For now after couple of weeks poking around Docker (and months with LXC) I can tell this one obvious thing that when You know LXC than Docker is just so easy (e.g. running some daemons inside spartan-like Docker images can be a tough fight whe some libs or dependencies are missing). Also creating and running Dockerfiles is very easy – just like creating Ansible playbooks.

I think that I’m gonna do this one thing that I did couple of years ago when XEN and KVM were running shoulder to shoulder in the FOSS full-virt race. I’m just gonna use them both – Docker and LXC and see how things will develop. Docker is very great and easy to manage apps only (so Continuous Development with Docker is killing feature) and I’ll LXC/Ansible within some basic services (GitLab, DNS, VPN etc). But for more fun – I’m gonna keep both tracks, so e.g. when deploying GitLab within LXC I’ll create also Dockerfile for this.

This way I think that I will have a really good answer in just a couple of weeks and this should be nice subject for some conference talk?

对于我一个用fdisk而不是gparted(或者virsh而不是virt-manager;)来说，Docker不是一个管理服务的好例子。老实来讲，我依然在找寻这个
标题的答案。对于现在，经过了几个星期对于Dokcer研究(几个月对于LXC的研究)，我可以告诉你一件明显的事，当你知道LXC, 了解Docker就是非常简单的一件事(跑一些精灵进程在斯巴达样的Dokcer镜像里面可能是件艰苦的战争，特别是有些库或者是依赖缺失的情况), 创建和跑起Dockerfiles也是一件容易的事，就像创建Ansible palybooks一样。

我想我要做一件事情，就像几年前当XEN 和 KVM 都出现在免费开源项目全虚拟化竞赛(FOSS full-virt race)里的一样. 我会两个都用，并且
观察Docker和LXC的后续发展。Docker非常容易和完美地管理应用(因此持续开发对于Docker是一个杀手级应用), 我也会用LXC/Ansible在一些基础服务上(GitLab, DNS, VPN 等)。但是为了好玩，我两个都会去尝试的。当用LXC开发GitLab时，我也会创建
Dockerfile为它。

就像我想的那样，我会有一个更好的答案在未来的一些日子里，这可能是个好的议题在一些会议演讲上吗？

---
这篇文章由[ Maciej Lasyk ](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981)发表，点击[此处](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981)可查阅原文。

The article was contributed by [Maciej Lasyk](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981) to read the original publication.
