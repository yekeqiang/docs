#开始使用 Docker


#####作者：[Arun Gupta](https://twitter.com/arungupta) 

#####译者：[我不围观](http://weibo.com/ooutman)
***

如果把来自不同会议和 meetup 上的演讲、 twitter 上的推文 、还有其它演讲稿累加，好像 Docker 有解决世界饥荒的节奏了。真是这样的话真真是极好的，不过显然还不行。但是 Docker 确实能很好的解决一些问题。

我们来听听 Docker 项目的发起人 [Solomon Hykes](http://twitter.com/solomonstre) 是如何阐述的：

<embed src="http://player.youku.com/player.php/sid/XNzI3ODg2NTM2/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

简单来说， Docker 简化了软件的分发过程，它使制作和共享包含软件环境的镜像文件变得简单了。这些镜像文件也叫**应用程序操作系统** 

##什么是应用操作系统？

你的应用程序一般都需要特定版本的操作系统、应用服务器、 JDK 、数据库服务器，还可能需要调整配置文件和其他一些依赖关系。应用程序可能需要绑定到指定的端口和一定量的内存。这些运行应用程序所需要的组件和配置就是所说的应用程序操作系统。

你当然可以写一个包含下载和安装这些组件的安装脚本。 Docker 简化了这个流程，通过创建一个包含应用程序和基础设施的镜像(image)，当作一个组件进行管理。这些镜像可以创建 Docker *容器(container)*，容器运行在 Docker 提供的*容器虚拟化平台*上。

##Docker的主要组件有哪些？

Docker 有两个主要组件：

* Docker：开源的容器虚拟化平台
* Docker Hub：共享和管理 Docker 镜像的 Saas 平台

Docker 采用 [Linux 容器](https://linuxcontainers.org/) 来提供隔离、沙箱、复制、资源限制、快照和其他的一些优势。详细信息的可以看 [excellent piece at InfoQ on Docker Containers](http://www.infoq.com/articles/docker-containers) 了解。

镜像是 Docker 的“构建组件”，也是应用操作系统的只读模版。容器是从镜像创建出来的运行状态，是 Docker 的“运行组件”。容器是可以运行、启动、停止、移动和删除的。镜像保存的仓库是 Docker 的“分发组件”。

Docker **按启动顺序**包含两个组件：


* 服务端：运行在宿主机上，负责构建、运行和分发 Docker 容器等重要工作
* 客户端：Docker 二进制程序，接收用户的命令和服务程序进行通信

##这些组件怎么一起工作？

客户端可以和服务端运行在一台主机上，也可以在不同的主机上。服务端需要用 [pull](https://docs.docker.com/reference/commandline/cli/#pull) 命令从仓库中拉一个镜像下来。服务端可以从 Docker Hub 或者其他配置的仓库中下载镜像。服务端主机可以从仓库中下载和安装多个镜像。

{<1>}![](http://resource.docker.cn/docker-architecture-techtip39.png)

然后客户端就可以用 [run命令](https://docs.docker.com/reference/commandline/cli/#run) 来启动容器。完整的客户端命令列表可以看 [这里](https://docs.docker.com/reference/commandline/cli/) 。

客户端和服务端通过 socket 或者 REST API 进行通信。

##因为 Docker 使用了 Linux 内核特性，是不是说只能在基于 Linux 的系统上使用？

不同操作系统的 Docker 服务端和客户端都可以从 <https://docs.docker.com/installation> 安装。你也看到了，其实 Docker 是可以在很多平台上安装的，包括 Mac 和 Windows 上。

对于非 Linux 的系统，需要安装一个轻量级的虚拟机，在虚拟机上安装 docker 服务程序。同时会安装一个原生的客户程序，可以和服务程序进行通信。下面是一个 Mac 上启动 docker 服务程序的日志：

```
bash
unset DYLD_LIBRARY_PATH ; unset LD_LIBRARY_PATH
mkdir -p ~/.boot2docker
if [ ! -f ~/.boot2docker/boot2docker.iso ]; then cp /usr/local/share/boot2docker/boot2docker.iso ~/.boot2docker/ ; fi
/usr/local/bin/boot2docker init 
/usr/local/bin/boot2docker up && export DOCKER_HOST=tcp://$(/usr/local/bin/boot2docker ip 2>/dev/null):2375
docker version
~> bash
~> unset DYLD_LIBRARY_PATH ; unset LD_LIBRARY_PATH
~> mkdir -p ~/.boot2docker
~> if [ ! -f ~/.boot2docker/boot2docker.iso ]; then cp /usr/local/share/boot2docker/boot2docker.iso ~/.boot2docker/ ; fi
~> /usr/local/bin/boot2docker init 
2014/07/16 09:57:13 Virtual machine boot2docker-vm already exists
~> /usr/local/bin/boot2docker up && export DOCKER_HOST=tcp://$(/usr/local/bin/boot2docker ip 2>/dev/null):2375
2014/07/16 09:57:13 Waiting for VM to be started...
.......
2014/07/16 09:57:35 Started.
2014/07/16 09:57:35 To connect the Docker client to the Docker daemon, please set:
2014/07/16 09:57:35     export DOCKER_HOST=tcp://192.168.59.103:2375
~> docker version
Client version: 1.1.1
Client API version: 1.13
Go version (client): go1.2.1
Git commit (client): bd609d2
Server version: 1.1.1
Server API version: 1.13
Go version (server): go1.2.1
Git commit (server): bd609d2
```

举个例子，在 Mac 上可以按照 [这个教程](https://docs.docker.com/installation/mac) 安装 Docker 服务端和客户端。

可以通过下面这个命令停止虚拟机：

```
boot2docker stop
```

然后这样重启：

```
boot2docker boot
```

这样登录：

```
boot2docker ssh
```

完整的 boot2docker 命令列表可以在帮助上看到：


```
~> boot2docker help
Usage: boot2docker []  []

boot2docker management utility.

Commands:
    init                    Create a new boot2docker VM.
    up|start|boot           Start VM from any states.
    ssh [ssh-command]       Login to VM via SSH.
    save|suspend            Suspend VM and save state to disk.
    down|stop|halt          Gracefully shutdown the VM.
    restart                 Gracefully reboot the VM.
    poweroff                Forcefully power off the VM (might corrupt disk image).
    reset                   Forcefully power cycle the VM (might corrupt disk image).
    delete|destroy          Delete boot2docker VM and its disk image.
    config|cfg              Show selected profile file settings.
    info                    Display detailed information of VM.
    ip                      Display the IP address of the VM's Host-only network.
    status                  Display current state of VM.
    download                Download boot2docker ISO image.
    version                 Display version information.
```

##光说不练，举个栗子呗？

一些 JBoss 项目可以在 [这里](www.jboss.org/docker) 找到 Docker 镜像，在页面上还可以看到安装的详细命令。比如， WildFly 的 Docker 镜像就可以这么安装：

```
~> docker pull jboss/wildfly
Pulling repository jboss/wildfly
2f170f17c904: Download complete 
511136ea3c5a: Download complete 
c69cab00d6ef: Download complete 
88b42ffd1f7c: Download complete 
fdbe853b54e1: Download complete 
bc93200c3ba0: Download complete 
0daf76299550: Download complete 
3a7e1274035d: Download complete 
e6e970a0db40: Download complete 
1e34f7a18753: Download complete 
b18f179f7be7: Download complete 
e8833789f581: Download complete 
159f5580610a: Download complete 
3111b437076c: Download complete
```

下载下来的镜像可以用这个命令来验证一下：

```
~> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
jboss/wildfly       latest              2f170f17c904        8 hours ago         1.048 GB
```

镜像下载下来了，就可以用如下的命令来启动容器：

```
docker run jboss/wildfly
```

默认情况下， Docker 容器是不提供交互 shell 的，也不提供标准输入。如果 WildFly 的 Docker 容器用上面的命令启动的话，就不能用 Ctrl+C 来停止了。可以指定 -i 选项来使其可交互， -t 选项分配一个伪终端。

另外，我们通常还想在容器外能够访问8080端口，比如运行容器的宿主机上。可以通过指定 -p 80:8080 来完成，其中80是宿主机的端口，8080是容器的端口。 

我们就可以这样运行：

```
docker run -i -t -p 80:8080 jboss/wildfly
=========================================================================

  JBoss Bootstrap Environment

  JBOSS_HOME: /opt/wildfly

  JAVA: java

  JAVA_OPTS:  -server -Xms64m -Xmx512m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true

=========================================================================

22:08:29,943 INFO  [org.jboss.modules] (main) JBoss Modules version 1.3.3.Final
22:08:30,200 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.2.Final
22:08:30,297 INFO  [org.jboss.as] (MSC service thread 1-6) JBAS015899: WildFly 8.1.0.Final "Kenny" starting
22:08:31,935 INFO  [org.jboss.as.server] (Controller Boot Thread) JBAS015888: Creating http management service using socket-binding (management-http)
22:08:31,961 INFO  [org.xnio] (MSC service thread 1-7) XNIO version 3.2.2.Final
22:08:31,974 INFO  [org.xnio.nio] (MSC service thread 1-7) XNIO NIO Implementation Version 3.2.2.Final
22:08:32,057 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 31) WFLYIO001: Worker 'default' has auto-configured to 16 core threads with 128 task threads based on your 8 available processors
22:08:32,108 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 32) JBAS010280: Activating Infinispan subsystem.
22:08:32,110 INFO  [org.jboss.as.naming] (ServerService Thread Pool -- 40) JBAS011800: Activating Naming Subsystem
22:08:32,133 INFO  [org.jboss.as.security] (ServerService Thread Pool -- 45) JBAS013171: Activating Security Subsystem
22:08:32,178 INFO  [org.jboss.as.jsf] (ServerService Thread Pool -- 38) JBAS012615: Activated the following JSF Implementations: [main]
22:08:32,206 WARN  [org.jboss.as.txn] (ServerService Thread Pool -- 46) JBAS010153: Node identifier property is set to the default value. Please make sure it is unique.
22:08:32,348 INFO  [org.jboss.as.security] (MSC service thread 1-3) JBAS013170: Current PicketBox version=4.0.21.Beta1
22:08:32,397 INFO  [org.jboss.as.webservices] (ServerService Thread Pool -- 48) JBAS015537: Activating WebServices Extension
22:08:32,442 INFO  [org.jboss.as.connector.logging] (MSC service thread 1-13) JBAS010408: Starting JCA Subsystem (IronJacamar 1.1.5.Final)
22:08:32,512 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-9) JBAS017502: Undertow 1.0.15.Final starting
22:08:32,512 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 47) JBAS017502: Undertow 1.0.15.Final starting
22:08:32,570 INFO  [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool -- 27) JBAS010403: Deploying JDBC-compliant driver class org.h2.Driver (version 1.3)
22:08:32,660 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-10) JBAS010417: Started Driver service with driver-name = h2
22:08:32,736 INFO  [org.jboss.remoting] (MSC service thread 1-7) JBoss Remoting version 4.0.3.Final
22:08:32,836 INFO  [org.jboss.as.naming] (MSC service thread 1-15) JBAS011802: Starting Naming Service
22:08:32,839 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-15) JBAS015400: Bound mail session [java:jboss/mail/Default]
22:08:33,406 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 47) JBAS017527: Creating file handler for path /opt/wildfly/welcome-content
22:08:33,540 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-13) JBAS017525: Started server default-server.
22:08:33,603 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-8) JBAS017531: Host default-host starting
22:08:34,072 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-13) JBAS017519: Undertow HTTP listener default listening on /0.0.0.0:8080
22:08:34,599 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-11) JBAS015012: Started FileSystemDeploymentService for directory /opt/wildfly/standalone/deployments
22:08:34,619 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-9) JBAS010400: Bound data source [java:jboss/datasources/ExampleDS]
22:08:34,781 INFO  [org.jboss.ws.common.management] (MSC service thread 1-13) JBWS022052: Starting JBoss Web Services - Stack CXF Server 4.2.4.Final
22:08:34,843 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://0.0.0.0:9990/management
22:08:34,844 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://0.0.0.0:9990
22:08:34,845 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.1.0.Final "Kenny" started in 5259ms - Started 184 of 233 services (81 services are lazy, passive or on-demand)
```

容器的 IP 地址可以这样看：

```
~> boot2docker ip

The VM's Host only interface IP address is: 192.168.59.103
```

已经启动的容器可以用如下的命令来验证：

```
~> docker ps
CONTAINER ID        IMAGE                  COMMAND                CREATED             STATUS              PORTS                NAMES
b2f8001164b0        jboss/wildfly:latest   /opt/wildfly/bin/sta   46 minutes ago      Up 12 minutes       8080/tcp, 9990/tcp   sharp_pare
```

现在可以在本地机器上通过 <http://192.168.59.103> 来访问 WildFly 服务器，结果就像这样：

{<2>}![](http://resource.docker.cn/wildfly-output-techtip39-942x1024.png)

最后，容器可以通过 Ctrl+C 来停止，或者用如下命令：

```
~> docker stop b2f8001164b0
b2f8001164b0
```

命令中的容器 id(b2f8001164b0) 可以通过“ docker ps ”来得到。

更多的镜像使用指南，比如启动域名模式、部署应用等可以在 [这里](https://github.com/jboss/dockerfiles/blob/master/wildfly/README.md) 找到。

你还想知道 WildFly 的 Docker 镜像包含什么？欢迎给我们提问题，在我们的 [Github](https://github.com/jboss/dockerfiles/issues) 页面留言。

<http://jboss.org/docker> 上有一些其他可用的镜像：

* [KeyCloak](http://keycloak.org/)
* [TorqueBox](http://torquebox.org/)
* [Immutant](http://immutant.org/)
* [LiveOak](http://liveoak.io/)
* [AeroGear](http://aerogear.org/)

![](http://resource.docker.cn/keycloak-200x150.png) ![](http://resource.docker.cn/immutant-200x150.png) ![](http://resource.docker.cn/torquebox-200x150.png) ![](http://resource.docker.cn/keycloak-200x150.png)

你知道 Red Hat 在 [Docker 贡献者中名列前茅](http://thenewstack.io/who-are-the-docker-developers/) 吗？有5名来自 [Project Atomic](http://www.projectatomic.io/) 的红帽子参与其中。

***
#####这篇文章由 [Arun Gupta](https://twitter.com/arungupta)   撰写，[我不围观](http://weibo.com/ooutman) 翻译。点击 [这里](http://blog.arungupta.me/2014/07/getting-started-with-docker/) 阅读原文。

#####The article was contributed by [Arun Gupta](https://twitter.com/arungupta) , click [here](http://blog.arungupta.me/2014/07/getting-started-with-docker/) to read the original publication.
***

