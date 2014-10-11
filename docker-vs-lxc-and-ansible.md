# Docker vs LXC/Ansible

![alt](http://resource.docker.cn/linux-containers-300x248.png)

##### 作者：[Maciej Lasyk](https://twitter.com/docent_net)
##### 译者：[@regrecall](http://weibo.com/u/2279238670)



## 为什么提出这问题？

在最近一次的 DevOPS 交流会上， [@GrzegorzNosek](https://twitter.com/GrzegorzNosek) 问了这样一个好问题：为什么要用 Docker 来替代 LXC/Ansible ?

老实说我也一直尝试回答此问题，并且回答了部分（参见我在那次交流会上的 [演讲](http://www.slideshare.net/d0cent/docker-rhel](http://www.slideshare.net/d0cent/docker-rhel) ) 。演讲主题是关于 Docker 让开发者运行开发环境十分简单。

就我本人而言，为什么会用 docker 呢? 作为系统管理员，我喜欢更底层的东西，用 LXC 自然是理所当然。

## 脸还是屁股？有何区别?


（如果你觉得这个标题令人尴尬或者心生厌恶，不妨回想一下 18 年前的 [毁灭公爵](https://en.wikiquote.org/wiki/Duke_Nukem) 维基词条）


你们都知道我在为 FedoraProject 做贡献，最近捣鼓一个项目 [Fedora-Dockerfiles project](https://git.fedorahosted.org/cgit/dockerfiles.git/) 。鼓捣这个项目一方面出于好玩，同时我也想对 Dokcer 了解更多。 当我和朋友们开始一些开源项目时，必须找到一种简单的方式去管理我们的开发环境。在这种情况下，答案就是 Docker 。

目前我在用 Docker 为哪些对 DevOPS/SysOPping 一无所知的人们准备开发环境，而编写 Dockerfiles 充满乐趣（不过偶尔也会有大坑）。 :)) LXC 表现如何呢？通过搭配 Ansible 使用，我能管理多个服务器资源（比如 VPN 、 DNS 、一些网页服务等）。有趣、速度快，并且非常稳定，让工作和生活简单多了。

## 谁是赢家？


不过对于我这种宁可用 fdisk 也不用 gparted （用 virsh 而不用 virt-manager ）的人来说，Docker 并不是一个管理服务的好例子。老实来讲，我依然在找寻这个
问题的答案。经过对 Dokcer 几周的研究（和数月对 LXC 的研究)，我可以告诉你一件明显的事，LXC 相比 Docker 还是简单些（例如在 spartan-like 这样的 Docker 镜像里运行 daemon 非常艰难，特别在某些库或者依赖缺失的情况下。）然而创建和运行 Dockerfile 却十分容易，就像创建 Ansible playbook 一样。

我觉得自己要做一件几年前已经做过的事情，那时我将 XEN 和KVM 共同运行在 FOSS full-virt race （免费开源项目全虚拟化竞赛）里。现在我打算同时使用 Docker 和 LXC 以观后效。 Docker 非常容易和完美地管理应用（因此使用 Docker 进行持续开发会是一个杀手级功能），而在一些基础服务（ Gitlab 、 DNS 、 VPN 等）中我会使用 LXC/Ansible 。不过为了好玩，我会让它们并行工作，比如我在 LXC 下部署 Gitlab 的时候也会顺便创建一个对应的 Dockerfile 。

借助这种开发方式，我想自己可能会在后面几周里找到这个问题的最佳答案，这也将会是非常棒的演讲话题。

---
##### 这篇文章由 [Maciej Lasyk](https://twitter.com/docent_net) 发表，点击 [此处](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981) 可查阅原文。

##### The article was contributed by [Maciej Lasyk](https://twitter.com/docent_net) , click [here](http://maciek.lasyk.info/sysop/2014/03/16/docker-vs-lxcansible/?utm_source=Docker+News&utm_campaign=778f653f1b-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-778f653f1b-235722981) to read the original publication.
