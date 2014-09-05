#十个让你乐开怀的 Docker 技巧

#####作者：[Nathan LeClaire](https://twitter.com/upthecyberpunks)

#####译者：[刘梦馨](http://weibo.com/oilbeater)

***

{<1>}![alt](http://resource.docker.cn/humpback.jpg)

作为 Docker 公司的一名解决方案工程师，我已经积累了很多使用 Docker 的建议和技巧。 Docker 社区每天都在产生着大量的内容，但是很多能让工作变得更容易更有乐趣的建议和技巧很容易就消失在这茫茫信息的洪流中，被人们忽视。

一旦你掌握了这些基础，创造性就永无止境，就像“寒武纪生命大爆发”那样无限涌出， Docker 正在带来这种让人兴奋的变化。

现在我将和你们分享我最喜欢的十条建议和技巧，你们准备好了么？

1. 在 VPS 上运行 Docker 来获得额外的速度
2. 在 Docker run 上 bind mount socket
3. 将容器作为可以高度任意使用的开发环境
4. bash 是你的好朋友
5. Insta-nyan
6. 在 OSX 上利用 `boot2docker` ip 地址修改 `/etc/hosts/`
7. `docker inspect -f` 的魔法
8. 利用 wetty 建立一个方便的浏览器终端
9. nsenter
10. #docker

## 在 VPS 上运行 Docker 来获得额外的速度 
{<2>}![alt](http://resource.docker.cn/vps.jpeg)

这是一个直接了当的建议。如果你和我一样家里的带宽很小的话，你可以在 [Digital Ocean](http://digitalocean.com/) 或者 [Linode](http://linode.com/) 上来部署 Docker 以获得更好的 pull 和 push 带宽。在家里 Comcast 网络下，我的下载带宽大约有 50mbps，然而在我 Linode 主机上的速度要比这快一个数量级。

所以如果你对速度有要求的话，考虑投资一个 VPS 作为你的个人 Docker 训练基地吧。这会节省你的生命，尤其是当你在咖啡馆利用 WiFi 或者在其他网络连接条件不好的时候。

## 在 Docker run 上 bind mount socket

在 privileged 模式下，想在容器里做一些 Docker 相关的事情，可是又不想运行一个完整的 Docker in Docker (dind) ，该怎么办？你可以使用一个装有 Docker 客户端的 image 并且使用 -v 参数 bind-mount 自己的 Docker socket 。


    docker run -it -v /var/run/docker.sock:/var/run/docker.sock nathanleclaire/devbox

现在你可以利用同一个 docker daemon 向你容器内的 Docker 发送 docker 命令了。

这样做充满乐趣，不仅可以让你混用主机上的 Docker 容器，同时拥有灵活性和轻量级的容器。同时也引出了下一个建议……

## 将容器作为可以高度任意使用的开发环境

{<3>}![at](http://resource.docker.cn/devenv.gif)

有多少次你需要快速隔离一个问题来观察它是否与特定事件相关？或者只是想切换到一个新分支，来对现在的环境进行些许细小的更改或实验，而不想导致大的意外发生？

Docker 提供了一种轻便的解决方法。

你只需要简单的制作一个 Dockerfile 定义你理想的开发环境所需的 CLI（包括 ack 、 autojump 、 Go 等等任何你想要的东西），并且开启该 image 的新实例，你就可以获得一个全新的机器并试验你的想法。下面是 Docker 创始人 [Solomon Hykes](https://github.com/shykes) 的一个开发环境。
    
    FROM ubuntu:14.04
    
    RUN apt-get update -y
    RUN apt-get install -y mercurial
    RUN apt-get install -y git
    RUN apt-get install -y python
    RUN apt-get install -y curl
    RUN apt-get install -y vim
    RUN apt-get install -y strace
    RUN apt-get install -y diffstat
    RUN apt-get install -y pkg-config
    RUN apt-get install -y cmake
    RUN apt-get install -y build-essential
    RUN apt-get install -y tcpdump
    RUN apt-get install -y screen
    # Install go
    RUN curl https://go.googlecode.com/files/go1.2.1.linux-amd64.tar.gz | tar -C /usr/local -zx
    
    ENV GOROOT /usr/local/go
    ENV PATH /usr/local/go/bin:$PATH
    # Setup home environment
    RUN useradd dev
    RUN mkdir /home/dev && chown -R dev:/home/dev
    RUN mkdir -p /home/dev/go /home/dev/bin /home/dev/lib /home/dev/include
    ENV PATH /home/dev/bin:$PATH
    ENV PKG_CONFIG_PATH /home/dev/lib/pkgconfig
    ENV LD_LIBRARY_PATH /home/dev/lib
    ENV GOPATH /home/dev/go:$GOPATH
    
    RUN go get github.com/dotcloud/gordon/pulls
    # Create a shared data volume
    # We need to create an empty file, otherwise the volume will
    # belong to root.
    # This is probably a Docker bug.
    RUN mkdir /var/shared/RUN touch /var/shared/placeholder
    RUN chown -R dev:dev /var/shared
    VOLUME /var/shared
    WORKDIR /home/dev
    ENV HOME /home/dev
    ADD vimrc /home/dev/.vimrc
    ADD vim /home/dev/.vim
    ADD bash_profile /home/dev/.bash_profile
    ADD gitconfig /home/dev/.gitconfig
    
    # Link in shared parts of the home directory
    RUN ln -s /var/shared/.ssh
    RUN ln -s /var/shared/.bash_history
    RUN ln -s /var/shared/.maintainercfg
    RUN chown -R dev:/home/dev
    USER dev

如果用 vim/emacs 来编辑这些东西的话会累死人的。你可以用 /bin/bash 作为你的 CMD 并且用 docker run -it my/devbox 。

当你运行容器时你可以 bind-mount Docker 二进制客户端到 socket 上（如上一条建议），在容器内访问主机 Docker daemon 。

类似的你可以快速地在新机器上用类似的方法搭建开发环境、安装 Docker 并且下载你的开发 image 。

## Bash 是你的朋友 

或者更宽泛地说，“shell 是你的朋友”。

就像你可以在 git 中利用 alias 来减少键盘敲击一样，你也可以自己建立常用 Docker 命令的快捷键。只需要把他们加入你的 ~/.bashrc 或其他等价的文件中即可。

下列是一些常用的快捷键:

    alias drm="docker rm"
    alias dps="docker ps"

我会在每次发现重复使用相同命令的时候进行设置。自动化是我的一大乐趣。

你也可以将这些快捷键以一种有趣的方式组合，例如你可以这样

    $ drm -f $(dps -aq)

来移除所有的容器（包括运行的容器）。或者你也可以使用

    function da () {  
        docker start $1 && docker attach $1
    }

开启一个停止的容器并绑定它。

我做了一个有趣的命令，可以让我开启 rapid-bash-container-prompt 来提醒我之前的技巧：

    function newbox () {
        docker run -it --name $1 \
        --volumes-from=volume_container \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -e BOX_NAME=$1 nathanleclaire/devbox
    }

## Insta-nyan

{<4>}![alt](http://resource.docker.cn/nyan.png)

十分简单，如果你想在终端显示一个 nyan 猫，并且你有 Docker 的话，你只需要执行下面的命令。

    docker run -it supertest2014/nyan

## 在 OSX 上利用 boot2docker ip 地址修改 /etc/hosts/ 

{<5>}![alt](http://resource.docker.cn/hacking.png)

最新版本的 [boot2docker](https://github.com/boot2docker/boot2docker) 包含了一个本地网络，通过它你可以通过 boot2docker 的虚拟机 ip 地址来访问容器的端口。 boot2doker ip 命令可以使你轻松地访问这些值。由于这个地址通常是 192.168.59.103 ，难记且不好拼，所以我在自己的 /etc/hosts 中加入了一个条目让我可以更容易的访问 boot2docker:port 。这个十分简单，试一下吧。 

**注意**：boot2docker 虚拟机的 IP 地址可能会改变，所以在使用前要考虑可能遇到的网络问题。如果你没有进行可能使网络配置变复杂的操作（开启并关闭多个虚拟机和 boot2docker ），你可能不太会碰到相关问题。

如果你在使用它的话，应该感谢一下 boot2docker 的作者 [@SvenDowideit](http://twitter.com/SvenDowideit) ，正是他持续维护着 boot2docker 。

## docker inspect -f 的魔法

如果你愿意学一些 [Go 模板](http://golang.org/pkg/text/template/) 的话，你可以通过 -f（或者 --format）标识，用 docker inspect 来做各种有意思的事情。

通常 docker inspect $ID 会打印一个大的 JSON 串，你可以通过模板像下面这样访问每个值：

    docker inspect -f '{{ .NetworkSettings.IPAddress }}' $ID

参数 -f 后面接的是一个 Go 模板。如果你做下面这些事情：

    $ docker inspect -f '{{ .NetworkSettings }}' $ID
    map[Bridge:docker0 Gateway:172.17.42.1IPAddress:172.17.0.4IPPrefixLen:16PortMapping:<nil>Ports:map[5000/tcp:[map[HostIp:0.0.0.0HostPort:5000]]]]

你将不会得到一个 JSON 串。但是如果你像下面这样：

    $ docker inspect -f '{{ json .NetworkSettings }}' $ID
    {"Bridge":"docker0","Gateway":"172.17.42.1","IPAddress":"172.17.0.4","IPPrefixLen":16,"PortMapping":null,"Ports":{"5000/tcp":[{"HostIp":"0.0.0.0","HostPort":"5000"}]}}

你将会得到一个 JSON 串，并且可以通过一个 Python 内置模块来美化它：

    $ docker inspect -f '{{ json .NetworkSettings }}' $ID | python -mjson.tool
    {
    	"Bridge": "docker0",
    	"Gateway": "172.17.42.1",
    	"IPAddress": "172.17.0.4",
    	"IPPrefixLen": 16,
    	"PortMapping": null,
    	"Ports": {
    		"5000/tcp": [
    			{
    				"HostIp": "0.0.0.0",
    				"HostPort": "5000"
    			}
    		]
    	}
    }

你还可以做其他一些有趣的事情，比如访问那些非字母关键字的对象属性，这可以帮你更好的了解 Golang 。

    docker inspect -f '{{ index .Volumes "/host/path" }}' $ID

这是一个帮助你提取容器运行信息的利器。由于他提供了大量的细节，他对调试十分有帮助。

## 利用 wetty 建立一个方便的浏览器终端 

我见过人们用 Docker 做一些有意思的 web 应用。你可以试着跑一个 [wetty](https://github.com/krishnasrinivas/wetty)（一个  JavaScript 制作的浏览器内的模拟终端）的实例。

你可以自己试一下：

    docker run -p 3000:3000 -dt nathanleclaire/wetty

{<6>}![alt](http://resource.docker.cn/wetty.png)

遗憾的是 wetty 只能在 Chrome 下工作。不过已经有其他的 JavaScript 终端模拟器将要 Docker 化。你可以通过控制浏览器来进行演示（想想一下你可以在你的 Reveal.js 幻灯片中展示一个可交互的 CLI 快照）。现在你可以在 web 应用中集成独立的终端应用并且你可以细粒度的控制你的环境的每个细节，无需担心主机和容器之间的任何污染。

## Nsenter

Docker 的工程师 [Jérôme Petazzoni](http://twitter.com/jpetazzo) 在几周前写了一篇令人瞩目的文章。在文章中他强调不要在容器中运行 sshd ，这样做违反 Docker 准则（每个容器只有一个关注点）。这篇文章十分值得一读。其中，他提到了 nsenter ，一个在已经初始化的容器中获得命令行的小技巧。

看 [这里](http://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/) 和 [这里](http://www.sebastien-han.fr/blog/2014/01/27/access-a-container-without-ssh/) 了解更多。

## #docker 

我不是在谈论标签！我谈论的是我们在 Freenode 上的 IRC 频道——在互联网上遇到正宗 Docker 爱好者的最好的地方。我们欢迎提问，并且你会遇到真正的专家。不论何时都有着大约一千人在线，是一个极好的社区和资源。如果你之前没试过的话，那么现在试一下。我知道 IRC 可能让那些不熟悉的人感到神秘，但是我保证学习使用它的代价和通过它获得的收获是不可比的。所以如果你还没有用过 IRC 的话赶紧来试一下吧。

加入我们：

1. 下载 IRC 客户端比如 [LimeChat](http://limechat.net/mac/)
2. 链接 irc.freenode.net
3. 加入 #docker 频道

欢迎你的加入！

## 结论 

这就是现在我们所知道的全部了。欢迎在 twitter 上关注 [@docker](https://twitter.com/docker)，告诉我们你最喜欢的 Docker 技巧吧！


***

#####这篇文章由 [Nathan LeClaire](https://twitter.com/upthecyberpunks) 撰写，[刘梦馨](http://weibo.com/oilbeater) 翻译。点击 [这里](http://blog.docker.com/2014/07/10-docker-tips-and-tricks-that-will-make-you-sing-a-whale-song-of-joy/) 阅读原文。

#####The article was contributed by [Nathan LeClaire](https://twitter.com/upthecyberpunks), click [here](http://blog.docker.com/2014/07/10-docker-tips-and-tricks-that-will-make-you-sing-a-whale-song-of-joy/) to read the original publication.
