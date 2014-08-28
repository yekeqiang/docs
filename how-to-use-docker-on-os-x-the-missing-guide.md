# 操作指南：在 OS X 系统上部署 Docker

***

##### 作者：[CHRIS JONES](http://viget.com/about/team/cjones)
##### 译者：[moonatcs](http://blog.yege.me/)

***

你知道 [Docker](https://www.docker.com/) 吗？很可能你已经听说过它了，很多人在谈论它，Docker 越来越火了。 甚至像我父亲这样的人都会问，“Docker 是什么东西啊？我看到很多人在 Facebook 上提到它。”

Docker 很大程度上简化了 [containers](http://en.wikipedia.org/wiki/Software_container) 的运行和管理。它具有从各个方面改变服务端应用程序（server-side applications）的潜力，从开发和测试到部署和扩展。

最近，我仔细翻阅了 [The Docker Book](http://dockerbook.com/) 这本书。这是一本相当好的书，我强烈的推荐它。但是，当我在 OS X 上运行书上的示例时，发现会存在一些问题。因为，在书里假定我们使用的是 Linux 系统，而且跳过了一些额外的配置。但是，如果要在 OS X 上运行书上的示例， 这些配置却是必不可少的。这并不是这本书的错，相反，它提到了如何在 OS X 上运行 Docker 这一潜在的问题。

这篇文章总结了 在 OS X 上运行 Docker 要面对的各种问题以及解决这些问题的具体方法。 但这并不意味着本文是一个关于 Docker 自身的教程，但我希望你能跟着这篇文章逐步的做完并且把每一个命令行都敲一遍。这样，你会更好的理解 Docker 通常是如何工作的，而具体到 OS X 上又是如何工作的。除此之外，如果你想在 Mac 上 对 Docker 有更深入的理解，这会节省你很多排查错误的时间。

首先我们来谈一下 Docker 是如何工作的以及为什么在 OS X 上运行它会存在问题。

## Docker 如何工作

Docker 是一个  client-server 应用。 Docker  **server**  是一个后台进程（daemon），要完成所有重要的工作： 建立和下载 images 、开始和结束 containers 等等。 它提供了用于远程管理的 REST API。

Docker **client** 是一个命令行程序，通过 REST API 与 Docker server 进行通信。从 client 发送命令到 server，就可以与 Docker 进行交互了。

运行 Docker server 的主机被称为 Docker **host** 。 host 可以是任何一台机器 —— 笔记本电脑、Cloud™ 上的服务器等等 —— 但是，由于 Docker 只能用于 Linux 这个特点，所以这些机器上必须运行的是 Linux （更具体点，是 Linux 内核）。

### Docker on Linux 

如果我们要在 Linux 笔记本电脑上直接运行 containers：

![](http://f.cl.ly/items/242F0M1e2B0B0J0L3S0n/Screen%20Shot%202014-08-20%20at%2012.14.11%20PM.png)  
DOCKING ON LINUX

不难发现，笔记本运行 client 的同时，也作为 Docker 的主机运行着 server 。

### Docker on OS X

要知道，OS X 不是 Linux， 它不能在直在本地运行 Docker 容器，而是需要在某个地方运行一个 Linux 环境。

这就需要  [boot2docker]( http://boot2docker.io/ ) 。 boot2docker 是一个“ 轻量级  Linux distribution ，专门用来运行 Docker containers。” 我们要在 Mac 上的 VM 中运行它。

这张图说明了我们是怎样使用 boot2docker ：

![](http://cl.ly/image/351d3i291t0O/Screen%20Shot%202014-08-20%20at%2012.14.47%20PM.png)   
DOCKING ON OS X

我们在 OS X 本地上运行 Docker client ， 而 Docker server 是在 boot2docker VM 里面运行。 也就是说， boot2docker 才是 Docker host 而不是 OS X。

能理解吗？ 现在，让我们来安装 DAT 软件。

## 安装 Docker 环境


### Step 1： 安装 VirtualBox
查看[具体方法]( https://www.virtualbox.org/ )，在这里不做过多介绍。

### Step 2: 安装 Docker 和 boot2docker

有两种安装方式可供你选择： [Docker 官方]( https://docs.docker.com/installation/mac/ )提供的方法 或者是 [homebrew]( http://brew.sh/ ) 方法。 我比较喜欢 homebrew 因为我喜欢使用命令行管理我的环境。你可以任选一种方法。

	> brew update
	> brew install docker
	> brew install boot2docker

### Step 3: 初始化和启动 boot2docker

首先，初始化 boot2docker （只需要做这一次）

    > boot2docker init
    2014/08/21 13:49:33 Downloading boot2docker ISO image...
        [ ... ]
    2014/08/21 13:49:50 Done. Type `boot2docker up` to start the VM.

然后，启动 VM。

    > boot2docker up
    2014/08/21 13:51:29 Waiting for VM to be started...
    .......
    2014/08/21 13:51:50 Started.
    2014/08/21 13:51:51   Trying to get IP one more time
    2014/08/21 13:51:51 To connect the Docker client to the Docker daemon, please set:
    2014/08/21 13:51:51     export DOCKER_HOST=tcp://192.168.59.103:2375

### Step 4: 配置 DOCKER_HOST 环境变量

Docker client 会以为 Docker host 是当前的机器，我们需要对`DOCKER_HOST` 环境变量进行配置，这样它才会去使用 boot2docker VM。

	> export DOCKER_HOST=tcp://192.168.59.103:2375

*Your VM might have a different IP address—use whatever boot2docker up told you to use. You probably want to add that environment variable to your shell config.*

### Step 5: 测试和使用 Docker

测试一下：

    > docker info
    Containers: 0
    Images: 0
    Storage Driver: aufs
     Root Dir: /mnt/sda1/var/lib/docker/aufs
     Dirs: 0
    Execution Driver: native-0.2
    Kernel Version: 3.15.3-tinycore64
    Debug mode (server): true
    Debug mode (client): false
    Fds: 10
    Goroutines: 10
    EventsListeners: 0
    Init Path: /usr/local/bin/docker
    Sockets: [unix:///var/run/docker.sock tcp://0.0.0.0:2375]

大功告成。总结一下这个过程，首先安装了一个运行 boot2docker 的 VirtualBox VM；然后在 VM 中启动 Docker server；最后通过 OS X 上的 Docker client 与 server 进行通信。

我们可以启动 containers 了。

## 通常存在的问题


我们有了一个可以“工作的” Docker 平台，那么来看一下它在哪里会出问题并如果解决这些问题。

### Problem #1: 端口映射（Port Forwarding）

**问题：** Docker 从 containers 到 host 的端口映射，这个 host 是 boot2docker 而不是 OS X。

让我们启动一个容器来运行 [nginx]( http://nginx.com/ ) ：

    > docker run -d -P --name web nginx
    Unable to find image 'nginx' locally
    Pulling repository nginx
        [ ... ]
    0092c03e1eba5da5ccf9f858cf825af307aa24431978e75c1df431e22e03b4c3

这个命令启动了一个新的 container 作为后台进程（daemon(`-d`)），自动映射到 image(`-P`) 中特定端口，容器命名为 ‘web’(`--name web`) ，并且使用`nginx` image ，新 container 的唯一标识是 `0092c03e1eba....` 。

验证 container 运行：

    > docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
    0092c03e1eba        nginx:latest        nginx               44 seconds ago      Up 41 seconds       0.0.0.0:49153->80/tcp   web


在 PORTS 下面，我们可以看到 container 使用的是 80 端口，而且 Docker 已经将 container 的这个端口映射到了 host 上的一个随机端口 49153 。

[curl](http://curl.haxx.se/) 一下我们的新站点：

    > curl localhost:49153
    curl: (7) Failed connect to localhost:49153; Connection refused

它没有正常工作，为什么呢？

Docker 将 80 端口与 Docker host 上的 49153 进行了绑定。如果是在 Linux 上， Docker host 就是 localhost ，但是现在情况有所不同，我们用的是 VM 。

> 关于端口映射可以看一下[官方文档](https://docs.docker.com/articles/networking/#tldr)，有详细的介绍。

**解决方法：** 使用 VM 的 IP 地址。

通过 boot2docker 命令可以得到 VM 的 IP 地址：

    > boot2docker ip

    The VM’s Host only interface IP address is: 192.168.59.103

使用 curl 命令测试一下：

    > curl $(boot2docker ip):49153

    The VM’s Host only interface IP address is:

    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
        [ ... ]



虽然，得到了 web 页面，但是却也收到 `The VM’s Host only interface IP address is:` 这个消息。
`boot2docker ip` 输出 IP 地址到标准输出，而 `The VM’s Host only interface IP address is:` 是一个标准错误输出。`$(boot2docker ip)` 子命令可以捕获标准输出但是却不能捕获标准错误输出，所以最后在终端输出了。

这很招人厌，下面的 bash 函数可以解决它：

    docker-ip() {
      boot2docker ip 2> /dev/null
    }


把上面的代码拷贝到 shell 配置文件中，然后运行一下：

    > curl $(docker-ip):49153
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
        [ ... ]

这样，在终端里面就有了这个 IP 地址的引用，当然，也可以为其他应用程序（例如，浏览器）做一个类似的 IP 地址引用。在 `/etc/hosts` 增加 `dockerhost` :

	> echo $(docker-ip) dockerhost | sudo tee -a /etc/hosts

于是就可以在任何地方使用它了：

![](http://f.cl.ly/items/2O1p1Q0A0B0P0U0x2M0H/Screen%20Shot%202014-08-20%20at%2011.30.34%20AM.png)

到这里，这个问题已经被很好的解决了。在继续之前，我们先停止并且移除这个 container 。

    > docker stop web
    > docker rm web

*VirtualBox assigns IP addresses using DHCP, meaning the IP address could change. If you’re only using one VM, it should always get the same IP, but if you’re VMing on the reg, it could change. Fair warning.*


**更好的解决办法：**  把 Docker 的所有端口都从 VM 映射到 localhost 。

如果你想使用 localhost 来访问 Docker container ，可以把 Docker 整个范围内的端口从 VM 映射到 localhost 。 bash 代码如下，出自[这里](https://github.com/boot2docker/boot2docker/blob/master/doc/WORKAROUNDS.md#port-forwarding-on-steroids):

    #!/bin/bash

    for i in {49000..49900}; do
      VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i";
      VBoxManage modifyvm "boot2docker-vm" --natpf1 "udp-port$i,udp,,$i,,$i";
    done

这样，Docker 就会将 80 端口 映射到 VM 上的 49153 端口（假如是这个端口），而 VirtualBox 会将 49153 端口从 VM 映射到 localhost 。

### Problem #2: 挂载卷（Mounting Volumes）

**问题：** Docker 挂载的卷来自于 boot2docker VM ，而不是 OS X。

Docker 支持 volumes：可以将 host 上的一个目录挂载到你的 container，Volumes 是 container 访问外部资源的一个途径。例如，我们可以启动一个 nginx 容器，使用 volumes 来为 host 上的文件提供服务。让我们试试。

首先，创建一个目录，并且添加一个 `index.html`

    > cd /Users/Chris
    > mkdir web
    > cd web
    > echo 'yay!' > index.html

（别忘了，用你自己的路径替换`/Users/Chris`）

然后，启动一个 nginx container ，这次将刚才建立的目录挂载在 container 里面 nginx 的 web 根目录上：

    > docker run -d -P -v /Users/Chris/web:/usr/local/nginx/html --name web nginx
    485386b95ee49556b2cf669ea785dffff2ef3eb7f94d93982926579414eec278

查看一下 container 中 80 端口所对应的端口：

    > docker port web 80
    0.0.0.0:49154

curl 一下新的 index.html 页面：

    > curl dockerhost:49154
    <html>
    <head><title>403 Forbidden</title></head>
    <body bgcolor="white">
    <center><h1>403 Forbidden</h1></center>
    <hr><center>nginx/1.7.1</center>
    </body>
    </html>

不难发现，它并没有正常工作。这个问题出现的原因也与 VM 相关。 Docker 试图将 host 上的 `/Users/Chris/web`  挂载到 container 里面，但是 boot2docker 是 host 而不是 OS  X 。但是 boot2docker 对 OS X 上的文件可是一无所知。

**解决方法：** 将 OS X 的 `/Users` 目录挂载到 VM 上。

将 `/Users` 目录挂载到 VM 上， boot2docker 就可以得到一个 `/Users` volume ，而且这个 volume 指向 OS X 上相同的路径。 于是 boot2docker 里面的 `/Users/Chris/web` 也指向了 OS X 上的 `/Users/Chris/web` 目录，而且 可以将 `/Users` 中的任何路径挂载到 container 中。

boot2docker 并不支持 VirtualBox Guest Additions , 不允许我们那样做。幸运的是，[有人](https://medium.com/@mkadenbach)已经[解决了这个问题](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c) , 通过对 boot2docker 定义，使 boot2docker 包含了 Guest Additions 和 相关配置，使问题得以解决。

首先，移出 web container ，关闭 VM ：

    > docker stop web
    > docker rm web
    > boot2docker down


然后，下载上述自定义生成的 boot2docker ：

    > curl http://static.dockerfiles.io/boot2docker-v1.2.0-virtualbox-guest-additions-v4.3.14.iso > ~/.boot2docker/boot2docker.iso

最后，将 /Users 目录在 VM 上共享，并且启动 VM ：

    > VBoxManage sharedfolder add boot2docker-vm -name home -hostpath /Users
    > boot2docker up

*Replacing the boot2docker image won’t erase any of the data in your VM, so don’t worry about losing any of your containers. Good guy boot2docker.*


让我们再次试一下 能否访问 web：

    > docker run -d -P -v /Users/Chris/web:/usr/local/nginx/html --name web nginx
    0d208064a1ac3c475415c247ea90772d5c60985841e809ec372eba14a4beea3a
    > docker port web 80
    0.0.0.0:49153
    > curl dockerhost:49153
    yay!

再验证一下所使用的 volume ，在 OS X 上新建一个文件，看看 nginx 是否能对它启动服务：

    > echo 'hooray!' > hooray.html
    > curl dockerhost:49153/hooray.html
    hooray!

最后别忘了停止并移出 container：

    > docker stop web
    > docker rm web

*If you update `index.html` and curl it, you won’t see your changes. This is because nginx ships with `sendfile` turned on, which doesn’t play well with VirtualBox. The solution is simple—turn off `sendfile` in the nginx config file—but outside the scope of this post.*


### PROBLEM #3: 进入容器的内部

**问题：** 如何进入容器内部？

现在，你已经将新建的 container 运行起来了， 端口映射已经成功， 挂载卷也已经完成。貌似所有事情都妥妥的了，但是当你想启动 container 中的一个 shell 并且敲几个命令行时，你会发现这个还做不到。

**解决方法：** Linux Magic

请看 [nsenter](https://github.com/jpetazzo/nsenter)。nsenter 允许你在 内核命名空间中运行命令行。 因为 每个 container 都是一个运行在它自己的内核命名空间中的进程，而我们也正需要在容器的内部启动一个 shell 。

*This part deals with shells running in three different places. Trés confusing. I’ll use a different prompt to distinguish each*

* `>` *for OS X*
* `$` *for the boot2docker VM*
* `%` *for inside a Docker container*

首先，建立一个ssh连接，来访问 boot2docker VM ：

	boot2docker ssh> boot2docker ssh

然后，安装 `nsenter`:

	$ docker run --rm -v /var/lib/boot2docker:/target jpetazzo/nsenter

*(How does that install it? `jpetazzo/nsenter` is [a Docker image configured to build nsenter from source](https://github.com/jpetazzo/nsenter/blob/master/Dockerfile). When we start a container from this image, it builds nsenter and installs it to `/target`, which we’ve set to be a volume pointing to `/var/lib/boot2docker` in our VM.*

*In other words, we start a prepackaged build environment for nsenter, which compiles and installs it to our VM using a volume. How awesome is that? Seriously, how awesome? Answer me!)*

最后，我们需要在 VM 中 把 `/var/lib/boot2docker`添加 到 `docker` 用户的`PATH` ：

    $ echo 'export PATH=/var/lib/boot2docker:$PATH' >> ~/.profile
    $ source ~/.profile

现在，我们能使用 nsenter了 ：

    $ which nsenter
    /var/lib/boot2docker/nsenter

再次启动 nginx container ，看看它的运行状况（这时我们一直用 SSH 访问 VM）：

    $ docker run -d -P --name web nginx
    f4c1b9530fefaf2ac4fedac15fd56aa4e26a1a01fe418bbf25b2a4509a32957f

该访问容器的内部了。nsenter 需要这个正运行的 container 的 pid。 获取 pid：

	$ PID=$(docker inspect --format '{{ .State.Pid }}' web)


这一刻终于到来了：

    $ sudo nsenter -m -u -n -i -p -t $PID
    % hostname
    f4c1b9530fef

成功了，有木有！那么接下来，确认一下是否真的在容器里面了，查看一下正在运行的进程（必须要安装 `ps`）：

    % apt-get update
    % apt-get install -y procps
    % ps -A
      PID TTY          TIME CMD
        1 ?        00:00:00 nginx
        8 ?        00:00:00 nginx
       29 ?        00:00:00 bash
      237 ?        00:00:00 ps
    % exit


我们可以看到两个 nginx 进程，一个是 shell 另一个是 ps，表明确实进入了容器内部。

获得 pid 并将它传给 `nsenter` 有点麻烦，jpetazzo/nsenter 中包含了 [docker-enter](https://github.com/jpetazzo/nsenter/blob/master/docker-enter) 这个 shell 脚本, 它能解决这个问题：

    $ sudo docker-enter web
    % hostname
    f4c1b9530fef
    % exit

它的默认命令行是 `sh`，我们可以运行任何命令行：

 	sudo docker-enter web ps -A
      PID TTY          TIME CMD
        1 ?        00:00:00 nginx
        8 ?        00:00:00 nginx
      245 ?        00:00:00 ps

这样做已经很棒了，如果能够直接在 OS X 做这些，那就更好了。  [jpetazzo](https://github.com/jpetazzo) 仍然能够可以帮助我们完成这件事( 这家伙考虑得十分周到)，只需要在 OS X 上安装一个  [bash 脚本](https://github.com/jpetazzo/nsenter#docker-enter-with-boot2docker) 。下面的脚本与之类似，只做了微小的改变。


只需要将以下的代码复制粘贴到 OS X PATH 上（并且 `chom +x ` 它），所有就到设置好了：

	#!/bin/bash
	set -e
 
    # Check for nsenter. If not found, install it
    boot2docker ssh '[ -f /var/lib/boot2docker/nsenter ] || docker run --rm -v /var/lib/boot2docker/:/target jpetazzo/nsenter'

    # Use bash if no command is specified
    args=$@
    if [[ $# = 1 ]]; then
      args+=(/bin/bash)
    fi

    boot2docker ssh -t sudo /var/lib/boot2docker/docker-enter "${args[@]}"

测试一下：

    > docker-enter web
    % hostname
    f4c1b9530fef

最后不要忘了，停止并移除 container

    > docker stop web
    > docker rm web

##结束语

你现在已经有一个运行在 OS X 上的 Docker 环境了，做你想做的事情吧。 也许你急切的想知道 Docker 是如何工作的以及如何去使用它。


如果你打算深入学习 Docker，读一读 [The Docker Book](http://dockerbook.com/)，我力荐这本书，花点钱去[这里](http://dockerbook.com/)买一本还是值得的。

##展望未来


Docker 可能会成为一颗冉冉升起的新星，但是我们已经考虑将它加入到工作流中了。敬请期待它的时代吧。

这篇文章对你有帮助吗？你是如何使用 Docker 的？ 可以在下面给留言评论。


***

#####这篇文章由 [CHRIS JONES](http://viget.com/about/team/cjones) 撰写，[moonatcs](http://blog.yege.me) 翻译。点击 [这里](http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide) 阅读原文。

#####The article was contributed by [CHRIS JONES](http://viget.com/about/team/cjones) , click [here](http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide) to read the original publication.
 
