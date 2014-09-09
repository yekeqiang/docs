#用 POWERSHELL DSC 配置 DOCKER

#####作者：[TechNet](https://twitter.com/MS_ITPro)

#####译者：[Stone Feng](http://blog.csdn.net/stonefeng)
***
> #####作者注：这篇博文的作者 Andrew Weiss ，是一位才华横溢的顾问，来自微软咨询服务部门。Andrew 一直致力于通过 PowerShell DSC 来提高管理 Docker 容器能力。感谢 Andrew 在 [Building Clouds](http://aka.ms/buildingclouds) 分享他的文章！

***
[Windows PowerShell Desired State Configuration for Linux](https://github.com/MSFTOSSMgmt/WPSDSCLinux) 首个预览构建的发布，带来了丰富的、跨平台的配置管理可能性。

本文提供了将 PowerShell DSC 和轻量、基于容器的平台 Docker 整合的、在微软云里创建可重用的 dev/test 环境的指南。更具体地说，你将使用 PlatypusTS 来 Docker 化一个样板 web 应用。 PlatypusTS 是一个在 TypeScript 上构建的 web 和移动框架。

###用代码管理构架

>译者注：标题原文为"Infrastructure-as-Code"

在云里构建应用时，重要的是为开发者和系统管理员均提供一组常见的工具和框架。 DevOps 介绍了 “infrastructure-as-code”（用代码管理构架——译者注）的概念，IT 专业人员可以与开发者们使用同样的技术和语言管理底层构架。

###什么是 Docker？
Docker 是一个构建在 LXC 之上的开源平台，是 Linux 容器，可以用来开发和部署应用。通过使用镜像和容器，开发者和 IT 专业人员可以容易地在内部和云端部署标准化的开发环境。 Docker 与传统的虚拟机的区别是：Docker 容器不依赖于庞大的虚拟操作系统，可以显著地减少空间占用。

如果你想了解更多内容， Docker 团队提供了很棒的 [交互式演示](http://www.docker.com/tryit/) 。

###DSC for Linux 入门

在 Linux 主机上安装和配置 DSC 的指南可以在 Kristopher Bash 较早发布的文章里找到，正值五月份 PowerShell DSC for Linux 被披露和发布的时候。

请参考该文章中的指导步骤，确保你的 Linux 主机已经为 DSC 配置好，然后才能依照本文中的其它指导步骤。我使用一个部署在微软云上的 Ubuntu 14.04 LTS 虚拟机来完成演练。

###前提条件

- 通过 SSH 和 TCP 端口 80 、 5985 和 5986 的 Linux 主机连接
- 运行 PowerShell v4.0 或更高版本的 Windows 主机
- Git (http://git-scm.com/)
- SSH客户端 (PuTTy)

###配置管理工作站

在开始使用 Windows PowerShell 控制台之前，我会进入我的工作站上一个合适的目录，拷贝我需要的 Docker DSC 配置文件。执行下面的代码来创建一个叫做 DockerClientDSC 的、带有我的配置源码的文件夹：

`git clone https://github.com/anweiss/DockerClientDSC`

在 DockerClientDSC 文件夹里有个脚本 DockerClient.ps1 ，脚本里包含了一个叫做 DockerClient 的 DSC 配置。我将用它来确保我云端的目标虚拟机是按照 Docker 主机配置，并且包含了我将使用 dev/test 的 Docker 镜像。在继续之前，我还将 DockerClientDSC 文件夹设置为我的工作目录。

###用 DSC 安装 Docker

使用 DockerClient 之前，我会创建一个变量来存储 Linux 虚拟机的主机名：

`$hostname = "platypus-dev01.contoso.com"`

 
然后我将配置装入当前会话：

`. .\DockerClient.ps1`

我现在可以带主机名参数执行 DockerClient 配置，创建一个新的 DockerClient 子目录，里面包含了与目标节点对应的 .mof 文件：

`DockerClient -Hostname $hostname`

 
为了使产生的配置生效，我需要取得目标主机的凭据（注意在写这篇文章的时候，DSC for Linux 仅支持通过“root”用户连接）：

`$cred = Get-Credential -UserName "root"`

 
还要配置一组选项，我才能成功建立一个连到目标的远程 PowerShell 会话：

```
$options = New-CimSession -UseSsl -SkipCACheck -SkipCNCheck -SkipRevocationCheck
```

我可以使用这些选项来初始化一个到 Linux 虚拟机的 CIM 会话：

```
$session = New-CimSession -Credential $cred -ComputerName $hostname -Port 5986 -Authentication basic -SessionOption $options -OperationTimeoutSec 600
```

最后，应用我的配置：

```
Start-DscConfiguration -CimSession $session -Path .\DockerClient -Verbose -Wait -Force
```
 
在命令正常结束之后，我在 Linux 虚拟机上安装并配置好了 Docker 。现在让我们用 DockerClient 配置脚本来下载一些 Docker 镜像。

###用 DSC 下载 Docker 镜像
我们接下来更新配置来使 Docker 主机包含一对 Docker 镜像。我们将下载 MongoDB 镜像和附带的包含一个预先打包的、基于 PlatypusTS 框架的样板 web 应用。可以通过下面所示的镜像参数做到：

```
DockerClient -Hostname $hostname -Image @("mongo:latest", "anweiss/docker-platynem:latest")
```
 
建立 CIM 会话之后，我们可以简单地重新运行 Start-DscConfiguration 命令代码：

```
Start-DscConfiguration -CimSession $session -Path .\DockerClient -Verbose -Wait -Force
```
 
现在我在主机上下载并安装好 MongoDB 镜像。

###用 DSC 运行链接容器

我们还可以用 DockerClient DSC 配置创建新的带有一个 MongoDB 实例和样板 web 应用的链接容器。既然我们的 web 应用依赖于一个 MongoDB 实例，我就先创建一个 MongoDB 容器。我将建一个散列表来存储容器使用的参数。

`$dbContainer = @{Name="db"; Image="mongo:latest"}`

然后我将用这个散列表来创建一个新配置。

`DockerClient -Hostname $hostname -Container $dbContainer`

 
如我们此前做的那样，重新运行 Start-DscConfiguration 命令代码：

```
Start-DscConfiguration -CimSession $session -Path .\DockerClient -Verbose -Wait -Force
```

太棒了！现在我们应该有了一个在容器中运行 MongoDB 的实例。我们用 PuTTy SSH 进入虚拟机并取得容器的 IP 地址。我们在 SSH 会话里执行下面的 Docker 命令：

```
sudo docker inspect --format="{{ .NetworkSettings.IPAddress }}" db 
```

我将拷贝显示出的IP地址（你的可能会有所不同）。

现在回到 PowerShell 控制台，我们将创建一个变量来存放 MongoDB 容器的 IP 地址，并且放在 web 容器的环境变量里。我们还将建另一个散列表来存放 web 容器的设置。

```
$mongoIP = "172.17.0.28"

$envVars = @("MONGOHQ_URL=mongodb://$mongoIP/platynem-dev","PORT=80")

$webContainer = @{Name="web";Image="anweiss/docker-platynem:latest";Port="80:80";Env=$envVars;Link="db"}

``` 

再一次，我们用这个散列表创建一个新的配置。

`DockerClient -Hostname $hostname -Container $webContainer`

让我们应用这个配置。

```
Start-DscConfiguration -CimSession $session -Path .\DockerClient -Verbose -Wait -Force
```
 

我们的 web 应用现在应该正运行在 80 端口。当我启动浏览器，并访问虚拟机的 URL ，应该能看到样板 web 应用在 web 容器中运行，并链接到正运行 MongoDB 的容器——一切都是用 PowerShell DSC 配置的。

注意：同样的技术可以很好地应用在开发环境的 Docker 容器上，不需要修改任何步骤。

非常感谢 Andrew 这么棒的文章。欢迎关注 [Building Clouds](https://twitter.com/Building_Clouds) !

***

#####这篇文章由 [TechNet](https://twitter.com/MS_ITPro) 撰写，点击 [这里](http://blogs.technet.com/b/privatecloud/archive/2014/07/17/configuring-docker-on-azure-with-powershell-dsc.aspx) 阅读原文。[Stone Feng](http://blog.csdn.net/stonefeng) 翻译了此文，如有任何问题，请点击 [这里](http://blog.csdn.net/stonefeng/article/details/38356259) 与译者联系。

#####The article was contributed by [TechNet](https://twitter.com/MS_ITPro) , click [here](http://blogs.technet.com/b/privatecloud/archive/2014/07/17/configuring-docker-on-azure-with-powershell-dsc.aspx) to read the original publication.

