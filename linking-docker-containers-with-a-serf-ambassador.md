# Linking Docker Containers with a Serf Ambassador
# 使用SERF AMBASSADOR来链接DOCKER CONTAINER

A few months ago, we wrote about using Serf with Docker to create a service registry so that your app could talk to your database indirectly. Then just a couple weeks ago, we wrote about using the Docker ambassadors model.

几个月前，我们写了一片关于[使用Serf结合Docker](http://www.centurylinklabs.com/decentralizing-docker-how-to-use-serf-with-docker/)实现服务的注册，这样你的应用就可以自动地和你的数据库服务链接在一起了。接着在数周前，我们又写了一篇关于如何使用[Docker ambassadors](http://www.centurylinklabs.com/deploying-multi-server-docker-apps-with-ambassadors/)模型的文章。

But in the Docker ambassadors post, we just setup a proxy ambassador, it didn’t do anything but forward TCP traffic.

但是在Docker ambassadors这片文章中，我们仅仅部署了一个代理ambassador,它除了转发TCP的请求啥都不做。

This week, we are going to use Docker ambassadors to do something more interesting: the ambassador will register the app container and the database container into the Serf network for us. This could just as easily be using ambassadors to register into etcd for us (in fact, I will show how at the end of this post). But for now, let’s start with Serf.

这个兴起，我们将要使用Docker ambassadors来做一些有趣的东西。ambassador将会帮我们把APP Container和数据库Container的信息注册到Serf网络中。这个和使用ambassadors把服务注册到etcd一样简单。(事实上，我最后会展示怎么做)。但是现在，我们先从Serf开始。

## Setup Serf With Docker
## 用Docker来建立Serf

First, let’s setup a serf agent on a Docker container.
首先，我们在一个Docker的容器上建立一个serf的代理。

```
$ SERF_ID=$(docker run -d --name serf_1 -p 7946 -p 7373 ctlc/serf /run.sh)
```

Well that was easy! Now let’s connect to the Serf agent with a Serf client container that will print out any members added to the network automatically.

那个太简单了！现在我们使用一个装有Serf客户端的container来链接到Serf代理服务，它会自动打印所有链接到网络上的成员。

```
$ docker run -i -t --link serf_1:serf_1 ctlc/serf-members
67ffbe3dba95  172.17.0.3:7946  alive  role=serf-members
f5af7d9dbc3c  172.17.0.2:7946  alive  role=serf-agent
```

So now we can see our progress as we continue.
所以现在我们可以看到我们的进度，我们继续下去。

## Setup a Random MySQL Container
## 建立一个随机的MySQL Container

Now we need to setup a MySQL database. I will pick a completely random MySQL database container from the Docker index:

腺癌我们需要建立一个MySQL数据库，我会在Docker Index中找一个完全随机的MySQL数据库Container:

```
$ docker run --name mysql -p 3306 -d brice/mysql
$ docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                                              NAMES
385c51c9de5a        brice/mysql:latest          /bin/sh -c mysqld   18 minutes ago      Up 18 minutes       0.0.0.0:49169->3306/tcp                            mysql   
67ffbe3dba95        ctlc/serf-members:latest   /run.sh             51 minutes ago      Up 51 minutes       7373/tcp, 7946/tcp                                 sleepy_brown       
f5af7d9dbc3c        ctlc/serf:latest           /run.sh             51 minutes ago      Up 51 minutes       0.0.0.0:49159->7373/tcp, 0.0.0.0:49160->7946/tcp   ...  
```

This is not a trusted image, so we actually have no idea what code we are running. However, brice/mysql is probably pretty unlikely to be running Serf since I picked it randomly from the index. So how do we get this database registered into Serf? Using the ambassador model of course.

这个不是一个受信任的镜像，所以我们也完全不知道到底在跑什么code。然而,brice/mysql也许不太可能来运行Serf，因为它是随机从index中找的一个镜像。那么我们怎么把它注册到Serf呢？当然就要使用ambassador模型。

```
$ docker run -d --link mysql:mysql --link serf_1:serf_1 --name mysql_ambassador -p 3306:3306 ctlc/amb-serf
```

Now look back at your other terminal window running the ctlc/serf-members container

现在回过头来看一下另外一个运行着ctlc/serf-members容器的终端窗口。

```
67ffbe3dba95  172.17.0.3:7946  alive   role=serf-members
f5af7d9dbc3c  172.17.0.2:7946  alive   role=serf-agent
31aa2ce79322  172.17.0.5:7946  alive   role=3306
```

Now you can see that Serf now knows the internal IP address of the host running MySQL. We are almost done
现在你可以看到Serf已经知道了运行MySQL服务的内部IP地址。我们几乎快完成了。

## Setup a WordPress Container
## 建立一个WordPress容器

Now all we need is a WordPress container that can consume the information from Serf. We can do this two ways:

我们现在需要一个WordPress容器来使用Serf中注册的服务。我们有两种方法来实现:

1. We can build Serf-logic into the container
1. 我们可以在容器中构件Serf－logic
2. We can create another ambassador container to bridge to the MySQL ambassador and link to the WordPress container (which kinda defeats the purpose of using Serf in the first place, unless your new container uses Serf instead of environment variables to figure out where to bridge to)

In this case, we will pick option 1 and leave option 2 as an exercise for the reader.
2.我们可以创建另外一个ambassador容器来桥接到MySQL ambassador，并且链接到WordPress容器。(这个有点违背了我们一开始使用Serf的初衷，除非你新的container使用Serf而不是环境变量来决定需要链接到哪里)

在这个情况下，我们会采用第一个方法，把第二个方法作为读者的练习。

```
$ docker run -d --link serf_1:serf_1 -p 80:80 -e "DB_USER=root" -e "DB_NAME=mysql" ctlc/wordpress-serf
```
Now look back at your other terminal window running the ctlc/serf-members container

现在回过头来看一下另外一个运行着ctlc/serf-members容器的终端窗口。

```
67ffbe3dba95  172.17.0.3:7946  alive   role=serf-members
f5af7d9dbc3c  172.17.0.2:7946  alive   role=serf-agent
31aa2ce79322  172.17.0.5:7946  alive   role=3306
ffa1b001a1b7  172.17.0.6:7946  alive   role=web
```

Success.
成功。

```
$ curl -s -L localhost | head
<meta name="viewport" content="width=device-width" /><meta http-equiv="Content-Type" content="text/html; charset=utf-8" />WordPress › Installation
		<link id="buttons-css" href="http://localhost/wp-includes/css/buttons.min.css?ver=3.9-beta3-27857" rel="stylesheet" media="all" type="text/css" />
	<link id="open-sans-css" href="//fonts.googleapis.com/css?family=Open+Sans%3A300italic%2C400italic%2C600italic%2C300%2C400%2C600&subset=latin%2Clatin-ext&ver=3.9-beta3-27857" rel="stylesheet" media="all" type="text/css" />
	<link id="install-css" href="http://localhost/wp-admin/css/install.min.css?ver=3.9-beta3-27857" rel="stylesheet" media="all" type="text/css" />
```

## Ambassadors for Etcd and CoreOS
## 在Etcd和CoreOS中使用Ambassadors

I promised I would show you how to register your database into Etcd instead of Serf at the beginning of this article. If you’ve made it this far, you deserve to see more. Ready for it? Instead of this:

我在文章的一开始就保证会展示如何把你的数据库服务注册到Etcd而不是Serf中。如果你已经看到这里，那你就应该能够知道更多。准备好了么？

```
$ docker run -d --link mysql:mysql --link serf_1:serf_1 --name mysql_ambassador -p 3306:3306 ctlc/amb-serf
```
Run this:
运行这个:

```
$ docker run -d --link mysql:mysql --link serf_1:serf_1 --name mysql_ambassador -p 3306:3306 ctlc/amb-etcd
```

The ctlc/amb-etcd container will register with etcd instead of serf automatically. Of course you will need to put this in systemd formatted file and deploy using fleetctl for this to actually work, but the idea is the same.

ctlc/amb-etcd容器会自动注册到etcd而不是serf. 当然你需要把这个写入systemd标准的文件，使用fleeetctl来进行部署从而让它真正工作,但是概念是一样的。

## CONCLUSION
## 总结

There is a lot you can do with the Docker ambassador model. This is just scratching the surface. In upcoming posts, we will show you how to do things like centralized transparent network logging and other fun stuff you can do with ambassadors. Make sure you subscribe to the mailing list (in the top bar above this blog post) to stay in touch.

这里写了很多关于如何使用Docker ambassador模型。这个也仅仅提及了一些表面的东西。在未来的文章中，我们会向你展示通过ambassador实现的例如中心化，透明化的网络日志等其他有趣的东西。确保你订阅了邮件列表(在这片文章的顶部)来保持练习