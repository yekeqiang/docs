# Docker，云时代的程序交付方式

**Posted on August 11, 2014 by [liubin](http://weibo.com/pmproad)**

要说最近一年云计算业界有什么大事件？Google Compute Engine
的正式发布？ Azure 入华？还是 AWS 落地中国？留在每个人大脑中的印象可能各不相同，但要是让笔者来排名的话那么 Docker 绝对应该算是第一位的。如果你之前听说过它的话，那么也许你会说“没错，就是它”，因为几乎世界各地的开发、运维都在谈论着 Docker ；如果你还没听说过 Docker ，那么我真的建议你花上10分钟来阅读本文。

## 1. Docker 简介

### 1.1. 什么是 Docker ？

Docker 是一个重新定义了程序开发测试、交付和部署过程的开放平台。 Docker 也是容器技术的一种，它运行于 Linux 宿主机之上，每个运行的容器都是相互隔离的，也被称为轻量级虚拟技术或容器型虚拟技术。而且它有点类似 Java 的编译一次，到处运行， Docker 则可以称为构建一次，在各种平台上运行，包括本地服务器和云主机等（ Build once，run anywhere ）。

容器就是集装箱，我们的代码都被打包到集装箱里； Docker 就是搬运工，帮你把应用运输到世界各地，而且是超高速。

Docker 是开源软件，代码托管在 GitHub 上，使用 Go 语言编写。 Go 可以称得上是互联网时代专门为开发分布式、高并发系统而生的编程语言。 Docker 也可以说是 Go 语言的一个杀手级应用，而且在 Docker 生态圈里很多软件也都是使用 Go 语言编写的。

### 1.2. Docker 历史

Docker 项目始于2013年3月，由当时的 PaaS 服务提供商 dotCloud 开发， dotClound 也是 YCombinator S10 的毕业生。尽管 Docker 项目很年轻，到现在也只有15个月而已，然而它的发展势头如此之猛已经让很多人感叹不已了。

2013年10月 dotCloud 公司名字也由 dotCloud, Inc. 改为 Docker, Inc. ，集中更多的精力放到了 Docker 相关的研发上。

### 1.3. Docker 的技术基石

在进入 Docker 的世界之前，我们先来看一下 Docker 实现所依赖的一些技术。

实际上 Docker 的出现离不开很多 Linux kernel 提供的功能，甚至可以说 Docker 在技术上并没有什么特别重大的创新之处，利用的都是已经非常成熟的 Linux 技术而已，这些技术早在 Solaris 10 或 Linux Kernel 2.6 的时候就有了。可以毫不夸张的说 Docker 就是“站在了巨人的肩膀上”。

下面我们就先来了解一下 Docker 主要利用的 Linux 技术。

#### 1.3.1. 容器技术

容器（ Container ）有时候也被称为操作系统级虚拟化，以区别传统的 Hypervisor 虚拟技术。它不对硬件进行模拟，只是作为普通进程运行于宿主机的内核之上。

在容器中运行的一般都是一个简易版的 Linux 系统，有 root 用户权限、 init 系统（采用 LXC 容器的情况下）、进程 id 、用户 id 以及网络属性。

容器技术在云计算时代已经被大量使用。Google 公司的 Joe Beda 在今年5月做了一次题为 *[Containers At Scale — At Google, the Google Cloud Platform and Beyond](https://speakerdeck.com/jbeda/containers-at-scale)* 的演讲，在其中提到“ Everything at Google runs in a container ”，每周启动容器次数竟然多达20亿次。

很多 PaaS 平台都是基于容器技术实现的，比如目前最成功的 PaaS 平台 Heroku 。此外，还有比较著名的开源 PaaS 平台 Cloud Foundry 的 Warden 以及 Google的 [Lmctfy](http://github.com/google/lmctfy) (Let Me Contain That For You) 等。

#### 1.3.2. LXC

这也是在 Linux 下使用比较广泛的容器方案。基本上我们可以认为 Linux containers =  cgroups（资源控制） + namespaces（容器隔离）。

LXC 很成熟很强大，然而它却不好使用，比如它不方便在多台机器间移动，不方便创建管理，不可重复操作，也不方便共享等等，相对于开发人员来说，它只是系统管理员的玩具。 Docker 的出现很好的解决了这些问题，它将容器技术的使用成本拉低到了一个平民价格。

#### 1.3.3. namespaces

这是用来为容器提供进程隔离的技术，每个容器都有自己的命名空间，比如 pid/net/ipc/mnt/uts 等命名空间，以及为容器提供不同的 hostname 。 namespace 能保证不同的容器之间不会相互影响，每个容器都像是一个独立运行着的 OS 一样。

#### 1.3.4. cgroups

cgroups 是一个 Google 贡献的项目，它主要用来对共享资源的分配、限制、审计及管理，比如它可以为每个容器分配 CPU 、内存以及 blkio 等的使用限额等。 cgroups 使得容器能在宿主机上能友好的相处，并公平的分配资源以及杜绝资源滥用的潜在风险。

容器技术实现方案可以用下面的图进行简单说明。

![Docker如何和Linux内核打交道](http://resource.docker.cn/docker-execdriver-diagram.png)

上图中的 cgroups 、 namespaces 和 apparmor 等都是 Linux 内核提供的功能。不管是传统的 LXC 还是 Docker 的 libcontainer ，都使用了 Kernel 的这些功能来实现容器功能。

#### 1.3.5. 联合文件系统

联合文件系统是一个分层的轻量、高性能文件系统。 Docker 之所以这么吸引人，很大程度上在于其在镜像管理上所做出的创新。而联合文件系统正是构建 Docker 镜像的基础。

AUFS（ AnotherUnionFS ）是一个分层的基于 Copy On Write 技术的文件系统，支持 Union Mount ，就是将具有不同文件夹结构的镜像层进行叠加挂载，让它们看上去就像是一个文件系统那样。

### 1.4. 容器技术 VS 虚拟机技术

容器技术和 Hypervisor 技术虽然不属于同一层次的概念，但是作为具有计算能力的应用运行载体来说，它们还是有一定的共通性和竞争关系，这里作此对比完全是为了加深读者对容器技术的理解而已。

比较方面 | 容器技术 | 虚拟机技术
:------: | :------: | :--------:
占用磁盘空间 | 小，甚至几十 KB（镜像层的情况） | 非常大，上 GB
启动速度 | 快，几秒钟 | 慢，几分钟
运行形态 | 直接运行于宿主机的内核上，不同容器共享同一个 Linux 内核 | 运行于 Hypervisior 上
并发性 | 一台宿主机可以启动成千上百个容器 | 最多几十个虚拟机
性能 | 接近宿主机本地进程 | 逊于宿主机
资源利用率 |高 | 低

比如开源 PaaS 实现软件 tsuru 最初使用的是基于虚拟机的技术，创建一个应用程序需要5分钟左右的时间，而在采用 Docker 之后，已经将这个时间缩短到了10秒钟了（详见 Andrews Medina 的分享 [tsuru and docker](https://speakerdeck.com/andrewsmedina/tsuru-and-docker) ）。
  

### 1.5. 我们能用 Docker 做什么？

Docker 可以应用在各种场景下，比如公司内部开发测试使用，或者作为公有或者私有 PaaS 平台等。

现在 PaaS 平台的发展已经非常成熟了，这里我们只罗列一些在开发中使用 Docker 技术可能会给我们带来的益处。

#### 1.5.1 在开发中

##### 构建开发环境变得简单

简单包括几个方面的意思：

- 快速：只需 docker run 即可

- 共享：通过 Dockerfile 或者Registry

- 自动化：一切代码化的东西都可以自动化

- 统一：每个人的开发环境都是一模一样的

设想我们要基于 Nginx/PHP 、 MySQL 和 Redis 开发，我们可以创建 3 个 Docker 镜像保存到公司私有的 Registry 中去，每个开发人员使用的时候是需要执行 docker run redis 即可以享用自己独有的 Redis 服务了，而且这 3 个容器不管从占用磁盘空间还是运行性能来说，都比虚拟机要好很多。

#### 1.5.2. 在测试中

##### 解决环境构建问题

有时候构建测试的环境是一项费时费力的工作，而 Docker 能让这变得轻松。如果你的测试比较简单的话，甚至直接拿开发构建的镜像就可以开始了。

##### 消除环境不一致导致的问题

“在我的机器上运行的好好的，怎么到你那里就不行了？”，我想超过半数的程序员都曾经说过类似的话。如果对导致这一问题的原因进行统计的话，我想排在第一位的应该非“环境不一致”莫属了，这包括操作系统和软件的版本、环境变量、文件路径等。

使用 Docker 的话你再也不用为此烦恼了。因为你交付的东西不光是你的代码、配置文件、数据库定义，还包括你的应用程序运行的环境：OS 加上各种中间件、类库 + 你的应用程序。

#### 1.5.3. 部署和运维

##### 基于容器的部署和自动化

Docker 定义了重新打包程序的方法。

> **Docker容器 + 用户应用 = 部署单位（构件）**

Docker 可以看作是用代码编写出来的国际集装箱，它可以把任何应用及相关依赖项打包成一个轻量、可移植（Portable）、自包涵的容器。

以前部署代码都是代码级别的，有了 Docker ，则可以进行容器级别的部署。这样带来的最大的好处就是开发者本地测试、 CI 服务器测试、测试人员测试，以及生产环境运行的都可以是同一个 Docker 镜像。

##### 快速进行横向扩展

Docker 容器的启动速度很快，可以瞬间启动大量容器，所以在非常适合在业务高峰期进行横向扩展。这比传统的启动 EC2 实例或者物理机可要快多了。

##### 天生的和云计算技术相结合

当然，由于 Docker 具有很好的移植性，所以它更强大的地方还在于和云环境结合使用。

Docker 容器是可移植，或者说跨平台。将来的应用部署可能是在本地进行打包（成 Docker 镜像）然后传送到云端运行，至于是 AWS 还是 GCE 这不是问题， Docker 都能在其上运行。这样不仅能在一定程度上解决 vendor-lockin 的问题，同时也使得在不同的云服务提供商之间迁移也变得简单。尤其是未来在使用多个云（ multi-cloud ）环境的时候，这将非常便利。

笔者认为基于 IaaS + 容器技术的应用交付、部署方式将来一定会成为一种流行的方式。

##### 进行 Blue-green 部署

「Blue-green deployment」这个词最初出现在 *Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation*一书，后经 ThoughtWorks 的 [Martin Fowler](http://martinfowler.com/bliki/BlueGreenDeployment.html) 发扬光大。

![blue_green_deployment](http://resource.docker.cn/blue-green-deployment.png)

Blue-green deployment 方法其实很简单，就是保持两套一样的生产环境，而实际上只有一套环境真正的对外提供服务（图中绿色环境），而另一套环境则处于待机状态（图中蓝色）。部署的时候，我们会先上线到蓝色环境中，如果测试没有问题了，再将路由切换到新的服务上。

Blue-green 部署能带来如下好处：

- 最小化停机时间

- 快速回滚

- hot standby

而未来的开发和部署和可能就会像下面这样进行了。

① 开发人员将代码 push 到 Git 仓库；

② CI 工具通过 webhook 得到最新代码，构建 Docker 镜像并启动容器进行测试；

③ 测试通过后将镜像打标签后 push 到私有镜像 Registry；

④ CI 工具通知 CD 工具；

⑤ CD 工具通过 Mesos/Marathon 等进行基于容器的部署；

⑥ 测试没有问题后进行容器的切换（即 Blue-green 切换）。

## 2. Docker 架构解析

### 2.1. Docker 整体结构

Docker 是一个构建、发布、运行分布式应用的平台（见下图）， Docker 平台由Docker Engine（运行环境 + 打包工具）、Docker Hub（ API + 生态系统）两部分组成。

![](http://resource.docker.cn/docker-platform.png/)

从图中我们可以看到， Docker 的底层是各种 Linux OS 以及云计算基础设施，而上层则是各种应用程序和管理工具，每层之间都是通过 API 来通信的。

#### Docker 引擎

Docker 引擎是一组开源软件，位于 Docker 平台的核心位置。它提供了容器运行时以及打包、管理等工具。

Docker 引擎可以直观理解为就是在某一台机器上运行的 Docker 程序，实际上它是一个 C/S 结构的软件，有一个后台守护进程在运行，每次我们运行 docker 命令的时候实际上都是通过 RESTful Remote API 来和守护进程进行交互的，即使是在同一台机器上也是如此。

#### Docker Hub

Docker Hub 是一个云端的分布式应用服务，专注于内容、协作和工作流。 Docker Hub 除了可以托管、下载、查找 Docker 镜像之外，还提供了包括管理、团队协作、生命周期流程自动化等功能，以及对第三方工具和服务的集成。

### 2.2. Docker 镜像（image）

#### 2.2.1. Docker 镜像

Docker 镜像是 Docker 系统中的构建模块（ Build Component ），是启动一个 Docker 容器的基础。

我们可以通过一个官方提供的示意图来帮助我们来理解一下镜像的概念。

![docker image](http://resource.docker.cn/image-container.png)

Docker 镜像位于 bootfs 之上，实际上 bootfs 在系统启动后会被卸载的。 Docker镜像（ Images ）是分层的，这得益于其采用的联合文件系统，前面我们已经介绍过了。镜像是有继承（父子）关系的，每一层镜像的下面一层称为父镜像，没有父镜像的称为基础镜像（ Base Iamge ，其实叫做 Root Image 可能更确切，不过这可能容易和 rootfs 混淆）。

#### 2.2.2. 镜像仓库

我们可以将 Docker 镜像仓库理解为 Git 仓库。 Dcoker 镜像仓库分为远程和本地，本地的概念好理解，而一般来说远程仓库就是 Registry ，包括官方的或者自建的私有 Registry ；我们通过 docker pull 和 docker push 命令在本地和远程之间进行镜像传输。

Docker 镜像的命名规则和 GitHub 也很像。比如我们自己创建的仓库名称都是类似 liubin/redis 这样格式的，前面的 liubin 是用户名或 namespace ，后面是仓库名。

不过我们前面已经看到运行的 ubuntu 镜像的时候是仓库名就是 ubuntu ，而不带用户名前缀，这是表明它是由官方制作的，或者由官方认可的第三方制作的镜像。我们可以认为官方仓库提供的镜像都是安全的、最新的，所以也可以放心使用。

### 2.3. Docker 容器（ Container ）

容器是一个基于 Docker 镜像创建、包含为了运行某一特定程序的所有需要的 OS 、软件、配置文件和数据，是一个可移植的运行单元。在宿主机来看，它只不过是一个简单的用户进程而已。

容器启动的时候， Docker 会在镜像最上层挂载一个 read-write 的文件系统，即上图中标记为 writable 的 Container 层，容器将跑在这个文件系统上。这层可写的文件系统是容器中才有的概念，如果我们对此容器进行 commit 操作，那么该层文件系统则会被提交为一个新的只读的镜像层，并位于镜像层的最上面的。

我们可以认为 Docker 镜像是“静”的" .exe "文件，只在“硬盘”上；而容器是“动”的，是在“内存中”的，要想启动一个容器，需要先把" .exe "装载到内存。

镜像和容器具有如下的转换关系：

- 镜像 -> docker run -> 容器

- 容器 -> docker commit -> 镜像

有时候我们经常会将两个名称混用，不过这并不会影响我们的理解。

### 2.4. Docker Registry

Docker Registry 是 Docker 架构中的分发模块，它用来存储 Docker 镜像，我们可以将它理解为 GitHub 。

Docker Hub 是一个官方的 Docker Registry ，也是 Docker 镜像的默认存储位置。

当然从安全管理的角度上来说，我们可能更愿意在自己公司内部托管一个私有的 Docker Registry ，这可以通过使用Docker官方提供的 [Registry](https://github.com/dotcloud/docker-registry)  软件实现。

运行私有 Registry 非常简单，这也是一个典型的 Docker 风格的应用发布例子。

`docker run –p 5000:5000 registry`

## 3. 使用 Docker

### 3.1. 初识容器

#### 3.1.1. 创建并启动容器

这里我们假定各位读者已经在自己的机器上安装好了 Docker 。 Docker 主要的命令就是 docker 了，它的参数很多，关于它的具体使用方法，可以参考[官方文档1](https://docs.docker.com/reference/commandline/cli/) 和 [官方文档2](https://docs.docker.com/reference/run/)，这里我们只简单的介绍其中一些常用的用法。
 
启动一个容器很简单，我们只需要运行 docker run 命令就可以了。

>*作者注：为了方便区分，本文中运行命令的时候如果提示符为 $ ，表示实在宿主机（ Ubuntu ）中，如果是 # ，则表示是在 Docker 容器中。*

```
$ sudo docker run -i -t ubuntu /bin/bash
Unable to find image 'ubuntu' locally
Pulling repository ubuntu
e54ca5efa2e9: Pulling dependent layers
... 省略 ...
6c37f792ddac: Download complete
... 省略 ...
root@81874a4a6d2e:/#
```

docker run 命令会启动一个容器。参数 ubuntu 指定了我们需要运行的镜像名称，后面的 bash 则指定了要运行的命令，注意这个命令是容器中的命令，而不是宿主机中的命令。参数 -i 用来为容器打开标准输入以和宿主机进行交互， -t 则会为容器分配一个终端。

在第一次启动某镜像的时候，如果我们本地还没有这个镜像，则 Docker 会先从远程仓库（ Docker Hub ）将容器的镜像下载下来，下载完成之后才会启动容器。

注意 Docker 里有一个很重要的概念就是容器 ID 或者镜像 ID ，比如这个例子里的 e54ca5efa2e9 。这个 ID 是一个容器或者镜像的唯一标识，它的长度为 64 位，不过很多时候都可以简写为 12 位，这也和 Git 很像。

#### 3.1.2. 让 Docker 容器在后台运行

这时候我们可以使用 -d 参数来通过守护模式启动一个容器，这样容器将会在后台一直运行下去。这非常适合运行服务类程序。如果需要，我们可以再通过 docker attach 命令连接到运行中的容器。

#### 3.1.3. 常用命令

##### docker ps

docker ps 用来查看正在运行中的容器。

从下面的输出结果我们可以看出该容器状态（ STATUS 列）为已经停止执行，且没有错误（ Exited 后面的状态码）。

```
$ sudo docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
60bab6f881e5        ubuntu:latest       /bin/bash           14 minutes ago      Exited (0) 5 seconds ago                       agitated_hopper     
```
docker ps命令的常用参数（及组合）如下。

- -a： 查看所有容器，包括已经停止运行的。
- -l： 查看刚刚启动的容器。
- -q： 只显示容器 ID 。
- -l -q： 则可以返回刚启动的容器 ID 。

##### docker stop/start/restart

docker stop 用来停止运行中的容器，同时你还可以用 docker start 来重新启动一个已经停止的容器。

docker restart 可以重启一个运行中的容器。这就相当于对一个容器先进行 stop 再start。

###3.2. 深入了解 Docker 镜像

在对 Docker 容器有一个简单的感性认识之后，我们再来深入了解一下 Docker 镜像的概念。

Docker 镜像实际上就是一个 tarball ，它是一个能完整运行的 OS 系统，这非常像 OS 或 VM 镜像。它里面有基础 OS 、各种软件包及类库等。我们启动一个容器，相当于是启动了一个“基础 OS ”。

####3.2.1. 标签（ Tag ）

我们还可以为镜像打标签，这也和 Git 非常相似。其实你也可能在前面留意到了， docker images 的输出中有一列就是 TAG 的。我们在执行 docker build 或者 docker commit 的时候都可以同时为仓库名称指定一个 TAG ，格式为 user_name/repo_name:tag ，如果没有指定这个 TAG ，则默认为 latest 。

####3.2.2. 常见镜像操作

这里我们再介绍一下对镜像常见的一些操作。

##### 查看本地镜像列表

docker images 命令用来列出当前系统中的所有本地镜像，即我们已经通过 docker run 或者 docker pull 下载下来的镜像，镜像文件保存在本地的 /var/lib/docker 文件夹下。

##### 下载镜像到本地

只需要运行 docker pull 命令即可，命令非常简单，问题在于你的网路速度和连通性。

##### 删除镜像

docker rmi 用来从本地仓库中删除一个不再需要的镜像，即" rm image "的缩写。

### 3.3. 构建镜像

我们可以创建自己的 Docker 镜像，在我们的日常工作中会经常进行镜像构建操作。构建 Docker 镜像非常简单，而且方法也有几种。

#### 3.3.1. 手工创建

这个方法最简单直接的方法。其流程是启动一个容器，在里面进行一些列安装、配置操作，然后运行 docker commit 命令来将容器 commit 为一个新镜像。

```
$ sudo docker run -t -i ubuntu bash
root@c4be1df52810:/# apt-get update
root@c4be1df52810:/# apt-get -y install redis-server

root@c4be1df52810:/# exit
```

通过下面的命令得到刚才容器的 ID 号并进行 commit 操作。

```
$ sudo docker ps -q -l
c4be1df52810

$ sudo docker commit -m="manually created image" -a="bin liu <liubin0329@gmail.com>" -run='{"CMD":["/usr/bin/redis-server"], "PortSpecs": ["6379"]}' c4be1df52810 liubin/redis:manually

Warning: '-run' is deprecated, it will be removed soon. See usage.
744ce29b2fcf0ad7ad8b2a89c874db51376c3fdd65d1f4f0c6f233b72f8c3400
```

注意上面的警告信息，在 docker commit 命令指定 -run 选项已经不被推荐了，这里为了说明这个例子而故意使用了这个选项。建议创建镜像还是使用 Dockerfile 的方式，既能将创建过程代码化、透明化，还能进行版本化。

再次运行 docker images 命令，就应该能看到我们刚才通过 docker commit 命令创建的镜像了（镜像 ID 为 744ce29b2fcf ，镜像名为 liubin/redis:manually ）。

#### 3.3.2. 使用 Dockerfile 文件

##### 使用 Dockerfile 构建 Docker 镜像

这是一个官方推荐的方法，即将构建镜像的过程代码化，比如要安装什么软件，拷贝什么文件，进行什么样的配置等都用代码进行描述，然后运行 docker build 命令来创建镜像文件。官方的自动构建即是基于保存在 GitHub 等代码托管服务上的 Dockerfile 进行的。 Dockerfile 既是具体的用于构建的配置文件名，也是这类文件的类型名称。

使用 Dockerfile 构建 Docker 镜像非常简单，我们只需要创建一个名为 Dockerfile 的文件，并编写相应的安装、配置脚本就可以了。我们还是以上面安装 Redis 服务为例，看看如何使用 Dockerfile 构建一个镜像。

首先，创建一个 redis 文件夹（文件夹名任意，无任何限制），并进入该文件夹，然后创建一个 Dockerfile 文件。这个文件的文件名是固定的，其内容如下。

```
FROM        ubuntu
MAINTAINER  bin liu <liubin0329@gmail.com>
RUN         apt-get update
RUN         apt-get -y install redis-server
EXPOSE      6379
ENTRYPOINT  ["/usr/bin/redis-server"]
```

Dockerfile 文件的语法非常简单，每一行都是一条指令，注释则以 # 开头。每条指令都是“指令名称 参数”的形式，指令名称一般都是大写。比如 FROM 指令表明了我们的镜像的基础镜像（严格来说叫父镜像，我们的所有操作都将以此镜像为基础），这里是 ubuntu ，但实际上它可以是存在的任何镜像，比如 liubin/ruby 。 RUN 指令则用来在构建过程中执行各种命令、脚本，比如这里是 apt-get 命令，你也可以指定一个很复杂很长的脚本文件路径。 AUFS 有 42 层文件系统的[限制](https://github.com/dotcloud/docker/issues/1171)，这时候我们可以通过在 RUN 指令中执行多条命令，即 cmd1 && cmd2 && cmd3 && ... 这种形式就可以可避免该问题了。 EXPOSE 表示此镜像将对外提供6379端口的服务。 ENTRYPOINT 则指定了启动该镜像时的默认运行程序。

具体的 Dockerfile 语法在 [官方网站](https://docs.docker.com/reference/builder/)有详细说明，相信花个10分钟就能通读一遍，这里唯一比较容易混淆的就是  ENTRYPOINT 和 CMD 指令了，关于它们的区别，还是留作每位读者自己的课题去研究一下吧。

Dockerfile 准备好了之后，运行 docker build 命令即可构建镜像了。

`$ sudo docker build -t liubin/redis:dockerfile .`

这里 -t 表示为构建好的镜像设置一个仓库名称和 Tag（如果省略 Tag 的话则默认使用 latest ）。最后的一个 **.** 表示 Dockerfile 文件的所在路径，由于我们是在同一文件夹下运行 docker build 命令，所以使用了 **.** 。

由于篇幅所限，这里我们就省略了 docker build 命令的输出。不过如果你亲自动手执行 docker build 命令的话，那么从它的输出应该很容易理解。 Dockerfile 里的每一条指令，都对应着构建过程中的每一步，而且每一步都会生成一个新的类似容器的哈希值一样的镜像层 ID 。也正是这些层，使得镜像能共享很多信息，并且能进行版本管理、继承和分支关系管理等。这除了能节省大量磁盘空间之外，还能在构建镜像的时候通过使用已经构建过的层（即缓存）来大大加快了镜像构建的速度。比如在我们在使用 Dockerfile 进行构建镜像时，如果在某一步出错了，那么实际上之前步骤的操作已经被提交了，修改 Dockerfile 后再次进行构建的话， Docker 则足够聪明到会从出错的地方开始重新构建，因为前面的指令执行结构都已经被缓存了。

如果你使用 docker history 命令来查看该镜像的历史信息，你会发现它的输出和 docker build 的记录是相匹配的，每一条 Dockerfile 中的指令都会创建一个镜像层。此命令还能查看每个镜像层所占空间大小，即 SIZE 列的内容。比如本例中 MAINTAINER 这样指令，实际上它只是关于镜像的元数据，并不占用额外的磁盘空间，所以它的层大小为 0 字节。而 RUN apt-get -y install redis-server 创建的层则会在镜像中增加文件，所以是需要占用磁盘空间的。

##### 自动构建（ Automated Builds ）

Docker Hub 的目的之一就是要成为应用程序交换的中转站，它还支持自动构建功能。自动构建的 Dockerfile 可以托管在 GitHub 或者 Bitbucket 上，当我们将代码提交并 push 到托管仓库的时候，Docker Hub 会自动通过 webhook 来启动镜像构建任务。

配置自动构建很简单，只需要在 Docker Hub 中绑定 GitHub 或者 Bitbucket 账号就可以了，如何具体操作这里不做详细说明了。

#### 3.3.3. 使用 Packer

[Packer](http://www.packer.io/) 是一个通过配置文件创建一致机器镜像（ identical machine images ）的非常方便的工具。 Packer 同样出自 Vagrant 的作者 Mitchell Hashimoto 之手。它支持虚拟机 VirtualBox 和 VMWare 等虚拟机软件，以及 Amazon EC2 、 DigitalOcean 、 GCE 以及 OpenStack 等云平台，最新版的 Packer 也增加了对 Docker 的支持。

Packer 的使用也比较简单，这里我们就不举例说明了，读者可以自己试一下。

### 3.4. 发布镜像

如果你愿意，还可以将在本地制作镜像 push 到 Docker Hub 上和其他人分享你的工作成果。

首先你要有一个 Docker Hub 账号并已经为登录状态，这样才能往 Docker Hub 上 push 镜像文件。 Docker Hub 账号只能通过 [网站](https://hub.docker.com/) 注册，这里我们假设各位读者已经拥有了 Docker Hub 账号。

登录 Docker Hub 通过 docker login 命令。

登录成功后，我们就可以 push 镜像了。注意这里我们没有指定 Tag ， Docker 知道如何去做。

`$ sudo docker push liubin/redis`

我们前面说过，镜像文件是分层的，很多镜像文件可以共用很多层。比如我们这次往服务器 push 镜像的时候，实际 push 的只有一层（ 744ce29b2fcf ）而已，这是因为我们的镜像文件是基于 ubuntu 这个 base 镜像创建的，而 ubuntu 镜像早已经在远程仓库中了。

我们在层 744ce29b2fcf 中对应的操作是 bash 命令，并在容器中安装了 Redis 。而这次修改只有不到 6M 的容量增加，而如果只是修改配置文件的话，那么一次 push 操作可能只需要耗费几 K 的网络带宽而已。

## 4. DockerCon14 总结

首届 Docker 大会（ DockerCon14 ）于当地时间6月9日~6月10日在旧金山举行。相对于计划中的 500 个参会名额，最终有超过 900 人报名，并提交了超过 150 个演讲申请。

关于这次 Docker 大会的更多信息可以参考其官方网站：http://www.dockercon.com/ 。

### 4.1. Docker 官方发布的产品和服务

#### 4.1.1. Docker 1.0 的发布及商业支持

在这次大会上最重要的事情莫过于 Docker 1.0 的发布了。Docker 1.0 已经可以在Red Hat、Debian、Ubuntu、Fedora、SuSE 等主流 Linux 系统下运行，在功能、稳定性以及软件质量上都已经达到了企业使用的标准，文档也更加系统、完善。并且提供了 Docker Hub 云服务，方便开发者和企业进行应用分发。最重要的是 Docker, Inc. 还宣布了对 Docker 的商业支持，尤其是对 Docker 1.0 版本的长期支持。此外，Docker, Inc. 还会提供 Docker 相关的培训、咨询等工作。

#### 4.1.2. Docker Engine + Docker Hub

同时从 1.0 开始， Docker 的架构也发生了较大的变化。Docker 已经从单一的软件转变为了一个构建、发布、运行分布式应用的平台。

新的 Docker 平台由 Docker Engine（运行环境 + 打包工具）、Docker Hub（API + 生态系统）两部分组成。

##### Docker 引擎

Docker 引擎是一组开源软件，位于 Docker 平台的核心位置。它提供了容器运行时以及打包、管理等工具。

##### Docker Hub

Docker Hub 是一个云端的分布式应用服务，它专注于内容、协作和工作流。

Docker Hub 可以看作是原来 Docker index 服务的升级版。Docker Hub 除了可以托管 Docker 镜像之外，还提供了包括更管理、团队协作、生命周期流程自动化等功能，以及对第三方工具和服务的集成。

在 Docker, Inc. 看来，典型的基于 Docker Hub 的软件开发生命周期为：在本地基于 Docker 引擎开发 -> 打包应用程序 -> 将应用程序 push 到 Docker Hub  -> 从 Docker Hub 上下载此应用镜像并运行。它将镜像构建的任务交给 Dev ，将镜像部署的任务交给 Ops 。

#### 4.1.3. 新组件

Docker Engine 也有了一些新的变化，而部分功能实际上早在 Docker 0.9 就开始提供了。如果你还在运行 Docker 0.8 及其以前的版本的话，那么还是及早升级的比较好。

##### libswarm

libswarm 是一个" toolkit for composing network services ”。它定义了标准接口用于管理和编配一个分布式系统，并提供了一致的 API 。 libswarm 打算支持各种编配系统，虽然它看上去更像个高层接口封装的 API 而已。

##### libcaontainer

libcontainer 是一个容器的参考实现，它通过 Go 语言实现来使用 Linux 的命名空间等技术，而不需要额外的外部依赖。

实际上在 Docker 0.9 的时候这个模块就已经分离出来了，到了 1.0 的时候，此模块成为了独立项目并且可以单独使用。并且从 0.9 版本的时候开始 Docker 就已经开始就采用 libcontainer 来代替 LXC 作为默认的容器实现方式了，LXC 变成了可选项之一。

##### libchan

libchan 现在是 Docker 的标准通信层，被称为网络上的 go channel 。普通的 Go channel 只能运行在单机上，而 libchan 可以跨 Unix socket 或纯 TCP/TLS/HTTP2/SPDY/Websocket 等运行。使用 libchan ，可以非常方便的进行任意结构的消息传递、实时双工异步通信、并发编程及同步等。

最后我们再从下面的这张图，更形象的认识一下这三个工具的作用及关系。

![libchan,libcontainer,libswarm](http://liubin-org.u.qiniudn.com/2014/docker01/lib3.png/zoom1)

### 4.2. 大公司的热情

如果看一下 [演讲嘉宾列表](http://www.dockercon.com/speakers.html) ，你一定会感叹这阵容太豪华了。不错，很多演讲嘉宾都来自大型互联网公司，比如 Facebook、Twitter、Google、Heroku、Yelp 以及 Groupon 等，很多还都是 VP 、CTO 等高级别的管理人员，可见这次大会规格之高，分量之重。并且他们中的很多人还都进入到了 Docker 治理委员会。

#### 4.2.1. Google

前面我们已经介绍了 Google 公司内部的服务都是跑在容器之中的， Google 对 Docker 也表现出了相当浓厚的兴趣。除了他们负责基础设施的 VP Eric Brewer 进行了主题为 *Robust Containers* 的演讲之外，他们还介绍了自己开源容器管理软件 Kubernetes 和对容器资源进行监控的 cAdvisor 。

#### 4.2.2. Red Hat

Red Hat Enterprise Linux 7 版将内置 Docker ，虽然版本还是 0.11 ，不过很快就会升级的。另外 Atomic 项目也是 Red Hat 主导开发的。

### 4.3. 其它感受

其他一些笔者认为比较有意思的就是使用基于 Mesos 工具群来对容器进行集群管理了。比如 Twitter 和 Groupon 都做了使用 Mesos + Aurora/Marathon + ZooKeeper 在数据中心进行资源分配和管理的分享；甚至在 Twitter 看来，数据中心也可以看做是一台计算机， Mesos 就是这台计算机的 OS 。

另外就像我们前面在 Docker 使用场景中介绍过的那样，很多公司都在使用 Docker 进行持续集成。

## 5. Docker 现状及展望
在本节我们将会站在一个开放的角度和更高的层次来审视一下 Docker 的现状，包括其问题点，以及对 Docker 将来的可能性做一些肤浅的推测。

### 5.1. 生态系统

Docker 的发展离不开其生态系统，我们学习 Docker 也同样需对其生态系统有所了解。我们可以从下面三点来审视一下 Docker 当前的发展状况。

>*作者注：关于 Docker 的生态环境，大家也可以参考网上有人制作的一份 [思维导图]( http://www.mindmeister.com/389671722/docker-ecosystem) 。*

#### 5.1.1. 厂商支持

前面我们已经说过了，包括 RedHat 等在内的 Linux 发行商以及 Google、AWS、Rackspace 等云服务提供商都表示对 Docker 非常浓厚的兴趣，甚至已经进行了非常深入的实践。从这一点上来说， Docker 有非常好的政治背景。

#### 5.1.2. 开源项目

围绕 Docker 的开源项目就更多了，主要有以下几类，我们将挑选出一些比较有意思且开发较活跃的项目进行简单介绍。

##### PaaS 平台

PaaS 平台大多基于容器技术， Docker 天生就适合做 PaaS 。

- **[Flynn](https://flynn.io/)**

Flynn 是一个高度模块化的下一代开源 PaaS 实现。  Flynn 分为两层， Layer 0 是底层，也叫资源层，基于[ Google 的 Omega 论文](http://eurosys2013.tudos.org/wp-content/uploads/2013/paper/Schwarzkopf.pdf) 开发，这一层也包括服务发现。 Layer 1 则用来进行部署、管理应用程序。 Flynn 目前开发比较活跃，是一个值得关注的开源项目，而且今年夏天很可能就会发布 1.0 的版本了。

- **[Deis](http://deis.io/)**

Deis 是一个支持公有和私有 PaaS 的开源实现。它支持运行使用 Ruby 、 Python 、 Node.js 、 Java 、 PHP 和 Go 等语言进行应用开发，并能部署到 AWS 、 Rackspace 和 DigitalOcean 等云上。

##### CI/CD（持续集成/持续部署）

由于 Docker 的沙箱性、创建速度快等特性，它与生俱来也适合进行 CI/CD 。很多基于 Docker 的 CI/CD 开源方案和服务如雨后春笋般的涌现出来。

- **[Drone](https://drone.io/)**

开源的支持各种语言的 CI 工具，并且提供了 CI/CD 服务 Drone.io 。

- **[Strider CD](http://stridercd.com/)**

开源的 CI/CD 方案，集成 GitHub 。

##### 私有仓库托管（ Registry ）/容器托管

这类服务主要进行私有仓库的托管，根据用户的托管仓库数量收费。 Docker Hub也提供私有仓库的收费套餐。

- **[Quay](https://quay.io/)**

Quay 除了能托管私有镜像之外，还能和 GitHub 集成，使用 Dockerfile 进行镜像构建。

- **[Shippable](https://www.shippable.com/)**

Shippable 支持 Github 和 Bitbucket ，并且提供100%免费的服务，包括私有仓库。

- **[Orchard](https://www.orchardup.com/)**

Orchard 也是一个和 StackDock 类似的 Docker 托管服务，它提供了便捷的命令行工具来运行各种 Docker 命令。同时它也提供免费的私有 Registry 服务，前面介绍的 Fig 工具就是此公司开发的。

笔者认为传统的云计算服务提供商除了在云主机上提供对容器的支持之外，说不定将来还会提供专门托管容器的服务。

##### 开发管理工具

软件工程师天生就是闲不住和想尽一切办法要提高自己效率的一群人。这里我们简单介绍两个方便进行 Docker 开发的工具。

- **[Shipyard](https://github.com/shipyard/shipyard)**

Shipyard 是一个 Docker 镜像和容器管理工具，除了基本的镜像构建，容器启动等功能之外，它还具有在浏览器中attach到容器的功能，并通过 [hipache](https://github.com/dotcloud/hipache) ( Hipache: a distributed HTTP and websocket proxy ) 来进行容器之间的连接。同时它也支持跨节点的 Docker 管理和容器 Metrics 采集。
 
- **[Fig](http://orchardup.github.io/fig/index.html)**

Fig 是一个为了提高基于 Docker 开发的效率而创建的工具，它通过一个配置文件来管理多个 Docker 容器，非常适合组合使用多个容器进行开发的场景。

#### 5.1.3. 社区

Docker 开发社区非常活跃，除了 35 名全职员工（外加一只乌龟）之外，还有 450 名左右的外部代码贡献者。到目前 Docker Hub 已经拥有超过 16000 多个应用，在 GitHub 上也有超过 7000 个 Docker 相关的项目，其中不乏很多受关注度非常高的项目。

在 Twitter 、科技媒体以及个人 Blog 上，每天都能看到很多关于 Docker 的内容。

线下社区活动也在蓬勃展开中。在世界范围内除了南极洲， Docker Meetup 已经遍布 35 个国家 100 多个城市，北京在今年3月8日举行了国内第一次的 Docker Meetup ，当时有超过40人报名参加。第二次北京 Docker Meetup 已在七月中举行。

### 5.2. 运用中的问题点

虽然 Docker 很火，有时候我们也需要反过来看看它还有哪些不令我们满意的地方，或者说在使用上还存有疑虑。当然这里的问题都是笔者个人主观看法，只是非常片面的一部分，各位读者一定要带着批判性的思维去理解它。

#### 5.2.1. Debug 、调优

查看日志可能是最简单直接的方式了。当然也有很多人都会在 Docker 容器中运行一个 SSHD 服务，然后通过 SSH 登录到容器中去，不过不建议使用这种方法。

官方推荐使用 [nsenter](https://github.com/jpetazzo/nsenter) 工具来完成类似的工作，通过它可以进入到指定的 namespace 中并控制一个容器。

#### 5.2.2. 数据管理

这里所说的数据包括数据库文件、 Log 、用户上传的文件等。

在容器中要想处理数据文件，可能最简单的方式就是通过共享卷标来实现，即 docker run -v 。但是随之带来的问题是既然是文件，都存在备份问题，如何备份？用 ftp 或者在容器和宿主机之间共享文件夹的方式？而且随着容器数量的增多，对共享卷标的管理也势必会更复杂。

笔者认为理想的解决方法就是使用云服务，比如数据库使用 RDS ，文件使用 S3 。如果不想使用云服务，则可以考虑自己通过 FastDFS 等实现自己的“云存储”。 Log 则通过 fluentd/logstash 进行集计再用 Graphite/Kibana 等进行可视化。

#### 5.2.3. 如何和配置管理工具配合使用

到底在容器时代，还需不需要传统的 Puppet 或 Chef 这样的配置管理工具？当然，从配置管理工具的角度来说，他们都不会放弃对 Docker 的支持，比如 Puppet 就已经增加了对 Docker（安装、管理镜像和容器）的支持。

但随着不可变基础设施的普及[作者注：作者个人观点]，幂等性将不再重要，因为我们的容器只需要配置一次。要对容器做出修改，可能只需要修改 Dockerfile/manifest/recipe 文件重新 Provisioning 即可。而且也不需要在容器内部安装任何 agent ，这样的话类似 Ansible 这样纯 SSH 的配置管理工具比较适合对 Docker 进行配置。甚至还可能出现专门为 Docker 的更简单的配置管理工具。


#### 5.2.4. 安全性

是软件就会存在 bug ，包括安全漏洞， Docker 也不例外。就在今年6月份，Docker 刚爆出了一个容器溢出的 [漏洞](http://blog.docker.com/category/security-2/) 。不管是 Hypervisor 技术还是容器技术，安全问题始终都是一个不可避免的话题，虽然它们出问题的几率要比中间件软件（ Apache，Nginx、Tomcat ）和软件框架（ Struts、Rails ）等的概率要小很多。

事后 Docker, Inc. 还是比较积极的面对了这件事，除了及时披露详细情况之外，还着重强调了他们的安全政策。

#### 5.2.5. 有状态和无状态容器

在不可变基础设施（Immutable Infrastructure）里，一切都可以分为有状态（stateful）的和无状态（stateless）的，容器也不例外。容器似乎更适合跑无状态的服务，然而业内对如何分别对待这两种服务还没有太好的最佳实践。

### 5.3. 对 Docker 展望

最后再容笔者斗胆对 Docker 的将来做一些展望。除了 Docker 本身自己会蓬勃发展之外，围绕Docker的生态圈必将更加成熟和强大。

#### 5.3.1. 集群管理（Orchestration）和服务发现（Service Discovery）

相对于对单台机器进行 Provisioning 而言，云环境下则需要对多台机器进行 Orchestration 。 Orchestration 这个词翻译过来就是编排、编配的意思，我们也可以理解为集群管理。它主要由两部分工作组成：

- 监控服务器，发现变化（软硬件异常、网络异常、正常变更等）

- 根据监视事件采取相应的行动。

##### 服务发现

在松耦合的分布式环境下，应用程序不一定跑在同一台机上，甚至是跨越数据中心的。这时候服务发现就显得格外重要了。

- **[Zookeeper](http://zookeeper.apache.org/)**

[Chubby](http://research.google.com/archive/chubby.html) 可以称得上是很多服务发现、集群管理软件的鼻祖了，包括 Zookeeper 。这些软件都提供数据存储、 leader 选举、元数据存储、分布式锁、事件监听（或 watch ，监视）等功能。

- **[etcd](https://github.com/coreos/etcd)**

etcd 很新也很轻量，安装很简单，配置也不复杂，所以非常适合入门。etcd 存储的是 key-value 格式的数据。

etcd 是 CoreOS 的一个组件。同时 CoreOS 提供了一个基于公有云的服务发现服务 discovery.etcd.io 。
 
- 此外，我们还可以有 **[Skydns/Skydock](https://github.com/crosbymichael/skydock) ** 、**[Discoverd](https://github.com/flynn/discoverd)** 等选择。

>*作者注：Skydns/Skydock 是基于 DNS 的服务发现。*

>*作者注：Discoverd 是 Flynn 的一个组件，它目前是基于 etcd 的，但是也可以扩展诸如 Zookeeper 等分布式存储机制。*

##### 集群管理

围绕 Docker 使用场景的开源集群管理软件有很多，比如 Geard、Fleet、Consul及 Serf 等，这些软件都是随着 Docker 应运而生的；此外还有很多老牌的集群管理软件，比如 Mesos 等也可以很好的结合 Docker 使用。

- **[Serf](http://www.serfdom.io/)** 和 **[Consul](http://www.consul.io/)**

Serf 是一个基于 Gossip 协议去中心的服务器发现和集群管理工具，它非常轻量，高可用并具备容错机制。

Consul 是一个服务发现和集群配置共享的软件，除了K/V store功能之外，它还支持跨数据中心及容错功能，并能进行服务健康监测。

这两个软件都是 Vagrant 作者所在公司 [HashiCorp](http://www.hashicorp.com/products) 发布的产品，这个公司也值得大家关注。

- **Apache Mesos & Marathon & deimos & etc**

Mesos 用于对多个节点的资源进行管理，它将多台服务器作为一台“虚拟机”看待，并在这台虚拟机上分配资源，用户通过使用 framework 进行资源管理。 Marathon 是一个 Mesos 的 framework ，用来启动、管理需要长时间运行的任务。deimos 则是一个为 Mesos 准备的 Docker 插件。

- 其它工具

Cloud Foundry 在5月份发布的 Docker 版的 BOSH 工具，有兴趣的读者可以参考一下 **[Decker](http://www.cloudcredo.com/decker-docker-cloud-foundry/)** 项目。

另外 **[Clocker](https://github.com/brooklyncentral/clocker) **这个项目也比较有意思，它基于 Apache Brooklyn（目前还在孵化器中），能在多云环境下基于 Docker 容器进行应用部署。这个项目的扩展性很好，非常方便自己定制。不过项目还太年轻，要想使用的话恐怕还需要些时日。
 
#### 5.3.2. 和 OS 的深度结合

在 Fedora 上使用的 systemd 就已经提供了集成容器和虚拟机的功能。

>*作者注： systemd 是用来替代 Linux 中 init 系统的系统软件，目前已经在 Fedora/RHEL 等中采用。*

Docker 除了能在各种主流 Linux 上使用之外，还出现了有专为运行 Docker 容器而定制的 OS 了，比如 [CoreOS](https://coreos.com) 、 RedHat 的 [Atomic](http://www.projectatomic.io/)。

##### CoreOS

CoreOS 是一个精简版的 Linux ，可以运行在既有硬件或者云上，它也是一个最近备受关注的项目。 CoreOS 不提供类似 yum 或者 apt 的包管理工具，你不需要在 CoreOS 中安装软件，而是让程序都在 Docker  容器中去运行。CoreOS 使用 systemd 和 fleet 来对容器进行管理，通过 etcd 进行服务发现和配置信息共享。

##### Atomic

Project Atomic 是最近才发布的一个项目，它也是一个瘦身版的 Linux ，只包含 [systemd/geard](http://openshift.github.io/geard/) /rpm-OSTree 以及 Docker 组件，专门用来部署和管理 Docker 容器。它能在接近硬件裸机级别上高性能的运行大量容器，而且它还是基于 SELinux 的，在安全上也有保障。

#### 5.3.3. Container 技术规范化和兼容性

就在 DockerCon14 开始的前一天， Flynn 发布了 Pinkerton ，一个支持在其它容器中使用 Docker 镜像的技术。

而另一方面，我们知道除了 LXC 、 Docker 之外，还有很多其它容器技术，比如Zones 、 jail 和 LMCTFY 等，那么试想这么多的容器之上，是否有统一接口、互相兼容或者在容器上加一层封装的可能性呢？比如让一种容器的镜像，能运行到其它容器中？ Docker 容器已经能互相连接了，会不会异构的容器之间也能进行某种交互呢？

## 6. 总结
Docker 虽然入门和使用起来非常简单，但整个生态系统还是挺庞大的，而且其底层技术也都很复杂，由于篇幅有限及笔者学识不精，也只能说一些皮毛之事，最多只能算是抛块砖而已；而且笔者也有一种意犹未尽的感觉，但是由于篇幅所限，不能说到面面俱到，更多的内容，还请各位读者自己去深入挖掘。

总之笔者认为 Docker 还是非常有趣的一个东西，值得大家花些时间体验一下，相信在各位的工作中多多少少都能用的上 Docker 。

***

##### 这篇文章最初由 [刘斌](http://weibo.com/pmproad) 发表于 [个人网站](http://liubin.org/) ，我们得到其授权后将其转载，并对错别字进行了修改。为了阅读方便，也对格式进行了调整。您可以点击 [这里](http://liubin.org/2014/08/11/docker-cloud-app-delivery-style/) 来阅读 [原始版本](http://liubin.org/2014/08/11/docker-cloud-app-delivery-style/)。 

##### 如果您有任何疑问，欢迎通过 [微博](http://weibo.com/pmproad) 、[GitHub](http://weibo.com/pmproad) 向作者提问；也欢迎您在 [Twitter](https://twitter.com/OurColorfulDays) 上 follow [@OurColorfulDays](https://twitter.com/OurColorfulDays) 。
