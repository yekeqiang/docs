#让 Docker 无处不在

#####作者：[Timo Derstappen](https://twitter.com/teemow)
#####译者：[Mark Shao](http://weibo.com/1889619455)

***

我答应过会继续写一些关于 docker 的文章。作为 CØ 中唯一一枚有本地 Linux 内核的幸运儿，我当仁不让地写点东西来分享自己如何在本地台式机上管理所有项目。

## 为什么是 docker 而不是纯 LXC

几年前当我还在 Adcloud 的时候，开始使用 LXC 来建立所有的 [微型服务](http://yobriefca.se/blog/2013/04/29/micro-service-architecture/) 。鉴于我在台式机上使用 Archlinux，效果还不错。但是我们的生产环境是 Ubuntu 的操作系统，并且会用 chef 脚本来创建容器，而我仍然把这些容器当做机器来维护。这些容器有自己的 DNS 域名、 IP 地址、 SSH 服务，和所有文件的磁盘绑定。所以每当我开始一个项目时，都得先通过 SSH 登录到机器然后再开始工作。并且我一点也不喜欢 Chef ，所以离开 Adcloud 后我就放弃 Chef ，开始使用 Babushka 。虽然后者能让我以最简单的方式来部署一个单机环境，但是创建新容器十分缓慢，容器起来以后占据了我 ssd 上的大量磁盘空间，并且我还需要手动去配置多个服务。


所以当我第一次启动 docker 容器的时候，它的速度之快真是让人惊讶。更棒的是分层的文件系统可以减少磁盘的使用。

## 容器不等于机器

有了 docker 容器以后，你就不再需要考虑机器的概念了。它们只是在不同的内核命名空间中的进程，你希望它们可以保持精简和干净。它们也需要能够运行在不同的环境中，所以我不再在容器中使用 SSH 。那我又是如何来创建我的项目呢？首先我创建了一系列容器，在上面安装了我项目中需要安装但不应该安装在本地的工具。

## 在盒子里使用Go

我们先来看一下 github 上的 [Dockerfile](https://github.com/teemow/docker-go/blob/master/Dockerfile)

使用下面的别名在容器中启动 go ，再把容器移除。

```
alias go="docker run --rm -t -i teemow/go"
```

你甚至可以使用不同的标签来运行不同版本的 go （ go:1.1 、 go:1.2 等）。不过要让它它真正工作，还需要更多操作。你可以把你当前的目录绑定挂载到容器中，来做类似`go get`这样的事情。

```
alias go="docker run --rm -t -i -v \$(pwd):\$(pwd) -w \$(pwd) teemow/go"
```

如果你有一个私有的 git 库的话，你甚至可以把你当前的 ssh 代理挂载到容器中.

```
alias go="AGENT=\$(ls -1 --sort t /tmp/ssh-*/agent.* | head -1) && docker run --rm -t -i -v \$AGENT:\$AGENT -e SSH_AUTH_SOCK=\$AGENT -v \$(pwd):\$(pwd) -w \$(pwd) teemow/go"
```

类似 GOPATH 这样的环境变量也可以传入到容器中。

你可以完全用你自己的工具链来完成这些事情，这样你的主机会很干净。别忘了给你的不同版本的镜像打上标签，否则你很难跳回到之前的项目中。也许你可以给类似 rbenv （ dockerenv ）的项目做记号。我已经创建了一系列 [包含工具的镜像](https://github.com/search?q=%40teemow+docker) ，比如 npm 、 grunt 、 coffee-script 、 aws-cli 和 tugboat 等等。

专家提醒: 不要过度使用别名，建议尽可能使用小脚本，比如 /usr/local/bin/ 。

## 像玩乐高那样玩 Docker 

使用上面的一系列别名，你可以构建你的项目，但是如果你想运行或者测试项目，通常需要多个容器。你不能在同一个容器内运行你的 webservice 、 postgres 和 redis 服务。每个服务都运行在自己的容器里面，现在你需要把它们装配在一起。

Docker 的 [links](http://docs.docker.io/en/latest/use/working_with_links_names/) 功能可以帮你实现。一个被链接的容器可以通过环境变量来实现自我描述，这样另外一台容器可以发现这个被链接的容器的 IP 地址和端口。 Heroku 添加插件信息也是使用了相似的方法。

你可以使用 [fig](http://orchardup.github.io/fig/) 让一切变得更简单，只需要在你的项目里面添加一个 fig.yml 文件，然后运行 `fig up` 来启动整个环境。本地目录会被绑定到容器中，通过一个文件观察器可以在文件被修改后自动重启服务。 Mac/Vagrant 用户应该很熟悉这个。我已经在我们的 [piratesinn angellist list](https://github.com/catalyst-zero/thepiratesinn-startups-api/blob/master/fig.yml) 中添加了 fig.yml 文件。

事实上就连 fig 自身也是运行在我本机上的一个容器中。不过你需要把 docker 的 socket 服务绑定到 fig 的容器中，这样它才可以启动其他的容器。

***

#####这篇文章由 [Timo Derstappen](https://twitter.com/teemow) 发表，[Mark Shao](http://weibo.com/1889619455) 翻译。点击 [这里](http://catalyst-zero.com/dockerize-all-the-things/?utm_source=Docker+News&utm_campaign=c3d355131c-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-c3d355131c-235722981) 可查阅原文。

#####The article was contributed by [Timo Derstappen](https://twitter.com/teemow), click [here](http://catalyst-zero.com/dockerize-all-the-things/?utm_source=Docker+News&utm_campaign=c3d355131c-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-c3d355131c-235722981) to read the original publication. 

