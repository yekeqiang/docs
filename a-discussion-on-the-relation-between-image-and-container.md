# Docker 的 image 和 container 是什么关系？

---

作者：周晓璐

---

![alt](http://resource.docker.cn/docker-container.png)

今天的话题来自于一个好多大牛的微信群的讨论，而讨论开始于这个帖子：[http://bbs.csdn.net/topics/390847409](http://bbs.csdn.net/topics/390847409) 本文就又来自于 CSDN Docker 技术讨论群（303806405）中大家的讨论。

好了，说完来龙去脉，我们来看大家的讨论：

楼主的问题是这样的：

> images 安装了我的 app ，但是它是 readonly 的，那我要 update app 是怎么做的？
 container 相当于一个我 app 的 dependency ，而 container 又是通过 image 创建的，这些是啥关系？对这些概览比较模糊，希望大侠能够帮助讲解一下

对于初学者来说，用类比的方式解释更好，所以大家提出了以下几个类比：

- 类比1：image是盗版的 xp 安装光盘， container 是装在你笔记本上的 C:\Windows 目录。

- 类比2：image vs container 的关系，就好比面向对象中的类 vs 实例。

- 类比3：也有人说 image 是序列化掉的 container ， Docker 就好比要运送虾仁，序列化就相当于冷冻，然后运输，再换到别的地方解冻。

	但这个类比有个问题，序列化和反序列化表述的是1对1的关系。内存 => 磁盘/网络叫序列化，磁盘/网络 => 内存叫反序列化。

	而 Docker 的 image 上传到 hub.docker.com ，可被很多人下载到本地 image 缓存库，并实例化为n个 container 。所以 image 和 container 的关系是1对n的关系。

- 类比4：以上观点是从全局来看，如果从局部去看， image 和 container 的区别在于是不是运行期。 run 了就是 container ，不 run 就是 image。其实是 run 的时候创建了一个可写的 image 在最后 mount 的。

类比5：image 和 container 就是可执行文件和进程的关系。实际上， image 里确实是一堆 bin 文件，而运行中的 container 就是一组进程。

然后小编去请教了 Docker的 Jerome ，他是这么回答的：

> An image is read only. It is like a template, a model. From the image, you can create one or many containers. The containers are read write. Another analogy: the image is like a blueprint, and the container is the execution of the blueprint.

由于很简单的内容，小编就不翻译了，有人是这么理解Jerome这段话的：

- 1. image 和 container 是1对n，n>=1
- 2. image 是静态的 template/blueprint ，而 container 是 image 的 execution 实例化

讨论到最后，也有人觉得“对于一个概念，最难让人理解的不是概念，而是**理解概念的上下文***”，“我觉得**与其花笔墨试图去解释概念是什么，还不如把要解决啥问题搞清楚。问题搞清楚，解决方案水到渠成**。”

其实小编觉得这个问题并不难，做一些实际的操作，慢慢就能理解。但是在讨论的过程中可以加深理解，顺便搞清楚一些其它问题。

感谢群里各位的分享，如果大家有什么问题，可以发给我，我会发到群里请各位大牛讨论。真理是越辩越明滴~

---

本文转载自由周晓璐维护的微信公众帐号 "Container 技术日报"。