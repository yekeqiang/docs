# Docker 1.3发布：增加安全机制及进程注入

---

##### 作者：[张天雷](http://www.infoq.com/cn/author/%E5%BC%A0%E5%A4%A9%E9%9B%B7)

---

10月16日，Docker发布了 [1.3版本](http://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/) 。历经45个贡献者的多达750次代码提交，这次发布的版本除了进行以往发布例行的质量改善以外，还增加了不少有用的新功能，如数字签名机制、进程注入、更好的镜像创建方式以及安全选项，下面对这几个功能进行逐一介绍：

## 数字签名机制

在这个版本中，Docker引擎将使用数字签名机制自动分析所有官方Docker镜像库（Repo）的来源和完整性。这种机制可以发现官方镜像库可能存在的入侵，从一定程度上保证镜像使用过程中的可信度。Docker团队表示，该机制只是接下来要发布的众多新特性之一，更多的还包括镜像分发商和用户经常会用到的身份验证，PKI管理以及镜像加密等。作为首次发布，该特性还处于继续研发的阶段，目前如果Docker引擎发现使用的镜像被入侵，只会抛出一个警告，而不是阻止用户继续使用该镜像。在未来的版本中，该特性将会被进一步增强，严格过滤恶意镜像。

## Docker exec：进程注入

在开发基于Docker的应用过程中，开发者可能需要运行时查看应用。诸如nsinit和nsenter这样的工具，在过去的开发过程中，在一定程度上起到了作用，但是这些都是第三方工具，需要开发者自己去寻找、学习和管理。本次发布中，有了一个新的命令exec，可以让开发者轻松在Docker容器里通过API和命令行工具生成进程，比如：

```$ docker exec ubuntu_bash -it bash```

就会在ubuntu_bash这个容器中运行bash。

进一步来讲，通过提供exec命令，开发者就拥有了更灵活的应用开发调试助手。

### Docker create：更加细化的镜像创建方式

之前开发者都是使用run命令来创建容器，并在其中运行应用。随着Docker的广泛应用，越来越多的开发者要求对容器的创建有更加细致的控制，本次发布的create命令就是解决这个需求的。通过run命令，开发者可以创建但是不去运行容器，稍后可以使用start和stop命令来控制容器的生命周期。

### 安全选项

可能国内很多同仁所关心的就是这个特性了，Docker命令行工具新增加了一个参数，--security-opt，为用户设定个性化的SELinux和AppArmor标签和属性。比如，需要设置一个安全选项，让容器只能监听Apache端口，假设这个策略命名为svirt_apache，那么就可以通过如下命令完成这个需求：

```$ docker run --security-opt label:type:svirt_apache -i -t centos \ bash```

这样做的好处就是，Docker容器不一定要运行在支持SELinux的内核上，也无需用—privileged参数运行，避免了很多安全隐患。

此外本次发布还有Mac OS X下的共享目录工具boot2docker。

--- 
感谢 [郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE) 对本文的审校。

---

本文原载于 InfoQ ，在得到授权后，我们将其转载。你可以阅读 [原始版本](http://www.infoq.com/cn/news/2014/10/docker-1.3-release) 。
