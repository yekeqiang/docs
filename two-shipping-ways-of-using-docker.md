# 两种 Docker 的“分发”方法

##### 作者：[Perficient](https://twitter.com/Perficient) 

##### 译者：[陈菊英](http://weibo.com/u/1716255775)
***

[docker](http://www.docker.com) 为开发人员和系统管理员提供了一个可供开发，分发( ship )和运行应用的平台。**将 Docker 化 的应用及其依赖环境不需要经过任何修改就可以分发到任何地方--提供给 QA 、团队成员或者分发到云平台中**。这是使用 Docker 的一个很重要的目的。

在 [Vernon Stinebaker](https://blogs.perficient.com/multi-shoring/blog/author/vstinebaker/) 之前的文章 [“Docker, mobile, and putting things in boxes”](http://blogs.perficient.com/multi-shoring/blog/2014/07/03/docker-mobile-and-putting-things-in-boxes/) 中，他介绍了 library system 的背景和因此产生的手机应用（Android版本和iOS 版本），以及对此进行 docker 化的必要。作为“ Library in a box ”项目的成员之一，我主要研究的部分是“把东西放进盒子里”。 

在本文中，我不是要介绍 docker 化的具体步骤，而是介绍两种 docker 的“分发”方法。

目标是对应用进行 docker 化。由于项目日程的紧迫，我和另外一个小组成员决定首先用交互的方式安装必要的软件把基础框架先搭建起来，例如：Java 7 、 Liferay 和 mysql 等。然后将 library 应用程序的 war 包部署到 container 中。 这几个步骤和在一般的环境中做的的准备工作和部署没有什么区别。 唯一的区别是安装和部署工作是在一个 docker 的 container 中进行，而不是在一个虚拟机上。按照下面命令中高亮的部分，用"**-t**"  和 "**-i**" 参数来启动交互式 container 。

```
$ docker run -t -i ubuntu:trusty/bin/bash
```

一旦 container 启动起来之后，我们将这个 container 作为 docker image 提交。这两个步骤完成后，我们把这个 docker images 用 docker save 这个命令提交到 PO ([Vernon Stinebaker](https://blogs.perficient.com/multi-shoring/blog/author/vstinebaker/)) 。

```
$ docker save library/app > library_app.tar.gz
```

PO 在得到了 library_app.tar.gz 这个文件之后就可以用 docker load 这个命令将 image 加载进自己的本地系统中。

```
$ docker load < library_app.tar.gz
```

当我们把这个 image 交付给 QA 并让他们只用一个简单的命令就可以将这个应用运行起来的时候，他们表示很惊讶。因为通常如果要将应用在 QA 的机器中运行起来，他们需要和 DEV 组沟通协作做好安装和配置工作。

除了用 save-load 命令， docker 鼓励大家将 images push 到 [Docker Hub](https://hub.docker.com) 平台，这样别人就可以通过 pull 命令从 [Docker Hub](https://hub.docker.com) 拿到 image 并直接运行。但是我们不想使用 push-pull 方法的主要原因在于这个 image 实在太庞大，足有 **2.3G** 。我们试图通过多种方法来减小这个 image 的大小，例如：将安装包删除， cache 清除，但最后这个 image 还是依然有**1.4个G**。这个如果从 [Docker Hub](https://hub.docker.com) 里 push 或者 pull 就有点太大了，你说是不是？尤其 PO 和 QA 其实和 DEV 组是在同一个子网中，拷贝 image 通常只需要几分钟。如果我们想把 image 分发给在不同网络环境中的组织或个人，其实更加推荐用 push-pull 命令来完成。所花时间时间完全取决于你的网络状况。

现在我们开始 [Dockerfile](https://docs.docker.com/reference/builder) 的编写，只要你“记录”的步骤遵循 Dockerfile 的规则， docker 将会从零开始将 docker image 给你生成出来。

```
$ docker build -t library/app .
```

这个 Dockfile 只有**2.4KB**大小，算上**22MB**大小的 library 应用 war 包也只有**23MB**左右。可见，利用 Dockerfile 生成 image ，让分发的应用小了很多。那么，我们就应该一直用 Dockerfile 来 build 和 run 应用么？

答案是否定的，因为依照 Dockerfile 中的 build 步骤， image 的生成过程依然需要下载，安装和配置变量等。举个例子，在我家里的机器上，生成“ Library in a box ”的 docker image 需要接近2.5个小时。

现在，我们了解了docker的两种分发方式：

	1. 你可以分发 container ，这样用户可以直接用它跑应用。
	2. 你可以分发 Dockerfile ，用户可以用它自己生成 container ，然后跑应用。

但是到目前，我们也没有一个结论说哪种方式更好。从一个程序员的角度来讲，我更愿意开发和维护 Dockerfile ，因为当我们在 container 中用交互的方式去下载、安装、配置软件包时，或多或少我们都会遗留一些无用的东西在里面。

***
##### 这篇文章由 [Perficient](https://twitter.com/Perficient)  撰写，[陈菊英](http://weibo.com/u/1716255775) 翻译。点击 [这里](http://blogs.perficient.com/multi-shoring/blog/2014/07/22/two-shipping-ways-of-using-docker/) 阅读原文。

##### The article was contributed by [Perficient](https://twitter.com/Perficient), click [here](http://blogs.perficient.com/multi-shoring/blog/2014/07/22/two-shipping-ways-of-using-docker/) to read the original publication.

