# 理想的 docker 部署和编配工具

#### 作者：[Nick Jackson](https://twitter.com/nickjjackson_) 
#### 译者：[@Dhbehjs](http://weibo.com/u/2609282261)
***
> ##### 译者注：编配是指自动化安排、协调和管理复杂计算机系统、中间件和业务

***

首先指出，基于个人经验这个工具尚不存在，否则我肯定在使用它了。另外我现在也没有这方面的能力去独立开发这样一个工具。


考虑到目前有许多的 [docker](https://www.docker.io/) [部署](https://flynn.io/) [工具](http://deis.io/) 还在开发中，我的这个想法也许还不成熟，但我认为自己的方法还是和他们大不一样。


我决定给这个工具起名 AirDock ，以下是它的工作方式。

***

AirDock 分为三部分。第一部分是一个服务，它运行在一个 Docker 容器内部，作用在全部用户的 web 主机上。这个“服务”提供一个 API，可以用来在 docker 容器中部署应用和其他服务。 [etcd](https://github.com/coreos/etcd) 在这里用于发现服务、连接主机群、存储组配置信息和必要的参数。

> ##### 译者注：etcd是 go 语言编写的高可用的 KEY/VALUE 存储系统，主要用于分享配置和服务发现，go 版本的 zookeeper  。


第二部分是一个 CLI （ Common Language Infrastructure ，命令行界面），用于同 AirDock API 对话，显示细节和部署应用。最后一部分， AirDock 提供一个 web app 用来显示应用组的概览内容，包括参数和其他包。


这个工具不需要那些用预先确定好的语言编写的应用，它可以使用 dockerfiles 通过正确的库和依赖包来引导应用和服务。在部署时有时也需要 airdockfiles 文件，用来提供更进一步的信息，包括如何在应用组实现部署。

***

## 应用组

应用组由许多主机组成。这些主机实现负载均衡，为用户提供可靠的服务。应用组可以运行在云里或者本地硬件设备上，可由一个或几个主机组成。通过 frmation API 可以进入任何一个主机，不过这种行为受限制。

主机群可以订制个性化智能标签，这种标签允许用户部署特定的服务到特定的主机。例如，我们可以部署一个博客并且定义标签为 “ blog ” ，这就意味着这个博客将会被部署到任何一个有 “ blog ” 标签的主机上。后面才加入应用组的带有“ blog ”标签的任一主机都可以选择是否部署博客。

## 服务和应用

除了 dockerfile ，服务和应用还需要一个设置所需的包含 URI （统一资源标志符） 的 airdockfile 文件，将所需的服务暴露给外面使用，就像运行测试组件一样。

像数据库这样的服务都有自己唯一的方法来实现水平扩展，如果试图自动执行一个进程时，则会遇上难题。 airDock 通过跟构建公开有效的服务包，来简化这个过程。

用户们可以有选择地部署负载均衡器，并且当这些部署在应用组之上的时候，这些均衡器可以通过 etcd 去触发重启和虚拟主机的更新操作。

当部署改变时，app 源代码可以做为一个压缩包被发送到线下，或者用户也可以使用 github 的 [hooks](http://developer.github.com/v3/repos/hooks/) 工具部署一个已发布的版本。在 github 上每发布一个新的版本，新的容器就会同步部署一次。测试组件能够在上线前运行，从而确保设置能够如预期工作。

> #### 作者注：就我个人来说，我不选择用 git push 去部署，而是更喜欢在自己的开发环境中部署和运行应用，不必每一次变更都去提交。由于 git 需要将全部的文件进行提交并且需要管理依赖关系，这些使得 git 过于繁琐。不管怎样，稍晚会添加 git 功能。

***

## 其他注意事项

1.用户可以在任意地方重新部署一个完整的应用组，甚至是本地机器上，这样这些应用可以在一个几乎和生产环境一样的地方进行测试。

2.应用组对应用友好，所以开发者可以构建小型的模块化的 API 和 web app 。

3.部署或者添加主机时，其他的应用可能需要重新部署以便得到最新的配置信息。

***

## CLI 例子

这里是一个 AirDock 命令行工具的例子，它提供一个简单的方法去管理应用组，包括部署服务和应用。


### 登录到应用组
This logs you in to an existing AirDock formation, in this case the “my-massive-stack” formation. If this is the first time running this command, the CLI will request an IP address of an airdock host within that formation.

你将登录到一个已经存在的 AirDock 应用组，在这个例子里这个应用组叫 “my-massive-stack” 。如果在第一时间运行命令，  CLI 工具需要一个 Airdock 主机的 IP 地址。

```
$ airdock my-massive-stack
user: foo
pass: ****************
```

### 显示应用组信息

登入 AirDock 后，你可以在当前应用组内运行命令。这个例子里我们能够看到全部应用组的概览，如果需要，你可以继续看到更详细的信息。

```
$ airdock show
 
IP          CONTAINERS                         TAGS
10.0.0.1    etcd, airdock, loadbalancer1,      main
10.0.0.2    etcd, airdock, www1, mongo1        web, dbs
10.0.0.3    etcd, airdock, www2, mongo2        web, dbs
10.0.0.4    etcd, airdock, www3, mongo3        web, dbs
```

### 部署 App

假设一个 dockerfile 文件和 airdock 文件连同 app 源代码都在当前目录下，下面的命令可以将一个 app 部署和扩展到任意一个含有 web 标签的主机上。

```
$ airdock deploy --scale web
```

你可以指定希望部署到有 “web” 标签的主机数量。在这个例子里，它被部署到一台主机上。

```
$ airdock deploy --scale web:1
```

或者按照多个标签进行部署。

```
$ airdock deploy --scale web:frontend:1 
```

### 部署服务

下面这个命令是用来构建一个 mongodb 数据库包，以下指令里， -name 标签允许你去特别指定一个唯一的句柄来操作新形成的服务，并且这个名字将被传递到剩下的应用组中。 -scale 标签可以具体规定部署到哪个标签的主机上。

```
$ airdock service mongodb --name mongo --scale dbs:3
```

***

这就是我认为理想的 docker 部署和编配工具的基本大纲，如果你有什么想法，请和我联系。

***
##### 这篇文章由 [Nick Jackson](https://twitter.com/nickjjackson_) 发表，[@Dhbehjs](http://weibo.com/u/2609282261) 翻译。点击 [这里](https://medium.com/geek-out/bc32e557f4ed) 可查阅原文。

##### The article was contributed by [Nick Jackson](https://twitter.com/nickjjackson_), click [here](https://medium.com/geek-out/bc32e557f4ed) to read the original publication.
