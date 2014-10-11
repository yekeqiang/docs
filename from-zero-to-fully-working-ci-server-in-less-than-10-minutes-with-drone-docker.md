#用 Drone 和 Docker 10 分钟搭建全功能的 CI 服务器

---

什么是 [Drone](https://github.com/drone/drone) ？ 官方的解释：“ Drone 是基于 Docker 构建的持续集成平台”。

什么是 [Docker](http://docker.io) ？官方的解释：“ Docker 是一个开源项目，它是一个容易创建的轻量级、易移植和对任何程序都是全功能的容器。”

10 分钟，你只需要 10 分钟。事实上，10 分钟外还需要额外的时间让你的配置工作起来。

10 分钟和让一台 [Jenkins](http://jenkins-ci.org) 服务器启动和运行所需的时间差了个数量级。我从来都不是 [Jenkins](http://jenkins-ci.org) 的粉丝，或许永远都不会是。还是让我们在其它文章中讨论这个问题吧。

##需要的条件

我们假设你有一台可以访问的 64 位 Ubuntu 13.04 版本服务器。在 Digital Ocean (我非常喜欢 Digital Ocean，请使用我的[推荐链接](https://www.digitalocean.com/?refcode=9b3537dd733f)，谢谢～) 你可以使用预先安装好 Docker 版本的 VPS ，在写这篇文章的时候 Docker 的版本是 0.8 。

理论上来说 64 位 Ubuntu 12.04 服务器也可以运行 Drone 。

你的应用需要托管在 [Github](http://github.com) ，貌似 [Bitbucket](http://bitbucket.org) 很快就会支持。目前只支持 [Github](http://github.com) 对我来说没什么问题，我喜欢 [Github](http://github.com) 。

###第 0 步：安装 Docker (可选，如果你已经安装了 Docker 可以跳过此步)

在你的 Ubuntu 机器上执行：


```
curl -s https://get.docker.io/ubuntu/ | sudo sh
```

你可以使用下面的命令验证安装：

```
sudo docker run -i -t ubuntu /bin/bash
```


###第 1 步：安装 Drone

Drone 安装非常简单！

你要做的就是：

```
wget http://downloads.drone.io/latest/drone.deb
sudo dpkg -i drone.deb
```

在 README 中有 “sudo start drone” 启动的部分，但是安装程序已经为我启动了 Drone 。

安装完成后，用浏览器打开 http://my-server-ip-or-addr:80/install ，根据向导创建账号。一旦登陆，请保持浏览器一直打开。

###第 2 步：在 Github 注册一个新的应用

现在你已经有一个 Drone 的服务在运行了，但是我们必须配置一个 Github 的应用使得 Drone 在 Github 的 Pull 或 Push 后能够 Build 你的代码。[在这里注册一个新的程序](https://github.com/settings/applications/new)。

* 选一个名字，可以叫“ my Drone server ” 或者其它任何对你有意义的名字
* 设定主页地址是 http://my-server-ip-or-addr/
* 描述就像名字一样，对你有意义就可以
* Github OAuth 认证的回调地址是 http://my-server-ip-or-addr/auth/login/github

###第 3 步：在 Drone 中设置 Github 的 Client ID 和 Client Secret 

在 Github 注册完成新应用后，你会得到 Client ID 和 Client Secret，把它们拷贝到 http://my-server-ip-or-addr/account/admin/settings 页面的 "GitHub OAuth Consumer Key and Secret" 部分。

###第 4 步：链接你的项目

点击  “New Repository”  按钮页面会转跳到 http://my-server-ip-or-addr/new/github.com 这里，点击 “Link Now” 按钮，一旦接受了 Drone 的 Repository Setup 页的，你会得到 Github 账户内所有代码仓库的信息(账户名+仓库名)。

Boom！你的项目几乎已经可以进行集成测试了，你现在只需要完成最后一步...

###第 5 步：配置你的 .drone.yml 文件

.drone.yml 文件是 Drone 用来配置编译步骤、各种服务和通知等的配置文件。

下面是一个简单的 Ruby on Rails 项目的配置文件：

```
image: ruby2.0.0
script:
  - cp config/database.drone.yml config/database.yml
  - bundle install
  - psql -c 'create database test;' -U postgres -h 127.0.0.1
  - bundle exec rake db:schema:load
  - bundle exec rspec spec
services:
  - postgres
notify:
  email:
    recipients:
      - email@example.com
```

你一定很疑惑 database.drone.yml 是什么，[请看这个gist](https://gist.github.com/jipiboily/0cad2550be91f5c9b5d)

这里有很多 Go、Python、Haskell、PHP、Scala、Node 的镜像。Drone 为我们提供了这些[官方镜像](https://github.com/drone/drone#images)，不过你也可以使用其它的镜像。

[这里还有很多服务的镜像](https://github.com/drone/drone#databases)

可以在 [README](https://github.com/drone/drone#builds) 和 [drone.io](http://drone.io/) 的[官方文档](http://docs.drone.io/)中找到如何设置 .drone.yml 文件的说明。

当你完成这些后，commit，push 和 watch 你的代码时 Drone 就会 Build 。反复重写你的 .drone.yml 直到没有问题。 

当第一次 Build 代码的时候会花上一些时间。

##特别提示！

下面是一些由 Drone 提供的、你可能需要的功能：

* 邮件通知( SMTP 设置在  http://my-server-ip-or-addr/account/admin/settings 底部 )
* SSL 设置( SSL 设置在 http://my-server-ip-or-addr/account/admin/settings 顶部 )
* HipChat 通知
* 可持续部署

##结论


就是这些，让你可以部署一个全功能的 CI 服务器。是不是应该使用它替代 Jenkins、Codeship 或者其它的 CI 服务么？这完全取决于你。这个项目是一周前发布的，我还不能确定是否把它用在我工作的项目中，但是我会保持关注。

让我知道你是否喜欢 Drone！

另外：如果你喜欢 Docker 和 类似 Heroku 的 PaaS 服务，你可能会感兴趣我的另一篇关于 [Dokku 安装](http://jipiboily.com/2013/install-dokku-postgresql-with-docker-for-your-rails-app-or-whatever-else-almost) 的文章。

---
#####这篇文章由 [Jean Philippe](https://github.com/jipiboily) 在2014年2月13日发布，点击 [这里](http://jipiboily.com/2014/from-zero-to-fully-working-ci-server-in-less-than-10-minutes-with-drone-docker) 可以阅读原文。

#####These article was contributed by [Jean Philippe](https://github.com/jipiboily), click [here](http://jipiboily.com/2014/from-zero-to-fully-working-ci-server-in-less-than-10-minutes-with-drone-docker) to read the original publication.
