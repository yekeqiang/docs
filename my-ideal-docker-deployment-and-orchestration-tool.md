## My ideal docker deployment and orchestration tool
## 理想中的 docker 部署和编配工具（编配：描述复杂计算机系统、中间件和业务的自动化的安排、协调和管理）

***

I should probably start by pointing out that to my knowledge this tool doesn’t exist yet and while I would definitely use this, I’m not capable of creating it on my own.

首先要指出的是根据我个人的经验这个工具还不存，否则我肯定在使用它了。另外我现在也没有这方面的能力去自己开发这样一个工具。

There are many [docker](https://www.docker.io/) [deployment](https://flynn.io/) [tools](http://deis.io/) in development, so maybe this idea is premature, but I believe there are some key differences in the way this works that I prefer.

有许多的 docker 部署工具在开发中，那么也许这个主意还不成熟，但是我相信会有一些关键的差异点在这个方法里，这个工作也是我喜爱的。

I’ve decided to call this imaginary tool AirDock and here is how I think it should work.

我决定给这个工具起名 airDock ，并且以下是我所思考的它的工作方式

***

AirDock is in three parts. The first part is a service which runs inside a Docker container on all of the users web hosts. This “service” exposes an API which can be used to to deploy applications and other services within containers. [Etcd](https://github.com/coreos/etcd) is used here for service discovery, as well as linking hosts, storing formation settings and useful metrics.

AirDock 分为三部分。第一部分是一个服务，它运行在一个 Docker 容器内部，作用在全部用户的 web 主机上。这个“服务”提供一个 API，可以用来在 docker 容器中部署应用和其他服务。Etcd在这里用于服务发现，连接主机群，存储组配置信息和必要的参数。Etcd（Etcd 是 go 语言编写的高可用的 KEY/VALUE 存储系统，主要用于分享配置和服务发现，go 版本的 zookeeper ）

The second part is a CLI which talks to the AirDock API to show details and make deployments. Finally, AirDock provides a web app with a formation overview including metrics and other gems.

第二部分是一个 CLI（Common Language Infrastructure,命令行界面），用于同 airDock API 对话，显示细节和部署应用。最后一部分是，a irDock 提供一个 web app 用来显示应用组的概览内容包括参数和其他包。
The tool does not require applications to be written in pre determined languages as it uses dockerfiles to boot apps and services with the correct libraries and dependencies. Airdockfiles may also be required when deploying, as these provide further information on how the deployment should occur across the formation.

工具不需要那些用预先确定好的语言编写的应用来支持，它可以使用 dockerfiles 来引导应用和服务使其有正确的库和依赖包。在部署时有时也需要 airdockfiles 文件，用来提供更进一步的信息，包括部署是怎样发生的通过应用组。

***

### Formations
### 应用组
Formations are a group of hosts that work together to load balance and ensure reliability of the users entire stack. A formation can exist on the cloud, or on a local dev machine and can be made up by several hosts, or just one. Access to the formation API is through any of the hosts, but of course this can be restricted.

应用组是有许多主机组成，它们一起工作来实现负载均衡和为用户提供可靠的服务。一个应用组可以运行云里或者在一个本地的硬件设备上并且可以由一个或几个主机组成。通过 frmation API 可以进入任何一个主机，但是这种行为是受限制的。

Hosts can be assigned custom smart tags which allow the user to deploy specific services, to specific hosts. For example, we could deploy a blog to the tag “blogs”, meaning the blog will be deployed to any hosts assigned with that tag. Any hosts with the tag “blogs” added to the formation at a later date, will be given the choice of having the blog deployed there also.

主机群可以订制客户化的智能标签，这种标签允许用户部署特定的服务到特定的主机。例如，我们可以部署一个博客并且定义标签为 “blog” ，这就意味着这个博客将会被部署到任何一个有 “blog” 标签的主机上。任何一个有 “blog” 标签的主机将在后面的一个时刻加入到 formaiton上，这些主机也将可以有一个选项是否在其上部署一个博客。

### Services and Applications
### 服务和应用
Along with the Dockerfile, services and application should have an airdockfile containing URI’s to setup, services that it exposes and those that it requires, as well as test-suites to run etc.

同 dockerfile 一起，服务和应用应该有一个包含 [URI](http://zh.wikipedia.org/zh-cn/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6) 的设置的 airdockfile 文件，将所需的服务暴露给外面使用，就像运行测试组件一样。

Services such as databases each have their own unique way of scaling horizontally, which provides a challenge when trying to automate the process. AirDock would simplify this with service build packs which could be made publicly available.

服务就像数据库服务一样，每一个服务有他自己唯一的方法来实现水平扩展，当你试图自动执行一个进程的时候，这个方法会是一个挑战。airDock 通过跟构建公开的有效的服务包，来简化这个过程。


Users can deploy load balancers of their choosing and these can be hooked into etcd to trigger restarts and vhost changes when deployments are made across the formation.

用户们可以有选择的部署负载均衡器，而且当这些部署的构造在应用组之上的时候，这些均衡器可以通过 Etcd 去触发重启和虚拟主机的更新操作。

When deploying changes, app source code can be sent down the line as a tarball, or users can make use of github hooks to deploy on a release. New containers will spin up every time a deployment is made with the newly deployed code in place. Test suites can be run before going live ensuring the setup is working as expected.

当部署改变时，app 源代码可以做为一个压缩包被发送到线下，或者用户也可以使用 github 的 [hooks](http://developer.github.com/v3/repos/hooks/) 工具部署一个已发布的版本。在 github 上每发布一个新的版本，新的容器就会同步部署一次。测试组件能够在现场确定安装在工作作为期望之前运行。

>On a personal level, I’ve chosen not to use git push to deploy because I would like to be able deploy and run tests inside my dev formation without having to commit every change. Git also requires all files to be checked in and managing dependencies in git can get tedious. However, git functionality could always be added later.

>就我个人来说，我不选择用 git push 去部署，因为我更喜欢在我自己的开发环境中部署和运行应用，而不必每一次变更都去提交。同时 git 也需要将全部的文件进行提交并且需要管理依赖关系，这些使得 git 过于繁琐。不管怎样 git 从功能上来说可以稍晚一些将这个功能添加到工具中。

***

### Other things to note.
### 其他需要注意的事情。
1.Users can redeploy an entire formation anywhere, even to a local machine so apps can be tested in an environment almost identical to production.

1.用户可以重复部署一个完整的应用组在任何地方，甚至在一个本地机器上，这样这些应用可以在一个几乎和生产环境一样的地方进行测试。

2.Formations play nice with several applications, so that developers can build small modular API’s and web apps.

2.应用组对应用友好，所以开发者可以构建小型的模块化的 API 和 web app。

3.Upon deploys or adding hosts, other applications may need to re-deploy in order to get the latest configurations.
部署或者添加主机时，其他的应用可能重新部署以便得到最新的配置信息。

***

### An example CLI
### CLI例子
Here we have an example command line interface for AirDock. It provides a simple way to manage the formation, including deployment of services and applications.

这里是一个 airDock 命令行工具的例子，它提供一个简单的方法去管理应用组，包括部署服务和应用。

### Logging into a formation
### 登录到应用组
This logs you in to an existing AirDock formation, in this case the “my-massive-stack” formation. If this is the first time running this command, the CLI will request an IP address of an airdock host within that formation.

你将登录到一个已经存在的 airDock 应用组，在这里例子里这个应用组叫 “my-massive-stack” 。如果在第一时间运行命令，CLI 工具需要一个 Airdock 主机的IP地址。

```
$ airdock my-massive-stack
user: foo
pass: ****************
```

### Showing details about the formation
### 显示应用组信息。
Once logged in, you can run commands in the context of that formation. Here we would be able to see an overview of the entire formation and drill down to more specific information if we wanted.

当登入 airDock 后，你可以在当前应用组内运行命令。这个例子里我们能够看到全部应用组的概览，如果需要，你可以继续看到更详细的信息。

```
$ airdock show
 
IP          CONTAINERS                         TAGS
10.0.0.1    etcd, airdock, loadbalancer1,      main
10.0.0.2    etcd, airdock, www1, mongo1        web, dbs
10.0.0.3    etcd, airdock, www2, mongo2        web, dbs
10.0.0.4    etcd, airdock, www3, mongo3        web, dbs
```

### Deploying an App
### 部署一个app
Assuming both a dockerfile and airdock file are present in the current directory along with any app source code, the following command deploys and scales that app to any hosts with the tag named “web”

假设一个 dockerfile 文件和 airdock 文件连同 app 源代码在当前目录存在。下面的命令可以将一个 app 部署和扩展到任意一个含有 web 标签的主机上。

```
$ airdock deploy --scale web
```

You can also specify how many hosts tagged with “web” you want it deployed to. In this case, its only to one host.

你可以特别指出希望部署到多少个标有 “web” 的主机上。在这个例子里，它被部署到一台主机。

```
$ airdock deploy --scale web:1
```

Or specify multiple tags such as “web”, “frontend” and how many of those to deploy to.
或者可以按照多个标签进行部署。

```
$ airdock deploy --scale web:frontend:1 
```

### Deploying a service
### 部署服务
The following command looks for a mongodb build pack, and follows the instructions. The —name flag allows you to specify a unique handle for the newly formed service and this name will be passed to the rest of formation. The —scale flag allows you to specify tags like above to deploy to.

下面这个命令是用来构建一个 mongodb 数据库包，以下指令里，-name 标签允许你去特别指定一个唯一的句柄来操作新形成的服务，并且这个名字将被传递到剩下的应用组中。-scale 标签可以具体规定部署到哪个标签的主机上。

```
$ airdock service mongodb --name mongo --scale dbs:3
```

***

Well there you have it, a basic outline of my ideal docker deployment and orchestration tool. If you have any thoughts, please get in touch.

这就是我理想的 d ocker 部署和编配工具的基本大纲，如果你有什么想法，请和我联系。
