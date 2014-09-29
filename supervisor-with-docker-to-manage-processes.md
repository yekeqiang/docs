#在 Docker 里使用（支持镜像继承的）supervisor 管理

![alt](http://resource.docker.cn/homepage-docker-logo.png)
#####作者：[Quinten Krijger](https://twitter.com/qkrijger)
#####译者：[李兆海](https://twitter.com/googollee)

---

在去年八月份，我写了一篇关于如何创建 tomcat 镜像的 [blog](http://blog.trifork.com/2013/08/15/using-docker-to-efficiently-create-multiple-tomcat-instances/) 。从那以后， docker 又改进了很多，我对 docker 的了解也增加了很多。我很高兴和你分们享我发现的管理 container 进程的好办法。在读完这篇文章后，我希望你能善加利用我 [github 仓库](https://github.com/Krijger/docker-cookbooks) 里的 supervisor 镜像。

## Docker 命令

在之前的文章里，我提到 Docker（只能）支持运行一个前台进程。我们通常习惯使用类似 upstart 这种管理服务来初始化启动流程，但是 Docker 默认没有这些服务的支持。刚开始使用 Docker 时会很不习惯，你必须指定你想要运行的进程。这种行为和虚拟机相比有个优点，会尽可能的保持轻量的 container 。你可以通过 run 命令最后的参数，在启动 container 时指定进程命令，比如：

`docker run ubuntu echo "hello world"`

另外一种方法，你可以利用 [CMD](http://docs.docker.io/en/latest/reference/builder/#cmd) 指令，在 Dockerfile 里指定 docker run 命令的默认参数。比如，如果你目录下的Dockerfile包含以下内容：
```
FROM ubuntu
CMD echo "hello world"
```
再使用下面的指令构造 `hello_world_printer` 镜像：

`docker build -t "hello_world_printer" `

使用下面的命令，你可以得到和之前 run 命令相同的执行结果。

`docker run hello_world_printer`

要注意，因为你可以覆盖掉 CMD 指定的命令行参数，这个只是个运行时的指令。有趣的事情是，在 Linux container 里，你可以只调用 upstart 命令然后得到和普通虚拟机大致相同的行为。


## 运行多个命令

运行多个进程是个很正常的想法，比如，一个 ssh 服务（这样就能登录到正在运行的 container ）和实际的应用。你可以用下面的方法运行container：

```
docker run ... /usr/sbin/sshd && run_your_app_in_foreground
```

这在开发时很方便。这样，当应用进程退出后，因为唯一的前台程序退出了， container 会自动关闭。当然你可以使用 `using /usr/bin/sshd -D` 保证 container 不会退出，但是这里真正的问题是，这种使用 run 命令设置初始程序的方式不够简洁。而且，随着你的 container 变复杂， run 命令会越来越长。

所以，在运行更复杂的 container 的时候，很多人使用复杂的 bash 脚本。典型的 bash 脚本会执行一个前台进程，并开启一个或者多个（ renegade ）守护进程。与只用 Docker 命令行的方式相比，这种方法最重要的改进在于，bash 脚本是可以做版本控制的：启动脚本在你的 Docker 镜像里，新的改动可以和软件项目一起分发。不过，使用 bash 脚本管理进程依旧简陋枯燥，而且容易出错。

##使用 supervisor

更好的方法是使用 [supervisor](http://supervisord.org/) 。 supervisor 可以更好的管理进程：使用更加简洁的代码管理进程；在崩溃时可以重启进程；允许重启一组进程并且有命令行工具和网页界面来管理进程。当然，能力越大，责任越大：大量使用 supervisor 特性的代码，预示着你应该将整个服务更好的拆分成多个小的 supervisor 来管理。

个人来讲，我喜欢 supervisor 让我用更清晰的代码管理启动的进程。我见过最简洁的使用例子，是子镜像扩展出一个进程组。比如，如果你经常使用 SSH ，使用一个 SSH 镜像作为基础镜像就是很合理的。这种情况下，在所有基于这个镜像的扩展镜像上实现启动 SSH 进程的代码，形式上就是一种重复代码。我来给你们展示下我找到的解决这个问题的好办法。

## supervisor基础镜像

首先，因为我默认使用 supervisor ，所以我所有的镜像都扩展自一个只包含 supervisor 和最新版本 ubuntu 的基础镜像。你可以在 [这里](https://github.com/Krijger/docker-cookbooks/blob/master/supervisor/Dockerfile) 找到这个 Dockerfile 。这个基础镜像包括一个配置文件 `/etc/supervisor.conf` ：

	[supervisord]
	nodaemon=true

	[include]
	files = /etc/supervisor/conf.d/*.conf

这个配置让 supervisor 本身以前台进程运行，这样可以让我们的 container 启动后持续运行。第二，这个配置将包含所有在 `/etc/supervisor/conf.d/` 目录下的配置文件，启动任何在这里定义的程序。

## 扩展基础镜像

![alt](http://resource.docker.cn/tomcat-stack-164x300.png)

想法其实很简单。所有的子容器通过将特定的 service.sv.conf 放到特定的目录的方式，将其自己的服务加入到 supervisor 的管理里。之后，使用如下命令启动 container ：

```
docker run child_image_name "supervisor -c /etc/supervisor.conf"
```

这会自动启动所有指定的进程。你可以对镜像做多层扩展，每层扩展加入一个或者多个服务到配置目录。在 Docker 里使用 supervisor 启动命令代替 upstart 也更有效和有范。

举个例子，我们可以看看之前 blog 提到的 Tomcat 工作栈，是如何使用这种改进后的方法的。

 - 首先，和之前讨论的一样，我们使用从 ubuntu 扩展而来的 supervisor 基础镜像

 - 然后，我们使用在 supervisor 上安装了 Java 的 [JDK镜像](https://github.com/Krijger/docker-cookbooks/tree/master/jdk7-oracle) 。 Java 只是其他服务使用的库，所以我们在这层不指定任何启动服务。这层要做一些类似设置 JAVA_HOME 环境变量的通常任务

 - Tomcat 镜像在工作栈上安装 Tomcat 并暴露 8080 端口。这层包括一个名字是 Tomcat 的服务，定义在 [tomcat.sv.conf](https://github.com/Krijger/docker-cookbooks/blob/master/tomcat7/tomcat.sv.conf) ：

		[program:webapp]
		command=/bin/bash -c "env > /tmp/tomcat.env && cat /etc/default/tomcat7 >> /tmp/tomcat.env && mv /tmp/tomcat.env /etc/default/tomcat7 && service tomcat7 start"
		redirect_stderr=true

执行 Tomcat 服务的命令并不像我喜欢的那样简洁，将其放到一个专门的脚本里会更好。命令先添加了一些环境变量，比如 [container的关联参数](http://docs.docker.io/en/latest/use/working_with_links_names/) 到 `/etc/default/tomcat7` ，这样我们可以在之后的配置中使用这些参数，后面的例子会展示这种用法。也许使用类似 etcd 的键值存储会更好，不过这超出了本文的范畴。

当然，我们这里只安装了默认的文件，没有真正的网络应用程序。

 - 你的网络应用程序应当扩展自 Tomcat 镜像，并安装入真正的应用程序。当启动 supervisor 的时候，会自动启动 Tomcat 。

## 一个 Tomcat 网络程序的 Dockerfile 例子

如何安装实际的网络应用超出了本文的范畴，不过，有始有终，我给出了个 Dockerfile 例子，演示如何使用这个工作栈。这个例子完全基于 Java Tomcat ，所以如果你对这个不感兴趣，别读了，玩别的去吧:)

假设，我们有一个使用 Elasticsearch 的网络应用：

	FROM quintenk/tomcat:7

	# 安装一些项目的依赖，这些依赖在每次更新时不会改变
	# RUN apt-get -y install ...

	RUN rm -rf /var/lib/tomcat7/webapps/*

	# 将配置加入 /etc/default/tomcat7 ，比如：
	...
	RUN echo 'DOCKER_OPTS="-DELASTICSEARCH_SERVER_URL=${ELASTICSEARCH_PORT_9200_TCP_ADDR}"' >> /etc/default/tomcat7
	RUN echo 'CATALINA_OPTS="... ${DOCKER_OPTS}"' >> /etc/default/tomcat7

	# 加入类似 log4j.properties 的配置文件，并将其chown root:tomcat7

	# 假设项目已经构建好了，而且 ROOT.war 在你构建 Docker 的目录（包含 Dockerfile 的目录）。基于缓存的考虑，这个作为最后的步骤
	ADD ROOT.war /var/lib/tomcat7/webapps/
	RUN chown root:tomcat7 /var/lib/tomcat7/webapps/ROOT.war

	CMD supervisord -c /etc/supervisor.conf

这段代码里， elasticsearch 的相关环境变量（搜索索引）已经被设置了，因为 supervisor 关于 Tomcat 的配置，会在启动时将所有环境变量添加到 /etc/default/tomcat7 。当然，我们在启动网络应用镜像时需要关联到 elasticsearch containter ，比如：

```
docker run -link name_of_elasticsearch_instance:elasticsearch -d name_of_webapp_image "supervisor -c /etc/supervisor.conf"
```
你现在的网络应用可以去访问 `ELASTICSEARCH_SERVER_URL` 路径了。你可以在配置文件里使用这个变量，像这样：
	elastic.unicast.hosts=${ELASTICSEARCH_SERVER_URL}

这样就可以将配置暴露给你的应用程序。如果你是个 Java 开发者，并且也阅读了前一篇文章，希望本文的技术能让你开始一段愉快的代码之旅。

***

#####这篇文章由 [Quinten Krijger](https://twitter.com/qkrijger) 发表，[李兆海](https://twitter.com/googollee) 翻译。点击 [这里](http://blog.trifork.com/2014/03/11/using-supervisor-with-docker-to-manage-processes-supporting-image-inheritance) 可查阅原文。

#####The article was contributed by [Quinten Krijger](https://twitter.com/qkrijger), translated by [@googollee](https://twitter.com/googollee). Please click [here](http://blog.trifork.com/2014/03/11/using-supervisor-with-docker-to-manage-processes-supporting-image-inheritance) to read the original publication.
