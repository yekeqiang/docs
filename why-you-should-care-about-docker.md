# 为什么你应该关注 Docker

##### 作者：[Chris Dowson](http://thenewstack.io/author/chrisdawson/)

##### 译者：[ Liu Mengxin](http://weibo.com/oilbeater)

***

当我在 [DockerCon](http://dockercon.com/) 上陶醉于那些令人激动地议题时，想到了一个问题：如何向在波特兰家中的妻子解释 Docker 呢？我妻子这时正在照料我们只有18个月大的生病的孩子。是什么让 Docker 这么有吸引力，以至于让我在 30 岁高龄时依旧奔波了 600 英里去参加这个大会？

现在会议中大多数关于 Docker 的新闻都需要你了解诸如 cgroup 、systemd 和 LXC 这样复杂的技术。如果你在 Stack Overflow 或者 Server Fault 这种网站的排名低于 1000 的话，去参加这样一场会议会让你很快感到绝望。我希望能够跳过那些技术，直接告诉你为什么 Docker 会让你赶到兴奋： Docker 能够让你的工作更简单，能够简化商业应用的流程，能够让一个公司更强大。

## Docker 可以加速新技术的采用，即使是在较为保守的机构 

昨天吃午饭的时候，我和两个在位列财富 500 强的金融服务公司工作的程序员聊天。他们向我讲述了在他们公司使用新的技术是一件多么困难的事情。公司里的安全专家只会对那些新技术说 “ no ” ，与希望使用新技术的程序员们做斗争，已经成为了一种常态。

Docker 作为一种标准的交付系统，把资源分配以及安全隔离的责任从操作人员和安全人员手中的责任清单中转移到了容器中。尽管这并不是银弹，但是如果安全团队只用负责验证 Docker 容器进程的安全性的话，他们会更可能同意使用新技术。这改变了游戏的规则。

## Docker 让维护旧的系统和代码更简单

无论你运行哪个版本的 Linux ， Docker 都会让维护系统变得简单。就像上面提到的，很多大型的企业都必须支持大量旧的系统和代码，而创业公司通常不会有这些问题。当我问来自 Heroku 的 [Fabio Kung ](http://fabiokung.com/) 和 [Rafael Rosa](http://www.grokpodcast.com/) 他们是如何解决这个问题的时候， Fabio 告诉我 Docker 使得他们维护旧系统和代码变得简单。你不需要用真实的物理主机去跑这些系统，也不需要用一个重量级的虚拟机（如果你的旧系统在一个 Linux 版本上运行）， Docker 给你提供了一个新的选择。 Docker 可以降低你维护旧系统的代价，甚至可以将你在上面的操作记录下来形成一个带版本控制的 “Dockerfile” 。

## Docker 可以快速降低部署的痛苦

管理者们通常会忽视持续集成、单元测试和敏捷开发这样的开发实践，但是他们会真切地关注一件事情，那就是开发的最后一个环节——部署。讽刺的是，尽管有上述和更多工具的支持，部署依然是一件令开发者十分头疼的工作。就像 Spotify 的工程师 [Rohan Singh](https://twitter.com/rohansingh) 昨天和我强调的那样，在提交经过测试的最终版代码和代码在生产服务器上运行之间还存在很大的距离。 Docker 可以极大的简化这最后一步，这对管理者和程序员们来说很重要，并且这样可以更快的让最新的产品呈现在用户面前。

## Docker 可以为财富500强的企业和创业公司解决问题

在大会上，那些大公司通过使用 Docker 获得了巨大的提升的事情深深的吸引了我。 Docker 现在正在经历着高速发展，你原以为只有小公司和早期使用者会持续跟进；但现在 Docker 已经展示了自己与大企业和小企业之间都有很强的关联性。

随着更多的公司采用并且改进 Docker，Docker 正在变得越来越好。参加 DockerCon 2014 是一段令人兴奋的经历。

***

##### 这篇文章由 [Chris Dowson](http://thenewstack.io/author/chrisdawson/) 撰写， [ Liu Mengxin](http://weibo.com/oilbeater) 翻译。点击 [这里](http://thenewstack.io/why-you-should-care-about-docker/) 可阅读原文。

##### The article was contributed by [Chris Dowson](http://thenewstack.io/author/chrisdawson/), click [here](http://thenewstack.io/why-you-should-care-about-docker/) to read the original publication.

