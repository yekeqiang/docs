# 结合Salt使用Docker #
>注: 原文地址[Combining Salt with Docker](http://blog.xebia.com/2014/06/14/combining-salt-with-docker/), 作者eembsen

你可以用Salt去构建和运行Docker容器，但是我不是这么用的。这篇博客介绍一个实验，该实验在Docker容器中运行Salt minions。使用场景？假设你有运行特定中间件若干容器，并且这些中间件都需要一个安全升级，比如，一个OpenSSL的补丁。这个时候就需要很快的执行更新。

###Dockerfile

```bash
#-------
# Standard heading stuff

FROM centos
MAINTAINER No Reply noreply@xebia.com

# Do Salt install stuff and squeeze in a master.conf snippet that tells the minion
# to contact the master specified.

RUN rpm -Uvh http://ftp.linux.ncsu.edu/pub/epel/6/i386/epel-release-6-8.noarch.rpm
RUN yum install -y salt-minion --enablerepo=epel-testing
RUN [ ! -d /etc/salt/minion.d ] && mkdir /etc/salt/minion.d
ADD ./master.conf /etc/salt/minion.d/master.conf

# Run the Salt Minion and do not detach from the terminal.
# This is important because the Docker container will exit whenever
# the CMD process exits.

CMD /usr/bin/salt-minion
#-------

```
###构建镜像
是时候通过docker运行Dockerfile了。使用的命令是:

```bash
$ docker build --rm=true -t salt-minion .
```
假设你运行这条命令时和Dockerfile以及master.confg在同一目录。Docker会创建一个镜像，其标签为'salt-minion'，同时，在构建成功后，会删除所有的中间镜像。

###运行一个容器
使用的命令是:

```bash
$ docker run -d salt-minion
```
Docker 会返回:

```bash
aab154310ba6452ba2c686d15b1e3ca5fd85124d38c7935f1200d33b3a3e7ced
```

容器中运行的Salt minion会搜索并连接Salt master，Salt master定义在配置文件```/etc/salt/minion.d/master.conf```的“master”选项中。或许你应该将Salt master运行在“auto_accept”模式，这样的话minion的key会被自动接收。Docker会给运行的容器指派一个ID。这是docker运行容器后返回内容的关键。

下面的命令可以列出正在运行的容器:

```bash
$ docker ps
CONTAINER ID        IMAGE                COMMAND                CREATED             STATUS              NAMES
273a6b77a8fa        salt-minion:latest   /bin/sh -c /etc/rc.l   3 seconds ago       Up 3 seconds        distracted_lumiere
```

###应用补丁

看: 你的Salt master控制着Salt minion。假设你有一个状态模块包含了OpenSSL的补丁，你可以很轻易的更新所有的docker节点去包含补丁:

```bash
salt \* state.sls openssl-hotfix
```

到这里，一切都搞定了~
