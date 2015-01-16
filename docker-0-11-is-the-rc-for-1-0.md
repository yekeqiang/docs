# Docker 0.11作为1.0 预览版发布

##### 作者：[Scott Johnston](https://twitter.com/scottcjohnston) ( SVP, Product of Docker, Inc )
##### 译者：[李兆海](https://twitter.com/googollee)

***
最近几个月，我们被问的最多的一个问题就是“什么时候发布1.0？” 今天我们宣布，立刻可用 的Docker 0.11 会是 1.0 的第一个预览版。从现在起，我们希望得到你们持续的反馈，尤其是质量问题的反馈。

除了被称作预览版，这个版本还有很多激动的特性。以下是一些重要的特性：

 - **支持SELinux**。大力感谢社区成员，拥有 Red Hat 的“ SELinux先生 ”称号的 [Dan Walsh](https://github.com/rhatdan) ，为我们贡献了这个特性。要运行支持SELinux支持的守护进程，使用 `docker -d --selinux-enabled` 。

 - **宿主网络**。 Docker 0.11 的“宿主网络”模式，是为那些希望容器能直接访问宿主系统的网络接口的人而开发的特性。宿主网络模式下运行的容器依旧处于沙箱中，只是与宿主系统共享一套网络栈。

	默认启动的容器，只会有一个环回接口`lo`和一个连接到docker桥接器的虚拟接口 `eth1` ：

	```$> docker run busybox ip a```

    
	你现在可以通过 `--net=host` 直接使用宿主系统的接口（这样会更快，因为不需要通过桥接）：

	```$> docker run --net=host busybox ip a``


 - **链接主机名**。如果容器之间被关联在一起，现在可以通过主机名来互相发现。比如，你的应用程序前端可以通过打开主机名为 “db” 的链接，来访问数据库。这种发现方法是可选的，你可以继续使用环境变量这种发现方法。你可以启动一个 redis 服务器，并用下面的方法命名为 ‘redis’ ：


		`$> docker run -d --name redis crosbymichael/redis`
		`a6011908ba`

	而现在如果想启动一个 redis-cli 容器，并关联到 `redis` 容器，可以这样做：

		`$> docker run -it --name redis-cli --link redis:redis crosbymichael/redis-cli -h redis`

	这样 `redis-cli` 容器的 /etc/hosts 会增加 `redis` 容器的 ip 。

 - **Ping Docker 守护进程**。我们加入了一个新的节点，允许像做健康检测那样， ping 一个 docker 守护进程，来确认它是否还活着。

 - **时间戳**。在 Docker 0.11 里，现在每个容器的守护进程的所有日志，都会有时间戳。查看 `docker logs -t` 。

 - **通过多个终端访问注册服务器的镜像节点**。这是个在 docker 里呼声很高的特性，可以集联多个注册服务器的镜像节点。比如，一个服务器挂了，可以立刻把请求转到待机镜像上。

 - **SHA-512** 。为了特定 SSL 认证而增加 SHA-512 的支持。

在 Docker 0.11 里，上面的主要特性加上修复 bug，**一共合并了266个推送请求**——这里是 [所有请求的列表](https://github.com/dotcloud/docker/pulse/monthly#merged-pull-requests) 。非常非常感谢所有的贡献者，尤其是 [Dan Walsh](https://twitter.com/rhatdan) 、 [Victor Marmol](https://github.com/vmarmol) 和[Rohit Jnagal](https://github.com/rjnagal) 。

在我们通向1.0的路上，我们希望听到你们的反馈和投入。

docker 化，越早越频越好，

Docker 团队

***

##### 这篇文章由 [Scott Johnston](https://twitter.com/scottcjohnston) 发表，[李兆海](https://twitter.com/googollee)，点击 [这里](http://blog.docker.io/2014/05/docker-0-11-release-candidate-for-1-0/) 可阅读原文。

##### The article was contributed by [Scott Johnston](https://twitter.com/scottcjohnston) , click [here](http://blog.docker.io/2014/05/docker-0-11-release-candidate-for-1-0/) to read the original publication.
