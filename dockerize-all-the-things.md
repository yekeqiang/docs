# Dockerize all the things
# 让Docker无处不在

Hej, as promised I'd like to continue with some docker related posts. Since I'm the only lucky guy at CØ with a native linux kernel I'll write a little bit about how I manage all the projects on my local desktop.

就像我承诺的一样，我会继续写一些关于docker的文章。因为我是CØ中唯一一个有本地Linux内核的幸运儿。我将会写一点关于我如何在我的台式机上管理我所有项目的。

## Why docker instead of plain LXC
## 为什么是docker而不是纯LXC

A few years ago I started using LXCs to setup all the micro services we created at Adcloud. This was nice, since I'm using Archlinux on my desktop, but we had Ubuntu machines in production and it was even possible to provision the containers with our chef recipes. But I was still handling those containers as if they were machines. They had a dns name, ip address, ssh and a bind mount with all my dotfiles. So everytime I worked on a project I first ssh'd into the container to work on it. I never really liked Chef that much, so I dropped it after Adcloud and started using Babushka, which was way simpler and enough to provision a single machine. But creating new containers was still slow, all the containers started to occupy a lot of space on my ssd, and orchestration of multiple services was something I had to do by hand.

几年前当我还在Adcloud的时候，我开始使用LXC来建立所有的[微型服务](http://yobriefca.se/blog/2013/04/29/micro-service-architecture/).这个看上去很不错，因为我在我的台式机上使用Archlinux，但是我们的生产环境是Ubuntu的操作系统而且甚至有可能我们我们的chef脚本来创建Containers.而且我仍然把这些Containers当做一台机器来。他们有一个DNS的域名，IP地址，SSH服务和一个有所有文件的磁盘绑定。所以每次当我开始工作在一个项目上的时候，我需要先通过SSH登录到机器然后再开始工作。实际上我一点也不喜欢Chef,所以在我离开Adcloud后我就放弃Chef,开始使用Babushka,它可以让我们以最简单的方式来部署一个单机的环境。但是创建新的containers仍然十分缓慢，containers起来以后占据了我ssd上的大量的磁盘空间，而且我不得不需要手动去配置多个服务。

So when I first fired up a docker container it was just awesome to see how fast it is. Even better with the layered fs they will use way less space on your disk.

所以当我第一次启动一个docker的container的时候，它的速度之快真是让人觉得惊讶。更好的是分层的文件系统可以减少磁盘的使用。

## Containers != machines
## 容器不是一台机器

With the docker containers you should stop thinking about machines. They are just processes in a separate kernel namespace and you want to keep them lean and clean. They should also run in different environments as is. So I stopped sshing in containers. But how do I setup my projects? First I created a handful of containers that wrap tools I use in my projects but shouldn't be installed on my box itself.

有了docker的容器以后，你就不再需要考虑机器的概念了。它们只是在不同的内核命名空间中得进程，你希望它们可以保持精简和干净。它们也需要能够运行在不同的环境中。所以我不再通过SSH的方式连接到containter中。但是我又是如何来建立我的项目呢？首先我创建一系列containers，它们上面安装了我项目中所需要的工具，但是它们没有被装在一个盒子里。

## Go in a box
## 在盒子里使用Go

Look at the Dockerfile on github.
我们来看一下在github中得[Dockerfile](https://github.com/teemow/docker-go/blob/master/Dockerfile)

The following alias starts go in a container and removes the container afterwards.

下面的别名在容器中启动go，然后再把容器移除

```
alias go="docker run --rm -t -i teemow/go"
```

You could even use tags to run different versions of go (go:1.1, go:1.2 etc). But you need some more to really work with it. You can bind mount the current directory into the container to do things like go get.

你甚至可以使用不同的标签来运行不同版本的go(go:1.1, go:1.2 等等).但是你需要更多的东西让它真正工作起来。你可以把你当前的目录绑定挂载到容器中，来做类似`go get`这样的事情。

```
alias go="docker run --rm -t -i -v \$(pwd):\$(pwd) -w \$(pwd) teemow/go"
```

And if you have private git repositories you can even bind mount your current ssh agent into the container.

如果你有一个私有的git库的话，你甚至可以把你当前的ssh代理挂载到容器中.

```
alias go="AGENT=\$(ls -1 --sort t /tmp/ssh-*/agent.* | head -1) && docker run --rm -t -i -v \$AGENT:\$AGENT -e SSH_AUTH_SOCK=\$AGENT -v \$(pwd):\$(pwd) -w \$(pwd) teemow/go"
```

Environment variables like GOPATH can be passed to the container as well.

类似GOPATH这样的环境变量也可以传入到容器中

You can do this with all of your tools. So your host system will stay clean. Don't forget to tag the versions of your images. Otherwise you won't be able to jump into an old project easily. Maybe you can pin them within your project similar to rbenv (dockerenv anyone?). I've created images for tools like npm, grunt, coffee-script aws-cli and tugboat etc

你可以完全用你自己的工具链来完成这些事情。这样你的主机会很干净。别忘了给你的不同版本的镜像打上标签。否则你很难跳回到之前一个老的项目中。也许你可以学习rbenv给你的项目做一个记号(dockerenv?).我已经创建了一些列[包含工具的镜像](https://github.com/search?q=%40teemow+docker)，比如npm，grunt，coffee-script，aws-cli和tugboat等等。

Expert tip: Don't overuse the alias – instead create small scripts in eg /usr/local/bin.

专家提醒: 不要过度使用别名 - 尽可能使用小的脚本，必须/usr/local/bin/

## Docker Lego
## Docker的乐高玩具

With the aliases above you can build your project, but if you'd like to run or test your project there are usually more than one containers involved. You are not running your webservice, postgres and redis within the same container. Everything has its own container and now you need to plug all the bricks together.

使用上面的一些列别名你可以构建你的项目，但是如果你想运行或者测试你的项目，通常需要不止一个容器。你没有在同一个容器内运行你的webservice，postgres和redis服务。每个服务都运行在自己的容器里面，现在你需要把它们装配在一起。

Docker has a feature called links that helps you doing this. A linked container will introduce itself via environment variables. So the other container can find ip address and port of the linked container. Heroku adds information about the addons in a very similar way. 

Docker有一个叫[links](http://docs.docker.io/en/latest/use/working_with_links_names/)得功能可以帮你做到。一个被链接的容器可以通过环境变量来实现自我描述。这样另外一台容器可以发现这个被链接的容器的IP地址和端口。Heroku添加关于插件信息也是使用了相似的方法。



But you can make this even easier by using [fig](http://orchardup.github.io/fig/). Just add a fig.yml file to your repository and run fig up to start the whole environment. The local directory can be bound to a container and with a file watcher you can restart your service if it was modified. Mac/Vagrant users will be familiar with this. I've added such a fig.yml in our piratesinn angellist list.

你可以使用[fig](http://orchardup.github.io/fig/)让一切变得更简单。只需要在你的项目里面添加一个fig.yml文件，然后运行`fig up`来启动整个环境。本地目录会被绑定到容器中，而且通过一个文件观察器可以在文件被修改后自动重启你得服务。Mac/Vagrant的用户应该对这个很习惯了。我已经在我们的[piratesinn angellist list](https://github.com/catalyst-zero/thepiratesinn-startups-api/blob/master/fig.yml)中添加了fig.yml文件。

Actually even fig itself runs within a container on my machine. But you need to bind mount the docker socket to the fig container so it can start other containers.

事实上甚至连fig自身也是运行在我本机上的一个容器中。但是你需要把docker的socket服务绑定到fig的容器中，这样它才可以启动其他的容器。

Stay tuned.
保持联系
