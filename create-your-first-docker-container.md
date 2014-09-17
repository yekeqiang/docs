#创建你的首个 Docker 容器

#####作者：[James Coyle](https://twitter.com/Jamescoyle_uk)

#####译者：[xanpeng](https://github.com/xanpeng)

***

在诸多支持跨操作系统的虚拟化软件中， Docker 很可能是最为简单易用的。尽管用 Docker 之前你仍需事先安装操作系统，但之后一切变得简单，你可以从 Docker 社区直接下载可用的模版或镜像。

如果你的系统中还没有安装好 Docker，请参考文章“ [在 Ubuntu 14.04 上安装 Docker](http://www.jamescoyle.net/how-to/1499-installing-docker-on-ubuntu-14-04) ”。

有一系列命令可用来管理 Docker 容器和镜像。我们首先来看本地 Docker 库中是否已有一些镜像。

```text
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
```

使用 `docker images` 命令列出可用来创建 Docker 容器的本地镜像。上面命令的输出显示本地还没有任何镜像，因此我们先从官方 Docker 镜像库中下载一个镜像。

当然我们必须指定待下载镜像的名称。目前官方 Docker 镜像库中有数以千计的可用镜像，可以通过相应的 `docker` 命令下载。我们先用搜索命令找一个镜像。

```text
$ docker search ubuntu
```

这条命令会列出所有名称中包含单词“ ubuntu ”的镜像。这个列表会有点大，因为如你所想，不仅很多标准操作系统镜像名称包含“ ubuntu ”，很多特定应用镜像、以及用户上传的镜像名称中也包含“ ubuntu ”。

我们来下载标准的 ubuntu 14.04 镜像：

```text
$ docker pull ubuntu:14.04
Pulling repository ubuntu
ad892dd21d60: Download complete
511136ea3c5a: Download complete
e465fff03bce: Download complete
23f361102fae: Download complete
9db365ecbcbb: Download complete
```

你可以验证这条命令是否下载了 ubuntu 14.04 的镜像文件，验证方法是使用前面提及的 `docker images` 命令。请注意下面验证结果中的镜像 ID ，我们后面可以通过这个 ID 创建容器。

```text
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04               ad892dd21d60        11 days ago         275.5 MB
```

下一步是创建容器并做相关操作。创建一个 Docker 容器的方法是使用 `docker run` 命令，该命令通过参数指定 Docker 容器的某些属性，同时也通过参数指定 Docker 容器运行的程序。我们将做这些操作：创建 Docker 容器，指定容器运行一个 bash 会话，然后我们通过 bash 登入容器并修改容器属性，最后将容器保存为一个镜像，以备后续使用。

通过 `docker run` 创建容器并指定运行 bash 后， Docker 呈现给我们一个 bash 会话，我们据此修改 Docker 容器。在下面的命令中使用你环境中实际的镜像 ID 替换我这里的 ad892dd21d60。

```text
$ docker run -i -t ad892dd21d60  /bin/bash
root@3a09b2588478:/#
```

现在你已经通过镜像 ad892dd21d60 创建了一个 Docker 容器，并在其中运行了 bash 。输入 `exit` 结束当前 bash 会话后容器会停止运行，不过对应的容器仍位于本地 Docker 环境中，可继续使用。

通过 `docker ps` 命令查看本地 Docker 环境中存在多少容器。

```text
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
f4b0d7285fec        ubuntu:14.04        /bin/bash           8 minutes ago       Exit 0                                  hungry_thompson
8ae64c0faa34        ubuntu:14.04        /bin/bash           10 minutes ago      Exit 0                                  jovial_hawking
3a09b2588478        ubuntu:14.04        /bin/bash           14 minutes ago      Exit 130   
```

上面的输出显示有 3 个容器正在运行，容器 ID 显示在列表左侧。我们可以重新进入容器，不过需要先启动容器。这里使用容器 3a09b2588478 作为示例，你操作时需要使用自己的容器 ID 。

```text
$ docker start 3a09b2588478
```

现在可以通过 `docker attach` 关联容器，并启动一个终端以支持登入操作。

```text
$ docker attach 3a09b2588478
```

现在你已重新进入容器。这里我们只演示一些简单的修改，比如只通过 `apt-get` 升级容器中的 ubuntu 14.04 系统安装的软件，然后退出容器。在实际环境中，你可能会安装一个应用程序，或者修改一些服务设定（比如 LDAP SSH 登录配置等）。

```text
$ apt-get update
$ apt-get upgrade
$ exit
```

我们示例的最后一步是将容器保存为一个新的镜像，从而后续可被用于创建新的容器。你需要指定容器 ID 以及待创建镜像的名称。如果使用已有的镜像名称表示将覆盖已有的镜像。

```text
$ docker commit 3a09b2588478 ubuntu:14.04
b2391f1efa6db419fad0271efc591be11d0a6d7f645c17487ef3d06ec54c6489
```

以上就是示例内容。回顾一下，你下载了一个 Docker 镜像，并据此创建了一个 Docker 容器，然后对容器做出修改，并保存了这些修改以备后续使用。当然还有很多其他的 Docker 使用方法，这里不一一枚举。我希望本文能促进你对 Docker 工作流程的基本理解。

***

这篇文章由 [James Coyle](https://twitter.com/Jamescoyle_uk) 撰写，点击 [这里]((http://www.jamescoyle.net/how-to/1503-create-your-first-docker-container)) 阅读原文。 [xanpeng](https://github.com/xanpeng) 翻译了本文，你可以通过 [撰写邮件](mailto:xanpeng@gmail.com) 与他联系。

The article was contributed by [James Coyle](https://twitter.com/Jamescoyle_uk) , click [here]((http://www.jamescoyle.net/how-to/1503-create-your-first-docker-container)) to read the original publication.
