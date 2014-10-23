# 使用 Docker 快速搭建 Java 测试环境

> 本文将以 Java 为例，讲解如何通过 Dockerfile 来创建自己的开发环境。同时讲解一些 Docker 的基本命令


随着容器虚拟化被越来越多的开发人员接受，敏捷开发、 DevOps 的流行。越来越多的开发者尝试通过 Docker 提供的虚拟化方式，给开发团队建立一套具备一致性可以复用的开发、测试环境，让开发环境可以通过 Image 的形式分享给项目的所有开发成员，以简化开发环境的搭建。当然在 Docker 技术之前就已经有如 Vagrant 的开发环境分发技术，但是本文的目的是通过开发环境的搭建来让读者了解 Docker 技术的基本使用。同时要说明的是与 Vagrant 等开发环境分发技术相比， Docker 在简化 CI (持续集成) 、CD (持续交付) 有着优势，作者将在后续的文章中进一步讲解 CI 、 CD 相关的内容。

## 安装 Docker

在诸多支持跨操作系统的虚拟化软件中， Docker 很可能是最为简单易用的。尽管用 Docker 之前你仍需事先安装操作系统，但之后一切变得简单，你可以从 Docker官方或者docker.cn社区直接下载可用的模版或镜像。

如果你的系统中还没有安装好 Docker ，请参考官方文档[“docker install”](https://docs.docker.com/installation/)。这里要提醒读者的是，在linux 安装Docker前请确认内核版本为3.8以上，否则将不能正常使用 Docker。本文将以 ubuntu 14.04 为例简单讲解Docker安装

Docker 官方为 ubuntu 14.04准备了标准的安装脚本，只要简单的下载并执行就可以了。下面给出标准的命令行

```
$ curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

在完成 Docker 安装后我们可以通过在 shell 键入 sudo docker 命令查看(注意 Docker 运行必须要root权限):

```
$ sudo docker

Usage: docker [OPTIONS] COMMAND [arg...]
 -H=[unix:///var/run/docker.sock]: tcp://host:port to bind/connect to or unix://path/to/socket to use

A self-sufficient runtime for linux containers.

Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from a container's filesystem to the host path
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    export    Stream the contents of a container as a tar archive
    history   Show the history of an image
    images    List images
    import    Create a new filesystem image from the contents of a tarball
    info      Display system-wide information
    inspect   Return low-level information on a container
    kill      Kill a running container
    load      Load an image from a tar archive
    login     Register or log in to a Docker registry server
    logout    Log out from a Docker registry server
    logs      Fetch the logs of a container
    port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
    pause     Pause all processes within a container
    ps        List containers
    pull      Pull an image or a repository from a Docker registry server
    push      Push an image or a repository to a Docker registry server
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image to a tar archive
    search    Search for an image on the Docker Hub
    start     Start a stopped container
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Lookup the running processes of a container
    unpause   Unpause a paused container
    version   Show the Docker version information
    wait      Block until a container stops, then print its exit code
```

也可以查看安装的 Docker 版本

```
$sudo docker version

Client version: 1.2.0
Client API version: 1.14
Go version (client): go1.3.1
Git commit (client): fa7b24f
OS/Arch (client): linux/amd64
Server version: 1.2.0
Server API version: 1.14
Go version (server): go1.3.1
Git commit (server): fa7b24f
```

## Docker 入门

Docker 提供了一系列命令来管理 Docker 镜像。我们首先来看本地 Docker 库中是否已有一些镜像。

```
$sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE

```

使用 docker images 命令列出可用来创建 Docker 容器的本地镜像。上面命令的输出显示本地还没有任何镜像，因此我们需要先下载一镜像， Docker 镜像可以从官方下载，也可以通过第三方库下载(国内的网络情况还是推荐到[docker.cn](http://docker.cn)等国内镜像库下载)。下面将演示从官方和 docker.cn 下载 ubuntu 14.04.1

<kbd>这里要说明的是 ubuntu 14.04.1 是我们产生其他镜像的一个基础镜像 (base image) Docker 提供了大量的基础镜像下载，当然如果你对 Docker 提供的基础镜像不满意，以可以尝试自己创建基础镜像</kbd>

从官方下载

```
$sudo docker pull ubuntu:14.04.1
Pulling repository ubuntu
9cbaf023786c: Download complete
511136ea3c5a: Download complete
97fd97495e49: Download complete
2dcbbf65536c: Download complete
6a459d727ebb: Download complete
8f321fc43180: Download complete
03db2b23cf03: Download complete
```

下载完成后可以使用 docker images 命令查看本地镜像

```
$sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04.1             9cbaf023786c        7 days ago          192.8 MB
```

从 docker.cn 下载

``````
$ sudo docker pull docker.cn/docker/ubuntu:14.04.1

Pulling repository docker.cn/docker/ubuntu
9cbaf023786c: Download complete
511136ea3c5a: Download complete
97fd97495e49: Download complete
2dcbbf65536c: Download complete
6a459d727ebb: Download complete
8f321fc43180: Download complete
03db2b23cf03: Download complete
``````

同样从 docker.cn 下载完成后可以使用 docker images 命令查看本地镜像

```
$sudo docker images

docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker.cn/docker/ubuntu   14.04.1             9cbaf023786c        7 days ago          192.8 MB

```

我们下载镜像后就可以运行这个镜像了。 Docker 通过 docker run 命令可以运行一个指定的镜像。我们可以通过两中方式制定要运行的镜像

```
指定 Image Id 来运行一个镜像
docker run [IMAGE ID]

指定 REPOSITORY/TAG 来运行一个镜像
docker run [REPOSITORY/TAG]
```

下面我们以 docker.cn 的 ubuntu 14.04.1 镜像为例演示运行 Docker 镜像。以下两条命令都可以运行 Docker 的 ubuntu 镜像

用IMAGE ID 方式指定要运行的镜像
```
$sudo docker run -t -i 9cbaf023786c /bin/bash
root@8e4a0a26ae66:/#
```

用 REPOSITORY/TAG 方式指定要运行的镜像

```
$sudo docker run -t -i docker.cn/docker/ubuntu:14.04.1 /bin/bash
root@6d236e3af3a3:/#
```

在 docker run 命令中我们使用了两个参数，-t 选项让 Docker 分配一个伪终端 (pseudo-tty) 并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开。在指定镜像后输入了命令 /bin/bash 启动 bash 开始 shell。

> **Docker 镜像 (image) 和 容器 (container) 的关系:**

> - 镜像 (image) 是 Docker 的模板
> - 容器 (container) 是 运行一个指定镜像的实例
> - Docker 实际上是通过 Linux LXC ，将镜像装入容器运行。同时一个镜像作为模板可以产生多个实例，也就是在多个容器中互不干扰的运行同一个镜像的实例。

## 管理 Docker 的镜像和容器

> **Docker 提供了三种方式来运行镜像，下面分别来讲解这三种方式:**

> - 执行操作后立即退出
> - 使用交互方式运行(shell)
> - 运行在独立 (守护) 模式(Detached mode or Daemon mode)

### 执行操作后立即退出

我们执行 docker run 命令，来显示 Hello world

```
$sudo docker run docker.cn/docker/ubuntu:14.04.1 /bin/echo 'Hello world'
Hello world
```

上面的命令是指定使用 REPOSITORY 为 docker.cn/docker/ubuntu 、 TAG 为 14.04.1 的镜像，来运行一个容器，并且在容器容器启动后立即执行 /bin/echo 'Hello world' 命令，执行完成后这个容器就退出了，但是这个容器并没有被删除，依然存在。可以使用 docker ps 命令查看容器进程信息。

```
$ sudo docker ps -l
CONTAINER ID        IMAGE                             COMMAND                CREATED             STATUS                      PORTS               NAMES
7f07bb50676a        docker.cn/docker/ubuntu:14.04.1   "/bin/echo 'Hello wo   13 minutes ago      Exited (0) 13 minutes ago                       distracted_mayer
```

sudo docker ps <kbd>-l</kbd> 列出最后一个容器的信息

|CONTAINER ID       | IMAGE                            | COMMAND                |CREATED             |STATUS                      |PORTS               |NAMES
|-------------------|----------------------------------|------------------------|--------------------|----------------------------|--------------------|-----------------
|7f07bb50676a       | docker.cn/docker/ubuntu:14.04.1  | "/bin/echo 'Hello wo   |13 minutes ago      |Exited (0) 13 minutes ago   |                    |distracted_mayer

下表中给出各部分所代表的含义：

<kbd>CONTAINER ID</kbd> <kbd>7f07bb50676a</kbd> CONTAINER ID 是我们管理容器的重要标志，我们可以通过 CONTAINER ID 来停止、删除、重新启动一个容器

<kbd>IMAGE</kbd> <kbd>docker.cn/docker/ubuntu:14.04.1</kbd>启动镜像标示。

<kbd>COMMAND</kbd> <kbd>"/bin/echo 'Hello wo</kbd> 容器运行的命令

<kbd>CREATED</kbd> <kbd>13 minutes ago</kbd> 容器创建的时间

<kbd>STATUS</kbd> <kbd>Exited (0) 13 minutes ago</kbd> 容器运行状态和时间

<kbd>PORTS</kbd> <kbd> 空 </kbd> 容器对外暴露的端口

<kbd>NAMES</kbd> <kbd>distracted_mayer</kbd> 容器使用的名字

从上面的 STATUS 列，可以看出刚才显示 "Hello World" 的容器已经退出。

### 使用交互方式运行

Docker 允许我们在启动一个容器后保持标准输入一直打开 ( <kbd>-i</kbd> 参数 )，并通过伪终端 (<kbd>-t</kbd> 参数) 来进行交互，

```
$sudo docker run -t -i docker.cn/docker/ubuntu:14.04.1 /bin/bash
root@32d83047546f:/#
```
上面的命令启动了 docker.cn/docker/ubuntu:14.04.1 镜像的容器实例，并且运行了bash，因为容器一直保持标准输入打开( <kbd>-i</kbd> 参数 )，并提供了伪终端(<kbd>-t</kbd> 参数)，所以我们就可以通过 bash 操作容器了


我们可以简单的在容器的 bash 中输入 <kbd>ll</kbd> 来显示 容器内当前目录的文件信息
```
root@32d83047546f:/# ll
total 72
drwxr-xr-x  47 root root 4096 Oct 22 06:20 ./
drwxr-xr-x  47 root root 4096 Oct 22 06:20 ../
-rwxr-xr-x   1 root root    0 Oct 22 06:20 .dockerenv*
-rwxr-xr-x   1 root root    0 Oct 22 06:20 .dockerinit*
drwxr-xr-x   2 root root 4096 Oct 13 03:38 bin/
drwxr-xr-x   2 root root 4096 Apr 10  2014 boot/
drwxr-xr-x   4 root root  360 Oct 22 06:20 dev/
drwxr-xr-x  64 root root 4096 Oct 22 06:20 etc/
drwxr-xr-x   2 root root 4096 Apr 10  2014 home/
drwxr-xr-x  12 root root 4096 Oct 13 03:37 lib/
drwxr-xr-x   2 root root 4096 Oct 13 03:36 lib64/
drwxr-xr-x   2 root root 4096 Oct 13 03:34 media/
drwxr-xr-x   2 root root 4096 Apr 10  2014 mnt/
drwxr-xr-x   2 root root 4096 Oct 13 03:34 opt/
dr-xr-xr-x 275 root root    0 Oct 22 06:20 proc/
drwx------   2 root root 4096 Oct 13 03:38 root/
drwxr-xr-x   7 root root 4096 Oct 13 03:37 run/
drwxr-xr-x   2 root root 4096 Oct 13 21:19 sbin/
drwxr-xr-x   2 root root 4096 Oct 13 03:34 srv/
dr-xr-xr-x  13 root root    0 Oct 22 06:20 sys/
drwxrwxrwt   2 root root 4096 Oct 13 21:20 tmp/
drwxr-xr-x  11 root root 4096 Oct 13 03:34 usr/
drwxr-xr-x  15 root root 4096 Oct 13 03:38 var/
root@32d83047546f:/#

```

下面我们在刚刚启动的 ubuntu 容器中进行一下系统更新

```
root@32d83047546f:/# apt-get update
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
Ign http://archive.ubuntu.com trusty-security InRelease
Ign http://archive.ubuntu.com trusty-proposed InRelease
Get:1 http://archive.ubuntu.com trusty Release.gpg [933 B]
Get:2 http://archive.ubuntu.com trusty-updates Release.gpg [933 B]
Get:3 http://archive.ubuntu.com trusty-security Release.gpg [933 B]
Get:4 http://archive.ubuntu.com trusty-proposed Release.gpg [933 B]
Get:5 http://archive.ubuntu.com trusty Release [58.5 kB]
Get:6 http://archive.ubuntu.com trusty-updates Release [59.7 kB]
Get:7 http://archive.ubuntu.com trusty-security Release [59.7 kB]

...

root@32d83047546f:/# apt-get upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  libssl1.0.0 tzdata
2 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 1003 kB of archives.
After this operation, 35.8 kB disk space will be freed.
...

root@32d83047546f:/#
```

现在已经更新了容器内的系统，下一步我们来保存这个更新的结果，也就是将更新后的容器保存为一个新的镜像。首先我们先退出这个容器：

```
root@32d83047546f:/# exit
exit
root@dockerdev:~#

```

我们可以使用 <kbd>exit</kbd> 命令，或者快捷键 <kbd>Ctrl+d</kbd> 来退出 shell，退出 shell 后容器因为没有要执行的操作了，也就退出了。现在我们通过 docker ps -l 命令来查看最后一个运行的容器，也就是我们刚刚在更新后退出的 ubuntu 容器。


```
$sudo docker ps -l

CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS                      PORTS               NAMES
09b218b78ea3        docker.cn/docker/ubuntu:14.04.1   "/bin/bash"         15 minutes ago      Exited (0) 15 minutes ago                       sick_einstein

```

|CONTAINER ID        | IMAGE                             | COMMAND             | CREATED             | STATUS                      | PORTS               | NAMES
|--------------------| ----------------------------------| --------------------| ---------------------------------------------------------------------------------
|09b218b78ea3        | docker.cn/docker/ubuntu:14.04.1   | "/bin/bash"         | 15 minutes ago      | Exited (0) 15 minutes ago   |                     | sick_einstein

这就是我们刚刚更新后退出的容器信息，我们接下来使用 dokcer commit 命令将这个容器设置为一个行的镜像

```
$sudo docker commit -m "ubuntu update & upgrade" -a "DockerDev" 09b218b78ea3 dockerdev/ubuntu:upbase

0e109fb78933beba666ca2dff991e1913475549a2cb498deeed4b84710ab7919


$sudo docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
dockerdev/ubuntu          upbase              0e109fb78933        2 minutes ago       192.8 MB
docker.cn/docker/ubuntu   14.04.1             9cbaf023786c        8 days ago          192.8 MB

```

> docker commit 命令 我们使用了两个参数:
> - <kbd>-m "ubuntu update & upgrade"</kbd> 填写一些要提交为镜像的说明信息
> - <kbd>-a "DockerDev"</kbd> 提交人的信息

<kbd>09b218b78ea3</kbd> 是我们通过 docker ps -l 命令查询到的容器 ID

<kbd>dockerdev/ubuntu:upbase</kbd> 是我们给要新提交创建镜像的 REPOSITORY:TAG

执行 docker images 命令我们可以看到，我们已经通过 docker commit 命令创建了一个新的镜像

新产生镜像的基本信息如下

<kbd>REPOSITORY:dockerdev/ubuntu</kbd>

<kbd>TAG:upbase</kbd>

<kbd>IMAGE ID:0e109fb78933</kbd>


### 运行在独立 (守护) 模式(Detached mode or Daemon mode)
下面，我们来让Docker以独立 (守护) 模式 (Detached mode or Daemon mode) 来运行容器

```
$sudo docker run -d dockerdev/ubuntu:upbase /bin/sh -c "while true; do echo hello world; sleep 1; done"

56919c8c390e7cc9d60cb2405348dc07ed7f99ad555d2c1861ff22f8763685f7

$sudo docker ps
CONTAINER ID        IMAGE                     COMMAND                CREATED             STATUS              PORTS               NAMES
56919c8c390e        dockerdev/ubuntu:upbase   "/bin/sh -c 'while t   16 seconds ago      Up 15 seconds                           silly_lumiere

```

上面我们使用的 <kbd>-d</kbd> 参数来指定将刚刚创建的镜像 dockerdev/ubuntu:upbase 以独立 (守护) 模式运行，只用我们通过让这个运行的容器每隔1秒就输出一个 "hello world"
可以通过docker logs 命令来查看一个指定容器的日志输出 在这里，我们将能看到输出很多行 "hello world"

```
$sudo docker logs 56919c8c390e

hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world

```
<kbd>56919c8c390e</kbd> 是我们以独立 (守护) 模式运行的容器ID

接下来我们可以停止这个以独立 (守护) 模式运行的容器，毕竟一直输出 "hello world" 没有什么实际意义,我们可以通过 docker stop 命令来停止一个正在运行的容器。

```
$sudo docker stop 56919c8c390e

56919c8c390e

$sudo docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$sudo docker ps -l

CONTAINER ID        IMAGE                     COMMAND                CREATED             STATUS                      PORTS               NAMES
56919c8c390e        dockerdev/ubuntu:upbase   "/bin/sh -c 'while t   6 minutes ago       Exited (-1) 5 minutes ago                       silly_lumiere
```

在 stop ID 为<kbd>56919c8c390e</kbd>的容器后，我们使用 docker ps 命令查看容器进程已经为空，说明容器已经停止。我们也可以使用 docker ps -l 命令显示最后一个运行的容器，我们可以看到我们停止的容器现在的状态为<kbd>Exited (-1)</kbd>


## 使用Docker构建Java+Tomcat+Redis开发环境

刚才我们已经构建了一个已经更新完毕的ubunt 14.04.1 为基础的镜像，下面我们将以此镜像为基础构建Java+Tomcat+Redis开发环境。

我们将以dockerdev/ubuntu:upbase镜像为基础，通过 Dockerfile 构建 openjdk 和 redis 镜像，再以 openjdk 为基础构建 tomcat 镜像。

### Dockerfile 是什么

Dockerfile 类似 shell 脚本，我们可以将构建镜像的一系列命令写进 Dockerfile ，批量执行并产生新的镜像

在前面的内容中，我们通过交互方式更新了 ubuntu ，并且通过 docker commit 保存为一个新的镜像，但是这并不是Docker推荐的镜像构建方式。在接下来的内容中，我们将学习如何通过编写Dockerfile脚本来构建和维护镜像。

首先我们要准备一下目录结构和文件

```
dockerdev
├── dockerdev.openjdk
│   └── Dockerfile
├── dockerdev.redis
│   └── Dockerfile
└── dockerdev.tomcat
    └── Dockerfile

```

我们以 dockerdev.openjdk 的 Dockerfile 为例来讲解 Dockerfile 基础

我们先看一下用来创建 openjdk 的 Dockerfile 内容，然后我们对其中指令进行讲解：

```
#
# OpenJDK Java 7 JDK Dockerfile
#
# docker.cn
#

# Pull base image.
FROM dockerdev/ubuntu:upbase

MAINTAINER fivestarsky

# Install Java.
RUN \
  apt-get update && \
  apt-get install -y openjdk-7-jdk && \
  rm -rf /var/lib/apt/lists/*

# Define working directory.
WORKDIR /

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

# Define default command.
CMD ["bash"]

```

#### FROM指令和MAINTAINER指令

通过FROM指令，docker编译程序能够知道在哪个基础镜像执行来进行编译。所有的Dockerfile都必须以FROM指令开始。 Docker 会首先查找本地镜像库是否有指定的镜像，如果没有将尝试下载指定的镜像。

MAINTAINER，用来标明这个镜像的维护者信息。

#### RUN指令

Run指令用来在 Docker 的编译环境中运行指定命令。上面这条指令会在编译环境运行/bin/sh -c "apt-get update && apt-get -y install ..."。
每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

在使用Run指令的时候需要注意每一个 Run 指令都会让Docker 产生一个新层(layer),而 Docker 规定一个镜像最多有127层。所以我们应该尽量将多个操作合并为一个RUN 指令

RUN指令还有另外一种格式：
RUN ["程序名", "参数1", "参数2"]
这种格式运行程序，可以免除运行/bin/sh的消耗。这种格式使用Json格式将程序名与所需参数组成一个字符串数组，所以如果参数中有引号等特殊字符，需要进行转义。

#### WORKDIR指令
WORKDIR 指令用于设置执行 RUN 指令、 CMD 指令和 ENTRYPOINT 指令执行时的工作目录。在 Dockerfile 中可以多次设置 WORKDIR ，在每次设置之后的命令将使用新的路径。

#### ENV指令
ENV指令用来指定在执行 docker run 命令运行镜像时，自动设置的环境变量。这些环境变量可以通过 docker run 命令的 --evn 参数来进行修改。

<kbd>注意：ENV 指令指定的环境变量在运行创建的镜像时依然有效</kbd>

#### CMD 指令和 ENTRYPOINT 指令

CMD指令通常是整个 Dockerfile 脚本的最后一条指令。当 Dockerfile 已经完成了所有环境的安装与配置，通过 CMD 指令来指示 docker run 命令运行镜像时要执行的命令。上面的例子里，在完成所有工作后，Dockerfile编译脚本将启动 bash。

ENTRYPOINT指令和前面介绍过的CMD一样，用于标明一个镜像作为容器运行时，最后要执行的程序或命令。

这两个指令有相同只处，也有区别。通过两个指令的配合使用可以配置出不同的效果。

ENTRYPOINT指令有两种格式，CMD指令有三种格式：

```
ENTRYPOINT ["程序名", "参数1", "参数2"]
ENTRYPOINT 命令 参数1 参数2

CMD ["程序名", "参数1", "参数2"]
CMD 命令 参数1 参数2
CMD 参数1 参数2
```

ENTRYPOINT 是容器运行程序的入口。也就是说，在 docker run 命令中指定的命令都将作为参数提供给 ENTRYPOINT 指定的程序。
同样，上面列举的 CMD 指令格式的后面两种格式也将作为参数提供给 ENTRYPOINT 指定的程序。

默认的 ENTRYPOINT 是/ bin/sh -c 。你可以根据实际需要任意设置。但是如果在一个 Dockerfile 中出现了多个 ENTRYPOINT 指令，那么，只有最后一个 ENTRYPOINT 指令是起效的。

一种常用的设置是将命令与必要参数设置到 ENTRYPOINT 中，而运行时只提供其他选项。例如：你有一个 MySQL 的客户端程序运行在容器中，而客户端所需要的主机地址、用户名和密码你不希望每次都输入，你就可以将ENTRYPOINT设置成：ENTRYPOINT mysql -u <用户名> -p <密码> -h <主机名>。而运行这个镜像时，只需要指定数据库名。

<kbd>注意：CMD 和 ENTRYPOINT 是在每次运行镜像的时候都执行的</kbd>


### 编写 需要的 Dockerfile

下面统一给出我们需要的所有 Dockerfile:

openjdk Dockerfile 路径：

```
dockerdev
└── dockerdev.openjdk
    └── Dockerfile
```

openjdk Dockerfile 内容：

```
#
# OpenJDK Java 7 JDK Dockerfile
#
# docker.cn
#

# Pull base image.
FROM dockerdev/ubuntu:upbase

MAINTAINER fivestarsky

# Install Java.
RUN \
  apt-get update && \
  apt-get install -y openjdk-7-jdk && \
  rm -rf /var/lib/apt/lists/*

# Define working directory.
WORKDIR /

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

# Define default command.
CMD ["bash"]

```

redis Dockerfile 路径：

```
dockerdev
└── dockerdev.redis
    └── Dockerfile
```

redis Dockerfile 内容：

```
#
# Redis Dockerfile
#
# docker.cn
#

# Pull base image.
FROM dockerdev/ubuntu:upbase

MAINTAINER fivestarsky

# Install Redis.
RUN \
  cd /tmp && \
  apt-get update && \
  apt-get install -y wget g++ make && \
  wget http://download.redis.io/redis-stable.tar.gz && \
  tar xvzf redis-stable.tar.gz && \
  cd redis-stable && \
  make && \
  make install && \
  cp -f src/redis-sentinel /usr/local/bin && \
  mkdir -p /etc/redis && \
  cp -f *.conf /etc/redis && \
  rm -rf /tmp/redis-stable* && \
  sed -i 's/^\(bind .*\)$/# \1/' /etc/redis/redis.conf && \
  sed -i 's/^\(daemonize .*\)$/# \1/' /etc/redis/redis.conf && \
  sed -i 's/^\(dir .*\)$/# \1\ndir \/data/' /etc/redis/redis.conf && \
  sed -i 's/^\(logfile .*\)$/# \1/' /etc/redis/redis.conf

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

# Define default command.
CMD ["redis-server", "/etc/redis/redis.conf"]

# Expose ports.
EXPOSE 6379

```

redis Dockerfile 出现了一些新的指令我们进行简单解释：

### VOLUME 指令

格式为 VOLUME ["/data"]。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

### EXPOSE 指令

格式为 EXPOSE <port> [<port>...]。

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。


tomcat Dockerfile 路径：

```
dockerdev
└── dockerdev.tomcat
    └── Dockerfile
```

tomcat Dockerfile 内容：

```
#
# Tomcat7 Dockerfile
#
# docker.cn
#

FROM openjdk

MAINTAINER fivestarsky

RUN apt-get update && \
    apt-get install -yq --no-install-recommends wget pwgen ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV TOMCAT_MAJOR_VERSION 7
ENV TOMCAT_MINOR_VERSION 7.0.56
ENV CATALINA_HOME /tomcat

# INSTALL TOMCAT
RUN wget -q https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz && \
    wget -qO- https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz.md5 | md5sum -c - && \
    tar zxf apache-tomcat-*.tar.gz && \
    rm apache-tomcat-*.tar.gz && \
    mv apache-tomcat* tomcat

EXPOSE 8080

CMD /tomcat/bin/catalina.sh run

```

在编写完这三个Dockerfile后 我们就可以将 Dockerfile 编译为镜像了，我们将逐一编译三个 Dockerfile 并讲解编译的依赖关系

编译 dockerdev.openjdk/Dockerfile

进入 dockerdev/dockerdev.openjdk 目录，并执行命令

```
dockerdev/dockerdev.openjdk# sudo docker build -t openjdk .

Sending build context to Docker daemon  2.56 kB
Sending build context to Docker daemon
Step 0 : FROM dockerdev/ubuntu:upbase
 ---> 0e109fb78933
Step 1 : MAINTAINER fivestarsky
 ---> Running in 1cf07d9a953e
 ---> 7b110000a941
Removing intermediate container 1cf07d9a953e
Step 2 : RUN   apt-get update &&   apt-get install -y openjdk-7-jdk &&   rm -rf /var/lib/apt/lists/*
 ---> Running in 41ca78e318c5
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
Ign http://archive.ubuntu.com trusty-security InRelease
Ign http://archive.ubuntu.com trusty-proposed InRelease

...

done.
done.
Processing triggers for sgml-base (1.26+nmu4ubuntu1) ...
 ---> 53a0764665b1
Removing intermediate container 41ca78e318c5
Step 3 : WORKDIR /
 ---> Running in a3366ac9972c
 ---> c3d1332cb5c6
Removing intermediate container a3366ac9972c
Step 4 : ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
 ---> Running in 48fe1533fcd4
 ---> 4474ea8b1ab8
Removing intermediate container 48fe1533fcd4
Step 5 : CMD ["bash"]
 ---> Running in 584cb3f262de
 ---> 32269ea6368f
Removing intermediate container 584cb3f262de
Successfully built 32269ea6368f

```
编译完成后我们可以看到 Successfully built 提示。接下来我们可以执行 docker images 命令查看编译后产生的镜像

```
$sudo docker images

REPOSITORY                TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
openjdk                   latest              32269ea6368f        About a minute ago   578.6 MB
dockerdev/ubuntu          upbase              0e109fb78933        3 hours ago          192.8 MB
docker.cn/docker/ubuntu   14.04.1             9cbaf023786c        8 days ago           192.8 MB

```
<kbd>这里需要注意：我们在执行编译时 加入的 '-t openjdk' 参数 是为编译产生的镜像指定一个 REPOSITORY ，如果不指定 TAG 则默认为 latest, 指定 TAG 可以表示为: '-t openjdk:tagname' </kbd>

接下来我们编译 tomcat

编译 dockerdev.tomcat/Dockerfile

进入 dockerdev/dockerdev.tomcat 目录，并执行命令

```
dockerdev/dockerdev.tomcat# sodu docker build -t opentomcat:latest .

Sending build context to Docker daemon 3.072 kB
Sending build context to Docker daemon
Step 0 : FROM openjdk
 ---> 32269ea6368f
Step 1 : MAINTAINER fivestarsky
 ---> Running in 796db53b96d7
 ---> 1b68febc5c4f
Removing intermediate container 796db53b96d7
Step 2 : RUN apt-get update &&     apt-get install -yq --no-install-recommends wget pwgen ca-certificates &&     apt-get clean &&     rm -rf /var/lib/apt/lists/*
 ---> Running in 65883eb7ff36

 ...

Removing intermediate container 65883eb7ff36
Step 3 : ENV TOMCAT_MAJOR_VERSION 7
 ---> Running in 793ce0acb114
 ---> a48458c0363f
Removing intermediate container 793ce0acb114
Step 4 : ENV TOMCAT_MINOR_VERSION 7.0.56
 ---> Running in 2dd8a1bf81d1
 ---> 602f00a98040
Removing intermediate container 2dd8a1bf81d1
Step 5 : ENV CATALINA_HOME /tomcat
 ---> Running in 641c4c1d402e
 ---> c796492000b2
Removing intermediate container 641c4c1d402e
Step 6 : RUN wget -q https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz &&     wget -qO- https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz.md5 | md5sum -c - &&     tar zxf apache-tomcat-*.tar.gz &&     rm apache-tomcat-*.tar.gz &&     mv apache-tomcat* tomcat
 ---> Running in 370717c866c3
apache-tomcat-7.0.56.tar.gz: OK
 ---> cce75f69363b
Removing intermediate container 370717c866c3
Step 7 : EXPOSE 8080
 ---> Running in a324157603cf
 ---> a824f34d2dd9
Removing intermediate container a324157603cf
Step 8 : CMD /tomcat/bin/catalina.sh run
 ---> Running in 2446926e243a
 ---> a2246837501f
Removing intermediate container 2446926e243a
Successfully built a2246837501f

```

编译完成后我们可以看到 Successfully built 提示。接下来我们可以执行 docker images 命令查看编译后产生的镜像

```
$sudo docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
opentomcat                latest              a2246837501f        13 minutes ago      595.9 MB
openjdk                   latest              32269ea6368f        24 minutes ago      578.6 MB
dockerdev/ubuntu          upbase              0e109fb78933        3 hours ago         192.8 MB
docker.cn/docker/ubuntu   14.04.1             9cbaf023786c        8 days ago          192.8 MB

```

<kbd>这里需要注意： tomcat 的 Dockerfile 我们在开头指定的 FROM openjdk 依赖于前面编译的 openjdk:latest 镜像，如果我们前面没有正常产生openjdk:latest 镜像 Docker 会尝试到网络仓库去下载，如果也没有找到将会报错！ 也就是说 tomcat 镜像依赖于 openjdk 镜像才能进行编译</kbd>

接下来我们编译最后一个 redis Dockerfile

进入 dockerdev/dockerdev.redis 目录，并执行命令

```
dockerdev/dockerdev.redis#sudo docker build -t openredis:latest .

Sending build context to Docker daemon 3.072 kB
Sending build context to Docker daemon
Step 0 : FROM dockerdev/ubuntu:upbase
 ---> 0e109fb78933
Step 1 : MAINTAINER fivestarsky
 ---> Using cache
 ---> 7b110000a941
Step 2 : RUN   cd /tmp &&   apt-get update &&   apt-get install -y wget &&   apt-get install -y g++ make &&   wget http://download.redis.io/redis-stable.tar.gz &&   tar xvzf redis-stable.tar.gz &&   cd redis-stable &&   make &&   make install &&   cp -f src/redis-sentinel /usr/local/bin &&   mkdir -p /etc/redis &&   cp -f *.conf /etc/redis &&   rm -rf /tmp/redis-stable* &&   sed -i 's/^\(bind .*\)$/# \1/' /etc/redis/redis.conf &&   sed -i 's/^\(daemonize .*\)$/# \1/' /etc/redis/redis.conf &&   sed -i 's/^\(dir .*\)$/# \1\ndir \/data/' /etc/redis/redis.conf &&   sed -i 's/^\(logfile .*\)$/# \1/' /etc/redis/redis.conf
 ---> Running in e3f9ece34e10
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
Ign http://archive.ubuntu.com trusty-security InRelease
Ign http://archive.ubuntu.com trusty-proposed InRelease
Get:1 http://archive.ubuntu.com trusty Release.gpg [933 B]
Get:2 http://archive.ubuntu.com trusty-updates Release.gpg [933 B]
Get:3 http://archive.ubuntu.com trusty-security Release.gpg [933 B]
Get:4 http://archive.ubuntu.com trusty-proposed Release.gpg [933 B]
Get:5 http://archive.ubuntu.com trusty Release [58.5 kB]
Get:6 http://archive.ubuntu.com trusty-updates Release [59.7 kB]
Get:7 http://archive.ubuntu.com trusty-security Release [59.7 kB]
Get:8 http://archive.ubuntu.com trusty-proposed Release [110 kB]

...

Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory `/tmp/redis-stable/src'
cd src && make install
make[1]: Entering directory `/tmp/redis-stable/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/tmp/redis-stable/src'
 ---> edf2f52bf379
Removing intermediate container e3f9ece34e10
Step 3 : VOLUME ["/data"]
 ---> Running in e4829536357c
 ---> 7d05ddb6c52d
Removing intermediate container e4829536357c
Step 4 : WORKDIR /data
 ---> Running in 7d9b1d23f0d4
 ---> 2f456b8b46e3
Removing intermediate container 7d9b1d23f0d4
Step 5 : CMD ["redis-server", "/etc/redis/redis.conf"]
 ---> Running in 5428d46542a8
 ---> a6d5744ad999
Removing intermediate container 5428d46542a8
Step 6 : EXPOSE 6379
 ---> Running in 6b26b08ff26f
 ---> 0a41117a14ef
Removing intermediate container 6b26b08ff26f
Successfully built 0a41117a14ef

```

编译完成后我们可以看到 Successfully built 提示。接下来我们可以执行 docker images 命令查看编译后产生的镜像

```
$sudo docker images
REPOSITORY                TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
openredis                 latest              0a41117a14ef        About a minute ago   331.3 MB
opentomcat                latest              a2246837501f        34 minutes ago       595.9 MB
openjdk                   latest              32269ea6368f        46 minutes ago       578.6 MB
dockerdev/ubuntu          upbase              0e109fb78933        4 hours ago          192.8 MB
docker.cn/docker/ubuntu   14.04.1             9cbaf023786c        8 days ago           192.8 MB
```

至此我们将 java + tomcat + redis 开发环境所需的镜像全部编译完成

## 将 java 开发的 war 包部署到 Docker 的 tomcat 容器进行测试

> 首先，我们来讲解一下我们如何配置并运行容器：我们将使用 opentomcat 和 openredis 镜像运行两个独立 (守护) 模式的容器。在 java 的 war 包中我们将参数 put 到 redis 进行存储并遍历显示。相关war包和java代码请从底部链接下载

> - opentomcat 容器挂载 host 宿主机的指定目录作为 webapps 目录，方便从宿主机直接部署 war 包

> - opentomcat 容器对外开放 8080 端口 提供 web 访问

> - opentomcat 通过容器间 link 方式访问 openredis

在 dockerdev/dockerdev.tomcat/ 目录下建立 webapps 子目录，并且将下载的 DemoWebDev.war 文件复制到 webapps 子目录

```
$sudo docker run -d --name redis_server openredis

0827f4ba71bf776f1a9127e5880c49b3d2e55760c7e2ce0b604aff59752ad030



$sudo docker run -d -p 8080:8080 --link redis_server:redis_server -v /root/dockerdev/dockerdev.tomcat/webapps:/tomcat/webapps opentomcat

4d1bf5d6621424da0f9be45193b42221bd1c09ac0b906ab930de8ecd44c6f3de

```

> 下面我们来解释相关参数含义

> - docker run -d <kbd>--name redis_server</kbd> openredis：为运行的容器设置一个名字

> - docker run -d -p 8080:8080 <kbd>--link redis_server:redis_server </kbd> ... opentomcat: 要链接的容器名字(对应<kbd>--name redis_server</kbd>):在 opentomcat 容器内部链接容器的名字

> - docker run -d <kbd>-p 8080:8080</kbd> --link redis_server ... opentomcat: opentomcat容器对外开放8080端口，8080<kbd>映射宿主机端口</kbd>:8080<kbd>容器内部端口</kbd>

> - docker run -d ... <kbd>-v /root/dockerdev/dockerdev.tomcat/webapps:/tomcat/webapps</kbd> opentomcat : 挂载宿主机/root/dockerdev/dockerdev.tomcat/webapps到容器的/tomcat/webapps。也就是我们可以在宿主机部署 war 包，可以被容器立刻发现并使用

在启动两个容器后我们可以查看 dockerdev/dockerdev.tomcat/webapps 目录 会发现 DemoWebDev.war 已经解压为 DemoWebDev 目录，这就是因为我们将宿主机的一个卷 (/root/dockerdev/dockerdev.tomcat/webapps) 挂载到了容器里(/tomcat/webapps),容器可以实时发现挂载卷的内容改变。
接下来我们可以测试一下我们的 DemoWebDev 在浏览器输入： http://host:8080/DemoWebDev/TestRedisPush?pushvalue=demo, host 为宿主机的ip地址、pushvalue 是我们要存储到 redis 的数据

```
push:demo OK!
now redis list values: [demo]

```

## 访问独立 (守护) 模式运行容器的shell

有的时候我们需要进入运行在独立 (守护) 模式下的容器，并通过shell查看容器内部信心， Docker 提供了多种方式进入容器，我们在这里介绍比较常用的一种 nsenter

> 提示：
> - util-linux 是一个开放源码的软件包，是一个对任何Linux系统的基本工具套件。含有一些标准 UNIX 工具，如 login。
> - util-linux 软件包包含许多工具。其中比较重要的是加载、卸载、格式化、分区和管理硬盘驱动器，打开 tty 端口和得到内核消息。

首先我们需要安装 nsenter 来进入 Docker 容器，nsenter 在 util-linux版本2.23开始提供很不幸，Ubuntu 14.04 仍然使用的是 util-linux 版本 2.20 ，所以我们需要安装新版的 util-linux。
从 util-linux 版本 2.23 开始，nsenter工具就包含在其中。它用来访问另一个进程的名字空间。 nsenter 要正常工作需要有 root 权限。安装比较新版本的 util-linux（2.24) 版，请按照以下步骤：

```
wget https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz; tar xzvf util-linux-2.24.tar.gz
cd util-linux-2.24
./configure --without-ncurses && make nsenter
sudo cp nsenter /usr/local/bin
```

安装完成后可以通过 nsenter -v 来查看安装是否成功以及 nsenter 版本：

```
$sudo nsenter -V
nsenter from util-linux 2.24

```

我们可以通过 docker ps 命令查看正在运行的容器，并得到容器 ID,然后查询容器 ID 所在的进程 ID，最后使用 nsenter 通过进程ID 进入 docker 容器。

```
为了连接到容器，我们需要找到容器的第一个进程的 PID，可以通过下面的命令获取。
PID=$(docker inspect --format "{{ .State.Pid }}" <container id>)

通过这个 PID，就可以连接到这个容器：
$ nsenter --target $PID --mount --uts --ipc --net --pid
```

这里我们以进入redis 容器为例：

```
$sudo docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                    NAMES
4d1bf5d66214        opentomcat:latest   "/bin/sh -c '/tomcat   27 minutes ago      Up 27 minutes       0.0.0.0:8080->8080/tcp   goofy_einstein
0827f4ba71bf        openredis:latest    "redis-server /etc/r   28 minutes ago      Up 28 minutes       6379/tcp                 goofy_einstein/redis_server,redis_server

$sudo PID=$(docker inspect --format "{{ .State.Pid }}" 0827f4ba71bf)

$sudo nsenter --target $PID --mount --uts --ipc --net --pid
groups: cannot find name for group ID 122
groups: cannot find name for group ID 123
root@0827f4ba71bf:/#

```

我们应该能看出来，nsenter 的使用步骤还是比较麻烦的，实际上已经有人替我们封装好了脚本 方便我们访问 Docker 容器，下面我们来下载这个脚本并使用这个脚本方便的访问 Docker 容器。
采用.bashrc_docker简化nsenter(作者的github：https://github.com/yeasy)。

```
wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc
```

脚本安装后我们可以使用如下格式来进入 Docker 容器

```
直接进入容器
$ sudo docker-enter <container id>

在容器内执行操作，并显示
$ sudo docker-enter <container id> ls
```

下面我们依然以进入 redis-server 容器为例演示脚本的使用

```
$sudo docker ps

CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                    NAMES
4d1bf5d66214        opentomcat:latest   "/bin/sh -c '/tomcat   40 minutes ago      Up 40 minutes       0.0.0.0:8080->8080/tcp   goofy_einstein
0827f4ba71bf        openredis:latest    "redis-server /etc/r   41 minutes ago      Up 41 minutes       6379/tcp                 goofy_einstein/redis_server,redis_server


$sudo docker-enter 0827f4ba71bf

dirname: invalid option -- 'b'
Try 'dirname --help' for more information.
root@0827f4ba71bf:~#

root@0827f4ba71bf:~#ll
total 24
drwx------  2 root root 4096 Oct 23 01:28 ./
drwxr-xr-x 69 root root 4096 Oct 23 01:28 ../
-rw-------  1 root root   21 Oct 23 01:28 .bash_history
-rw-r--r--  1 root root 3106 Feb 20  2014 .bashrc
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-r--r--  1 root root   12 Oct 23 01:28 .rediscli_history

root@0827f4ba71bf:~#redis-cli
127.0.0.1:6379> keys *
1) "TestList"
127.0.0.1:6379> exit

root@0827f4ba71bf:~# exit
logout

```

从上面的演示可以看出，我们可以进入 redis-server 容器，并且调用容器内部的 redis-cli 查询所有的 redis keys。

到此位置，我们已经简单的学习了Docker 如何搭建 Java+tomcat+redis 开发环境的搭建，对 Docker 有了初步的认识。

相关 Dockerfile 、java 源代码、war 包 下载地址：XXXXX
