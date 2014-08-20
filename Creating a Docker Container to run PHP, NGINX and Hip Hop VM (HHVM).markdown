# 创建一个运行 PHP，NGINX 和 Hip Hop VM(HHVM) 的镜像

标签（空格分隔）： PHP NGINX HHVM Docker

---
> 注：本文由 [Mike Ebinum][1] 编写，原文地址 [Creating a Docker Container to run PHP, NGINX and Hip Hop VM (HHVM)][2]

![此处输入图片的描述][3]

对于 [Docker][4]，我感到非常的兴奋，作为一个开放人员，在早些时候，我花费了太多的时间陷入了 .NET　工作中我不喜欢的几件事情中，如在不同的环境中部署和测试。部署一个 web 应用程序的过程绝对是一个噩梦般的经历。并且甚至在那之后，我迁移到基于 UNIX 平台开发，然后使用开源的工具/语言，如 Node, Java, Scala, PHP 等等，我发现同样的部署问题一次又一次的发生。

使用如 Docker 这样的工具，你可以让你开发环境的配置精确的如生产环境的镜像一样。部署一个你的 web 应用程序的容器，任何东西都被配置了，你再也不用太担心关于部署的那些麻烦事。

如果你是一个 Docker 的新手，并且不是十分确定它是什么，以下这些文章能给你一个完美的学习纲要，去吧，读完它们，我等着。

 - [Docker Lightweight linux containers for consistent development and deployment][5]
 - [Docker: Using Linux Containers to Support Portable Application deployment][6]

作为一个懒惰的程序员，我的梦想成真了，只要做一次，然后你再也不用为它操心了（在一定程度上），无论如何你都不会来到这里对我咆哮，在这篇文章中，我将向你展示，为你开发环境基于以下怎样创建并且运行一个 Docker 容器。

 - [CentOS][7]
 - [Nginx web server][8]
 - PHP with [Hip Hop VM (HHVM)][9]

## Dockerfile

准备开始，我们创建一个 ```Dockerfile``` - 一个 Dockerfile 中包含怎样创建你想要的镜像的指令。

```
FROM    centos:centos6

MAINTAINER Mike Ebinum, hello@seedtech.io
```

## 使用 Cent OS 6.x

告知 Docker 使用官方社区最新版本的 CentOS 6.x  可用镜像。

## 更新镜像

安装所有最新版本的包更新，并且把 Red Hat EPEL 的仓库加入可用的仓库列表。

```
RUN yum update -y >/dev/null && yum install -y http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm  && curl -L -o /etc/yum.repos.d/hop5.repo "http://www.hop5.in/yum/el6/hop5.repo"
```

## 安装包

安装 ```supervisord ``` - 我们将使用这个配置和控制运行在容器中的进程 -, nginx, php, 一些 PHP 的开发包以及 Facebook 的 hhvm

```
RUN yum install -y python-meld3 http://dl.fedoraproject.org/pub/epel/6/i386/supervisor-2.1-8.el6.noarch.rpm

RUN ["yum", "-y", "install", "nginx", "php", "php-mysql", "php-devel", "php-gd", "php-pecl-memcache", "php-pspell", "php-snmp", "php-xmlrpc", "php-xml","hhvm"]
```

## 配置 Nginx, HHVM 和 Supervisord

为 nginx 创建目录，并且把 ```index.php``` 文件加入 nginx 来展现。

```
RUN mkdir -p /var/www/html && chmod a+r /var/www/html && echo "<?php phpinfo(); ?>" > /var/www/html/index.php
```

下一组指令是：

 - 为 HHVM 添加一个[配置文件][10]，然后重起我们的 HHVM 服务
 - 为 Supervisord 添加一个[配置文件][11]，然后启动 Nginx 和 HHVM
 ```
   ADD config.hdf /etc/hhvm/config.hdf 

  RUN service hhvm restart

  ADD nginx.conf /etc/nginx/conf.d/default.conf

  ADD supervisord.conf /etc/supervisord.conf

  RUN chkconfig supervisord on && chkconfig nginx on
 ```
 - 添加一个 shell 脚本 ```/run.sh```,当 Docker 容器正在运行的时候将启动

**run.sh**

