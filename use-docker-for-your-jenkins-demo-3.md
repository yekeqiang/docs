# 使用docker来提升你的Jenkins演示 - 3

---

##### 作者：蔡煜

---

## 系列介绍

使用docker来提升你的Jenkins演示会安排五篇文章循序渐进地学习：

- 第一篇： **[使用docker来提升你的Jenkins演示 - 1](http://www.larrycaiyu.com/2014/11/04/use-docker-for-your-jenkins-demo-1.html)** ：把需要的Jenkins软件、插件和作业配置放到了Jenkins docker容器中，使得你很容易演示，别人也很轻松的可以下载自己尝试。
- 第二篇： **[使用docker来提升你的Jenkins演示 - 2](http://www.larrycaiyu.com/2014/11/16/use-docker-for-your-jenkins-demo-2.html)** ：演示了如何更进一步docker化你的Jenkins应用程序，学到了如何管理配置文件和巧妙地使用run --volume和exec来获取容器内的数据。
- 第三篇： **[使用docker来提升你的Jenkins演示 - 3](http://www.larrycaiyu.com/2014/11/24/use-docker-for-your-jenkins-demo-3.html)** ：用docker中的docker来解决Jenkins主从机的演示
- 第四篇：**巧用ENTRYPOINT/CMD配合脚本来解决复杂的环境**
- 第五篇：**讨厌的 Jenkins docker 插件**

## 回顾

前两篇还是很简单，搭了个架子而已，在 **Jenkins** 的持续集成部署中，它的一个重要的使用方式就是构建的实际任务放在 **Slave**（从属）机器上， **Jenkins Master**（主机）主要负责调度。这样能确保干净的构建环境和整体性能。

![alt](http://resource.docker.cn/jenkins-demo3-1.png)

本文就是这个博客系列的第三篇文章。我会用一些例子一步一步来说明如何用 docker 实现这种演示，如果你对 Jenkins 的这种用法还不熟悉的话，正好可以一起学习。

## 用 docker 容器作为从属机器

现在我们考虑来个 C++ 编译的 Jenkins 任务，在 Jenkins 中为了不污染主机和提高性能，一般都把具体的累活在从属机上实现。

既然 jenkins 主机已经用 docker 了，从属机当仁不让的也用 docker 了，这个已经有很多文章了，我就用了我以前做的一个镜像 larrycai/jenkins-slave-ubuntu ，可以查看 [Dockerfile](https://github.com/larrycai/jenkins-docker-demo1/blob/master/jenkins-slave-ubuntu/Dockerfile) ，里面主要有以下几个：

1. 1openjdk-7-jdk1 包， jenkins 从属机运行环境
2. sshd 服务
3. ssh的公钥（public key)，当然密码也有。
4. 为了演示方便，镜像里面已经预先下载好了一个小的 c++ 程序 [outhome](https://github.com/larrycai/out2html/archive/master.zip) 仓库。

先把它启动后在2222端口监听。

```
docker@boot2docker:~$ docker run -d -p 2222:22 larrycai/jenkins-slave-ubuntu 
docker@boot2docker:~$ docker ps
CONTAINER ID        IMAGE                                  COMMAND                CREATED             STATUS              PORTS                    NAMES
4d1189c2c7be        larrycai/jenkins-slave-ubuntu:latest   "/usr/sbin/sshd -D"    16 minutes ago      Up 16 minutes       0.0.0.0:2222->22/tcp     boring_morse
```

## 创建新环境 craft3

把上次的 [larrycai/jenkins-demo2](https://github.com/larrycai/docker-images/tree/master/jenkins-demo2) 构建目录拷贝一份到新的工作环境，这次叫 `larrycai/jenkins-demo3` ，重新构建，启动（别忘了上次怎么做 `JENKINS_HOME` ）。

```
docker@boot2docker:$ docker build -t larrycai/jenkins-demo3 .
docker@boot2docker:$ docker run -v ~/jenkins:/data -p 8080:8080 -it larrycai/jenkins-demo3
```

先在系统中配好从节点，如下：

![alt](http://resource.docker.cn/jenkins-demo3-2.png)

`Host` 要选 `172.17.42.1` 这是 boot2docker 的主机 IP 地址，每个 docker 容器都能访问，端口就是映射出的 2222 端口。这里也可以直接访问从属机的IP地址，那么端口就是标准的 `22` 了。

其中还要配好 ssh 访问的私钥。

![alt](http://resource.docker.cn/jenkins-demo3-3.png)

最后创建好新的工作 craft3 。

![alt](http://resource.docker.cn/jenkins-demo3-4.png)

里面配好从属机和工作脚本，运行一下看看

![alt](http://resource.docker.cn/jenkins-demo3-5.png)

一级棒，它工作了。

用上一篇博客教的方式把数据拷贝出来放到镜像里，重新构建镜像实验一下。

- $JENKINS_HOME/credentials.xml # 这是一些Jenkins的认证信息（私钥）。
- $JENKINS_HOME/config.xml # 包含了从属节点的配置

## 自动启动从属节点

现在从属节点是手工下载和启动的，那么在演示时怎么能够自动启动从属节点呢，一般有以下种办法

1. 用脚本或编配（ orechstration ）工具如 fig 来启动多个 docker 镜像
2. 在 Jenkins 任务里面启动 docker 从属节点
3. 在 docker 容器启动 docker 从属节点
4. 用 [Jenkins Docker 插件](https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin)

第一种用脚本的方式肯定可以，但是感觉有点老土，而且还要下载其他脚本，何况也不能确定 fig 等工具是否能运行。

第四种情况听起来不错，但是坑很多，留待以后的博客专门来批判。

第二、第三种都需要要用到 docker 里的 docker 技术，其中第二种需要改变 jenkins 里的任务脚本，不够简洁，下面着重讨论第三种。

### docker 里的 docker

Docker是有客户端和服务器Daemon端组成，具体请看 InfoQ 上的 [Docker 源码分析（一）：Docker 架构](http://www.infoq.com/cn/articles/docker-source-code-analysis-part1) 。

所以我们可以在镜像装好docker客户端，很简单，就是一个二进制文件而已。

```
RUN curl https://get.docker.io/builds/Linux/x86_64/docker-latest -o /usr/local/bin/docker
RUN chmod +x /usr/local/bin/docker
```

为了访问，我们必须传递访问的套接字 `/var/run/docker.sock` ，这个用我们学过的标记 `-v` 就可以了。

我们把这段写在已有的 `start.sh` 脚本中，放在启动 `jenkins` 之前。

```
#!/bin/bash

export DOCKER_HOST=unix://docker.sock
docker run -d -p 2222:22 larrycai/jenkins-slave-ubuntu

exec java -jar /opt/jenkins/jenkins.war
```

重新构建运行一下（别忘了传递 Docker 主机的 docker.sock ）。

```
docker run -v /var/run/docker.sock:/docker.sock -p 8080:8080 -it larrycai/jenkins-demo3
```
就这么简单！是的，docker 就是简洁，永远的 v5 。

当然现在的 `start.sh` 还是比较简单，如果第二次运行或者想每次强制更新从属机的镜像还不行，留待下次讨论，不过你可以想想，如果是你怎么解决。

## 摘要

在这篇博客中，我们讲解了如何用 Docker 演示 Jenkins 持续集成的主从应用实践，和利用 docker 中的 docker 技术来自动启动从属机器。

所有的代码你都可以在 [github 上的 jenkins-demo3](https://github.com/larrycai/docker-images/tree/master/jenkins-demo3) 上找到。

现在，您可以更新您的 Jenkins 新功能打包到 Docker 到处演示。

在接下来的博客中，我将展示如何更好地用docker和脚本来控制jenkins的从属节点。

Docker可以帮助我们做很多事情，关注新浪微博 [@larrycaiyu](http://weibo.com/larrycaiyu) 、 [@Docker中文社区](http://weibo.com/dockboard) 、 [@infoQ的Docker专栏](http://www.infoq.com/cn/dockers) 。

---

本文原载于 [蔡煜](http://weibo.com/larrycaiyu) 的个人博客，原文地址：[使用docker来提升你的Jenkins演示 - 3](http://www.larrycaiyu.com/2014/11/24/use-docker-for-your-jenkins-demo-3.html)