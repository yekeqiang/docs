# Docker容器的安全配置

***
作者：[Nicholas Clements](http://www.linkedin.com/pub/nicholas-clements/13/544/478) 

译者：[我不围观](http://weibo.com/ooutman)
***

[技术支持](http://www.cohesiveft.com/support/support-contacts/)是CohesiveFT的一个重要内容，投入了较大的力量。用户经常会有一些关于我们产品的问题需要咨询，我们提供高质量的支持，也提供其他产品和服务的支持和最佳实践。最近就有一个用户询问Docker镜像和容器安全配置的最佳实践。因为我们支持在[VNS3 Manager的3.5版本](http://www.cohesiveft.com/support/vns3-release-notes/)上使用容器，我们还是比较乐于聊聊这个的。

用户想知道怎么比较保险和安全的方式创建和定制[Docker容器](https://www.docker.com/whatisdocker/)。一般来说，他们希望每个容器是独立的，密钥和配置只用于登录容器的用户和组。这样他们就可以控制谁能看到什么，而且比较容易增加或者删除权限。他们希望发布的镜像和Dockerfile不能被随意获取，生成的证书存在于可控和私有的仓库中。

他们还希望完成这些工作不要有太多的开发工作。

有很多方法可以给镜像和容器加入数据。很多Docker爱好者用的一个方法是，镜像本身是可以公共访问的，在容器启动的时候传递敏感信息，可以通过环境变量或者加载一个外部地址到容器中。尽管这是在Docker发展过程中浮现于脑海中的一种有用方法，但用户和CohesiveFT都觉得这种方式削弱了Docker的沙箱功能。

![](http://i.kinja-img.com/gawker-media/image/upload/s--8dnJmCqZ--/c_fit,fl_progressive,q_80,w_636/17m8iznzixs6jjpg.jpg)

这种方法遇到VNS3就不好用了--用户不能直接访问VNS3文件系统，而且我们目前也不支持通过API或者界面的方式传递环境变量。(我们正在开发中，相信我，可能在3.7的版本就支持了。)

现在，用户可以直接上传图片到VNS3上，或者把图片打包进网络平台的VNS3应用中。用户可以用Dockerfile和公共的仓库从头开始搞，也可以上传他们自己的镜像。

![](http://img.scoop.it/WO4z2Deip51YhTBUMirOWzl72eJkfbmt4t8yenImKBVvK0kTmF0xjctABnaLJIm9)

我们和用户一起分析过几个场景。他们认为有两个直接的方法风险最小：


* 在容器内部运行一个配置管理的客户端，仔细想想就会发现，这种方法只是推迟了问题发生的时间，因为同样的问题在以后还是需要解决的。
* 在镜像内部运行sshd服务，内置SSH密钥，给每个人分发密钥和配置文件后再连接。这里的问题包括定时，还需要有办法提前生产和存储所有的密钥(或者商量好用相同的密钥，只是这主意不是很好)。

尽管[Docker 1.0已经发布了](http://blog.docker.com/2014/07/announcing-docker-1-1/)，Docker仍然在飞速发展，这是大家公认的。为了尽可能的保护我们的用户，我们决定采用保守的方法。

目标是：

* 用安全的方式制作标准镜像，尽可能少的对外暴露
* 对标准镜像进行安全的定制，每个VNS3示例、每个镜像都不一样
* 安全存储修改的内容
* 最小的开发工作

我们从安全的和不影响使用的方式开始存储对象、镜像文件、Dockerfile文件、配置文件和相关文件。这种方式简单，而且把最后实现的选择留给了用户。我们的建议是，使用内部Web服务器，部署在防火墙后面并加入到VNS3拓扑中。另外一个方案是使用S3和过期时间长的签名URL。

下面我们来比较一下使用Dockerfile和预先制作好镜像这两个方案。因为用户是在寻找增加容器到一个已存在的拓扑网络的方法，我们推荐后面的一个方案。这会对网络的影响最小。结果其实是一样的，我们使用的镜像，也是用Dockerfile来制作的，然后导出生成最后的容器。

最后，对每个镜像管理进行定制。在VNS 3.5中，我们实现了一个特性，就是能够基于本地存储的镜像重新制作一个镜像。这种方法用了Dockerfile，但是因为使用的基础镜像是本地的，管理的影响就比较小了。而且如果在Dockerfile中使用的命令很少，整体的重建就会很快。

在最初的VNS3界面中我们是这样来验证(proof-of-concept)的，导入到基础镜像（弱弱的起个名字，就叫base_image吧）中的root密码是不知道的，通过外部的Dockerfile来实现个性化。

```
FROM vns3local:base_image
RUN mkdir -p /root/.ssh
RUN echo "[PUBLIC KEY HERE]" >> /root/.ssh/authorized_keys
RUN chmod 400 /root/.ssh/authorized_keys
```

再花几个小时写了一个简单的bash脚本，这个脚本利用了VNS3的API，还从密钥服务器下载公钥，作为概念验证程序(proof-of-concept)。

这样只用了几行简单的代码，就能在VNS3内部安全的创建，更重要的是安全的销毁，一个定制的Docker容器。用户不需要花很多时间和钱去搞一个大型的方案解决去他们的问题。[Security and control](http://www.cohesiveft.com/products/vns3/)


***
这篇文章由 [Nicholas Clements](http://www.linkedin.com/pub/nicholas-clements/13/544/478)  撰写，[我不围观](http://weibo.com/ooutman) 翻译。点击 [这里](http://blog.cohesiveft.com/2014/07/secure-provisioning-of-docker-containers.html) 阅读原文。
***