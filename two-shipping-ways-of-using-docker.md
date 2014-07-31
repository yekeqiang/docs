# 两种docker的“发布”方法

***
作者：[Perficient](http://blogs.perficient.com/multi-shoring/blog/2014/07/22/two-shipping-ways-of-using-docker/) 

译者：[Chenia](http://weibo.com/u/1716255775)
***

[docker](http://www.docker.com) 为开发人员和系统管理员提供了一个可供开发，发布(ship)和运行应用的平台。**将Docker化的应用及其依赖环境不需要经过任何修改就可以发布到任何地方--提供给QA，团队成员或者发布到云平台中**。这是使用Docker的一个很重要的目的。

在 [Vernon Stinebaker](https://blogs.perficient.com/multi-shoring/blog/author/vstinebaker/) 之前的文章 [“Docker, mobile, and putting things in boxes”](http://blogs.perficient.com/multi-shoring/blog/2014/07/03/docker-mobile-and-putting-things-in-boxes/)中，他介绍了图书系统的背景和因此产生的手机应用（Android版本和iOS 版本）以及对此进行docker化的必要。作为“盒子里的图书馆”项目的成员之一，我主要研究的部分是“把东西放进盒子里”。 

在本文中，我不是要介绍docker化的具体步骤，而是介绍两种docker的“发布”方法。

目标是对应用进行docker化。由于项目日程的紧迫，我和另外一个小组成员决定首先用交互的方式安装必要的软件把基础框架先搭建起来，例如：Java 7，Liferay和mysql等。 然后将图书馆应用程序的war包部署到container中。 这几个步骤和在一般的环境中做的的准备工作和部署没有什么区别。 唯一的区别是安装和部署工作是在一个docker的container中进行而不是在一个虚拟机上。如下面命令中高亮的部分，用"**-t**"  和 "**-i**" 参数来启动交互式container。

```
$ docker run -t -i ubuntu:trusty/bin/bash
```

一旦container启动起来之后，我们将这个container作为docker image提交。这两个步骤完成后，我们把这个docker images用docker save这个命令提交到PO ([Vernon Stinebaker](https://blogs.perficient.com/multi-shoring/blog/author/vstinebaker/))。

```
$ docker save library/app > library_app.tar.gz
```

PO 在得到了 library_app.tar.gz 这个文件之后就可以用docker load这个命令将image加载进自己的本地系统中。

```
$ docker load < library_app.tar.gz
```

当我们把这个image交付给QA并让他们只用一个简单的命令就可以将这个应用运行起来的时候，他们表示很惊讶。因为通常如果要将应用在QA的机器中运行起来，他们需要和DEV组沟通协作做好安装和配置工作。

除了用save-load命令，docker其实鼓励大家将images push到[Docker Hub](https://hub.docker.com)平台，这样别人就可以通过pull命令从[Docker Hub](https://hub.docker.com)拿到image并直接运行。但是我们不想使用push-pull方法的主要原因在于这个image实在太庞大，足有**2.3G**。我们试图通过多种方法来减小这个image的大小，例如：将安装包删除，cache 清除，但最后这个image还是依然有**1.4个G**。这个如果从[Docker Hub](https://hub.docker.com)push或者从pull就有点太大了，你说是不是？尤其是，PO和QA其实和DEV组是在同一个子网中，拷贝image通常只需要几分钟。但如果我们想把image发布给分布在不通网络环境中的组织或个人，其实更加推荐用push-pull命令来完成。这需要花多少时间完全取决于你的网络状况。

因此我们开始[Dockerfile](https://docs.docker.com/reference/builder)的编写，只要你“记录”的步骤遵循Dockerfile的规则，docker将会从零开始将docker image给你生成出来。

```
$ docker build -t library/app .
```

这个Dockfile只有**2.4KB**大小，算上**22MB**大小的图书馆应用war包也只有**23MB**左右。可见，利用Dockerfile生成image，让发布的应用小了很多。由此，我们就应该一直用Dockerfile来build和run应用么？

答案是否定的，因为依照Dockerfile中的build步骤，image的生成过程依然需要下载，安装和配置变量等。举个例子，在我家里的机器上，生成“盒子里的图书馆”的docker image需要接近2.5个小时。

现在，我们了解了docker的两种发布方式：

	1. 你可以发布container，这样用户可以直接用它跑应用。
	2. 你可以发布Dockerfile，用户可以用它自己生成container，然后跑应用。

但是到目前，我们也没有一个结论说哪种方式更好。从一个程序员的角度来讲，我更愿意开发和维护Dockerfile，因为当我们在container中用交互的方式去下载，安装，配置软件包时，或多或少我们都会遗留一些无用的东西在里面。

***
这篇文章由 [Perficient](http://blogs.perficient.com/multi-shoring/blog/2014/07/22/two-shipping-ways-of-using-docker/) 撰写，[Chenia](http://weibo.com/u/1716255775) 翻译。点击 [这里](http://blogs.perficient.com/multi-shoring/blog/2014/07/22/two-shipping-ways-of-using-docker/) 阅读原文。
***