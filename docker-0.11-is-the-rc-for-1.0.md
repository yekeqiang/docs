Docker 0.11是1.0的预览版

最近几个月，我们被问最多的问题之一是，“什么时候发布1.0？”虽然这不是明确的回答，我们今天宣布，立刻可用的Docker 0.11，会是1.0的第一个预览版。从现在起，我们希望得到你们持续的反馈，尤其是质量问题的反馈。

预览称号之外，这个版本还有很多激动的特性。这里是一些重要的特性：

 - **支持SELinux**。大声的感谢社区成员，拥有Red Hat的“SELinux先生”称号的[Dan Walsh](https://github.com/rhatdan)，为我们贡献了这个特性。要运行支持SELinux支持的守护进程，使用`docker -d --selinux-enabled`。

 - **宿主网络**。Docker 0.11的“宿主网络”模式，是为那些希望容器能直接访问宿主系统的网络接口的人而开发的特性。宿主网络模式下运行的容器依旧处于沙箱中，只是与宿主系统共享一套网络栈。

	默认启动的容器，只会有一个环回接口`lo`和一个连接到docker桥接器的虚拟接口`eth1`：

		$> docker run busybox ip a

	You can now directly use the interfaces of you host (it’s much faster as it doesn’t go through the bridge) via --net=host:
	你现在可以通过`--net=host`直接使用宿主系统的接口（这样会更快，因为不需要通过桥接）：

		$> docker run --net=host busybox ip a

 - Link hostnames. When containers are linked together, they can now discover each other by hostname. For example your application frontend could connect to its database by simply opening a connection to the hostname “db”. This discovery method is optional, you can still use the usual environment variable discovery method.You can start a redis server on you machine, giving it the name ‘redis’ with:
 - **链接主机名**。如果容器之间被关联在一起，现在可以通过主机名来互相发现。比如，你的应用程序前端可以通过打开主机名为“db”的链接，来访问数据库。这种发现方法是可选的，你可以继续使用环境变量这种发现方法。你可以启动一个redis服务器，并用下面的方法命名为‘redis’：

		$> docker run -d --name redis crosbymichael/redis
		a6011908ba

	而现在如果想启动一个redis-cli容器，并关联到`redis`容器，可以这样做：

		$> docker run -it --name redis-cli --link redis:redis crosbymichael/redis-cli -h redis

	这样`redis-cli`容器的/etc/hosts会增加`redis`容器的ip。

 - **Ping Docker守护进程**。我们加入了一个新的节点，允许像做健康检测那样，ping一个docker守护进程，来确认它是否还活着。

 - **时间戳**。在Docker 0.11里，现在每个容器的守护进程的所有日志，都会有时间戳。查看`docker logs -t`。

 - **通过多个终端访问注册服务器的镜像节点**。这是个在docker里呼声很高的特性，可以集联多个注册服务器的镜像节点。比如，一个服务器挂了，可以立刻把请求转到待机镜像上。

 - **SHA-512**。为了特定SSL认证而增加SHA-512的支持。

The above major features plus bug fixes total 266 merged pull requests in Docker 0.11 – check out the full list.  Big thanks and shout-out to all contributors, particularly Dan Walsh, Victor Marmol, and Rohit Jnagal!
在Docker 0.11里，上面的主要特性加上修复bug，__一共合并了266个推送请求__——这里是[所有请求的列表](https://github.com/dotcloud/docker/pulse/monthly#merged-pull-requests)。非常非常感谢所有的贡献者，尤其是[Dan Walsh](https://twitter.com/rhatdan)，[Victor Marmol](https://github.com/vmarmol)，和[Rohit Jnagal](https://github.com/rjnagal)。

在我们通向1.0的路上，我们希望听到你的反馈和输入。

docker化，越早越频越好，

Docker团队