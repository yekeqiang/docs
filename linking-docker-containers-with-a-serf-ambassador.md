#使用 SERF AMBASSADOR 来链接 DOCKER 容器


#####作者：[Lucas Carlson](http://www.centurylinklabs.com/author/cardmagic/)
#####译者：[Mark Shao](https://github.com/markshao)

***

几个月前，我们写了 [去中心化的 Docker](http://www.centurylinklabs.com/decentralizing-docker-how-to-use-serf-with-docker/) 一文来说明如何配合 Docker 使用 Serf 实现服务注册，从而使应用与数据库服务间接链接在一起了。数周前，我们又写了一篇如何使用 [Docker ambassadors](http://www.centurylinklabs.com/deploying-multi-server-docker-apps-with-ambassadors/) 模型的文章。

但是在关于 Docker ambassadors 的 [这篇文章](http://www.centurylinklabs.com/deploying-multi-server-docker-apps-with-ambassadors/) 中，我们仅仅部署了一个代理 ambassador ，它除了转发 TCP 请求，别的啥都不做。

这周我们要使用 Docker ambassadors 来做一些有趣的东西： ambassador 会帮我们把应用容器和数据库容器的信息注册到 Serf 网络中。这和使用 ambassadors 把服务注册到 etcd 一样简单。（我最后会展示如何实现)。首先先从 Serf 开始。

## 使用 Docker 创建 Serf

首先，在一个 Docker 容器上建立一个 serf 代理。

```
$ SERF_ID=$(docker run -d --name serf_1 -p 7946 -p 7373 ctlc/serf /run.sh)
```

非常简单！现在我们使用一个装有 Serf 客户端的容器来链接到 Serf 代理服务，它会自动打印所有链接到网络上的成员。

```
$ docker run -i -t --link serf_1:serf_1 ctlc/serf-members
67ffbe3dba95  172.17.0.3:7946  alive  role=serf-members
f5af7d9dbc3c  172.17.0.2:7946  alive  role=serf-agent
```

现在我们可以看到进度，继续下去。

## 创建一个随机的 MySQL 容器

现在我们要创建一个MySQL数据库，我在 Docker Index 中找出一个完全随机的 MySQL 数据库容器:

```
$ docker run --name mysql -p 3306 -d brice/mysql
$ docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                                              NAMES
385c51c9de5a        brice/mysql:latest          /bin/sh -c mysqld   18 minutes ago      Up 18 minutes       0.0.0.0:49169->3306/tcp                            mysql   
67ffbe3dba95        ctlc/serf-members:latest   /run.sh             51 minutes ago      Up 51 minutes       7373/tcp, 7946/tcp                                 sleepy_brown       
f5af7d9dbc3c        ctlc/serf:latest           /run.sh             51 minutes ago      Up 51 minutes       0.0.0.0:49159->7373/tcp, 0.0.0.0:49160->7946/tcp   ...  
```

这个不是可信镜像，我们也完全不知道到底在跑什么代码。由于是从 index 中随机选出的镜像，因此 brice/mysql 有极大可能不能运行 Serf 。那我们如何把这个数据库注册到 Serf 呢？当然要用到 ambassador 模型。

```
$ docker run -d --link mysql:mysql --link serf_1:serf_1 --name mysql_ambassador -p 3306:3306 ctlc/amb-serf
```

现在回过头来看一下另外一个运行着 ctlc/serf-members 容器的终端窗口。

```
67ffbe3dba95  172.17.0.3:7946  alive   role=serf-members
f5af7d9dbc3c  172.17.0.2:7946  alive   role=serf-agent
31aa2ce79322  172.17.0.5:7946  alive   role=3306
```

现在你可以看到 Serf 已经知道了运行 MySQL 服务的内部 IP 地址。我们几乎快完成了。

## 创建一个WordPress容器

我们现在需要一个 WordPress 容器来使用 Serf 中注册的服务。我们有两种方法来实现:

1. 我们可以在容器中构建 Serf－logic 。

2.我们可以创建另外一个 ambassador 容器来桥接到 MySQL ambassador ，并且链接到 WordPress 容器（这个有点违背了我们一开始使用 Serf 的初衷。除非你的新容器使用Serf，而不是根据环境变量来决定要链接到哪里）。

在这个情况下，我们会采用第一种方法，你们可以把方法二作为课外练习。

```
$ docker run -d --link serf_1:serf_1 -p 80:80 -e "DB_USER=root" -e "DB_NAME=mysql" ctlc/wordpress-serf
```

现在回过头来看一下另外一个运行着 ctlc/serf-members 容器的终端窗口。

```
67ffbe3dba95  172.17.0.3:7946  alive   role=serf-members
f5af7d9dbc3c  172.17.0.2:7946  alive   role=serf-agent
31aa2ce79322  172.17.0.5:7946  alive   role=3306
ffa1b001a1b7  172.17.0.6:7946  alive   role=web
```

成功。

```
$ curl -s -L localhost | head
<meta name="viewport" content="width=device-width" /><meta http-equiv="Content-Type" content="text/html; charset=utf-8" />WordPress › Installation
		<link id="buttons-css" href="http://localhost/wp-includes/css/buttons.min.css?ver=3.9-beta3-27857" rel="stylesheet" media="all" type="text/css" />
	<link id="open-sans-css" href="//fonts.googleapis.com/css?family=Open+Sans%3A300italic%2C400italic%2C600italic%2C300%2C400%2C600&subset=latin%2Clatin-ext&ver=3.9-beta3-27857" rel="stylesheet" media="all" type="text/css" />
	<link id="install-css" href="http://localhost/wp-admin/css/install.min.css?ver=3.9-beta3-27857" rel="stylesheet" media="all" type="text/css" />
```

## 在 Etcd 和 CoreOS 中使用 Ambassador

我在文章的一开始就保证会展示如何把数据库服务注册到 Etcd 而不是 Serf 中。如果你已经看到这里，那你就应该能够知道更多。准备好了么？

```
$ docker run -d --link mysql:mysql --link serf_1:serf_1 --name mysql_ambassador -p 3306:3306 ctlc/amb-serf
```

运行:

```
$ docker run -d --link mysql:mysql --link serf_1:serf_1 --name mysql_ambassador -p 3306:3306 ctlc/amb-etcd
```

ctlc/amb-etcd 容器会自动注册到 etcd 而不是 serf 。当然你需要把这个写入 systemd 标准的文件，使用 fleeetctl 来进行部署从而让它真正工作，不过概念是一样的。

## 总结

使用 Docker ambassador 模型你能做很多事情，这里仅仅提及了一些表面的东西。在后续的文章中，我们会向你展示如何通过 ambassador 实现中心化、透明化的网络日志，以及其他有趣的东西。欢迎订阅邮件列表进行学习。

---

#####这篇文章由 [Lucas Carlson](http://www.centurylinklabs.com/author/cardmagic/) 发表，点击 [这里](http://www.centurylinklabs.com/linking-docker-containers-with-a-serf-ambassador/?utm_source=Docker+News&utm_campaign=c3d355131c-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-c3d355131c-235722981) 可阅读原文。 [Mark Shao](https://github.com/markshao) 翻译了本文，您可以在 [GitHub](https://github.com/markshao) 上与他交流。

#####The article was contributed by [Lucas Carlson](http://www.centurylinklabs.com/author/cardmagic/) , click [here]((http://www.centurylinklabs.com/linking-docker-containers-with-a-serf-ambassador/?utm_source=Docker+News&utm_campaign=c3d355131c-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-c3d355131c-235722981)) to read the original publication.