```
#!/bin/bash
 
set -e -x
echo "starting supervisor in foreground"
supervisord -n
```
```
 ADD scripts/run.sh /run.sh

 RUN chmod a+x /run.sh 

 EXPOSE 22 80

 ENTRYPOINT ["/run.sh"]
```

构建容器，并且打 tag
```
docker build -t centos-nginx-php5-hhvm .
```

现在我们有一个全功能的容器，我们可以像下面这样运行他：

```
docker run -d -p 80:80 centos-nginx-php5-hhvm
```
如果你已经有本地的服务已经在运行并且占用了 80 端口，你能很容易的的改变容器的对外端口。

![此处输入图片的描述][12]

这个 Docker 镜像在 [docker registry][13] 的可用版本。

## Dockerfile

完整的 Dockerfile 如下

```
# DOCKER-VERSION 1.0.0
 
FROM    centos:centos6
 
MAINTAINER Mike Ebinum, hello@seedtech.io
 
# Install dependencies for HHVM
# yum update -y >/dev/null && 
RUN yum install -y http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm  && curl -L -o /etc/yum.repos.d/hop5.repo "http://www.hop5.in/yum/el6/hop5.repo"
 
# Install supervisor
RUN yum install -y python-meld3 http://dl.fedoraproject.org/pub/epel/6/i386/supervisor-2.1-8.el6.noarch.rpm
 
#install nginx, php, mysql, hhvm
RUN ["yum", "-y", "install", "nginx", "php", "php-mysql", "php-devel", "php-gd", "php-pecl-memcache", "php-pspell", "php-snmp", "php-xmlrpc", "php-xml","hhvm"]
 
# Create folder for server and add index.php file to for nginx
RUN mkdir -p /var/www/html && chmod a+r /var/www/html && echo "<?php phpinfo(); ?>" > /var/www/html/index.php
 
#Setup hhvm - add config for hhvm
ADD config.hdf /etc/hhvm/config.hdf 
 
RUN service hhvm restart
 
# ADD Nginx config
ADD nginx.conf /etc/nginx/conf.d/default.conf
 
# ADD supervisord config with hhvm setup
ADD supervisord.conf /etc/supervisord.conf
 
#set to start automatically - supervisord, nginx and mysql
RUN chkconfig supervisord on && chkconfig nginx on
 
ADD scripts/run.sh /run.sh
 
RUN chmod a+x /run.sh 
 
 
EXPOSE 22 80
#Start supervisord (which will start hhvm), nginx 
ENTRYPOINT ["/run.sh"]
```

在这篇文章中提到的其他的可用文件在 [Github][14] 上。

## 下一步?

太棒了,我们现在有了一个环境配置，但我如何运行PHP应用程序？好问题，我将做后续的文章说明通过使用这个容器如何安装和配置PHP应用程序。[订阅这个博客][15], 在 twitter 关注 [@mikeebinum][16] 和 [@SEEDtechio][17] 来获得更新



  [1]: https://plus.google.com/u/1/+MikeEbinum/
  [2]: http://blog.seedtech.io/post/91801062414/creating-a-docker-container-to-run-php-nginx-and-hip
  [3]: http://media.tumblr.com/da087e7a4e37c42c9357d9846141f602/tumblr_inline_n8otvj69fU1sbfcgv.png
  [4]: http://docker.io/
  [5]: http://www.linuxjournal.com/content/docker-lightweight-linux-containers-consistent-development-and-deployment
  [6]: http://www.infoq.com/articles/docker-containers
  [7]: http://www.centos.org/
  [8]: http://nginx.org/
  [9]: http://hhvm.com/
  [10]: https://github.com/mebinum/dockerfiles/blob/master/centos-nginx-php-hhvm/config.hdf
  [11]: https://github.com/mebinum/dockerfiles/blob/master/centos-nginx-php-hhvm/supervisord.conf
  [12]: http://media.tumblr.com/389c2d0de9aa93f5ad1c9b157e1228da/tumblr_inline_n8oso9TmJU1sbfcgv.png
  [13]: https://registry.hub.docker.com/u/mebinum/centos-nginx-php5-hhvm/
  [14]: http://github.com/mebinum/dockerfiles
  [15]: https://www.tumblr.com/register/follow/seedtech
  [16]: https://twitter.com/mikeebinum
  [17]: https://twitter.com/seedtechio