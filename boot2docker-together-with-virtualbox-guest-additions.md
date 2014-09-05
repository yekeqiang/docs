#boot2docker 和 VirtualBox Guest Additions -- 如何把 /Users 挂载到 boot2docker


#####作者：[Matthias Kadenbach](https://twitter.com/mkadenbach)

#####译者：[Peter Zhang](https://github.com/duobei)

***

在 Mac 上使用 boot2docker，我想无缝地运行如下的命令：

```
docker run -i -t -v /Users/mattes/project1:/data/project1 ubuntu /bin/bash
```

--

用 VirtualBox Guest Additions （查看[链接](https://gist.github.com/mattes/2d0ffd027cb16571895c#file-readme-md)） 打造你私人定制的 boot2docker.iso 镜像，或下载 http://static.dockerfiles.io/boot2docker-v1.0.1-virtualbox-guest-additions-v4.3.12.iso ，保存到 ~/.boot2docker/boot2docker.iso 。

--

问题是在 box 外 boot2docker 不支持 -v /Users/mattes/project1:/data/project1 ，因为 box 不包含 [VirtualBox Guest Additions](https://www.virtualbox.org/manual/ch04.html#idp54846192) ， VirtualBox Guest Additions 短时间或不久将来内能否将它纳入 boot2docker 中尚未明朗。

> 我们将 boot2docker 做的尽可能地小而简单，基于这一点，添加与所有用例（bare HW, vbox, vagrant, hyper-v, vmware, kvm, 等）无关的工具不在计划中。该计划在将来可能会有变化，但是现在我们还没实现最初的功能。

在 5月 30 号 [SvenDowideit](https://github.com/SvenDowideit) （ boot2docker repo 拥有者） 发表了这样的 [评论](https://github.com/boot2docker/boot2docker/issues/282#issuecomment-44601104)。

尽管如此，还是有包括 VB guest additions 的 pull request，请查看 https://github.com/boot2docker/boot2docker/pull/284 。

所以，我根据上面的 pull request 中的步骤尝试创建自己定制的 boot2docker.iso 。我遇到了很多 issue 和问题，我觉得把自己尝试的步骤记成文档应该不错。

准备：以下都假设你已经安装了 boot2docker 。如果还没安装，可以根据 http://docs.docker.com/installation/mac 使用 Docker OSX 安装器安装。

如果你不是 .pkg 安装爱好者，请参阅：

- 安装 Docker: https://gist.github.com/mattes/d04e07e089ab5afd89ef

- 安装 Boot2docker: https://gist.github.com/mattes/9961a8bd5c9d53f48ce6

关于 homebrew 的一些说明：好像 [brew 正在从它的 formula 里移除 docker](https://github.com/Homebrew/homebrew/pull/30013) 。（我个人不支持这么做。）应该用 [boot2docker-cli](https://github.com/boot2docker/boot2docker-cli) 替换 [boot2docker formula](https://github.com/Homebrew/homebrew/blob/master/Library/Formula/boot2docker.rb) ，但是一个 [formula](https://github.com/Homebrew/homebrew/pull/29513) 依旧缺失。当我写下这些的时候， [docker formula](https://github.com/Homebrew/homebrew/blob/master/Library/Formula/docker.rb) 已经过期。

--

好啦，用这个 [Dockerfile](https://gist.github.com/mattes/2d0ffd027cb16571895c#file-dockerfile-tmpl) ，基本上我创建了一个新的 boot2docker.iso 。它完全基于 [steeve](https://github.com/steeve) 的 [PR](https://github.com/boot2docker/boot2docker/pull/284) ，所有的功劳都应归于他。为了使每个 boot2docker 永久挂载 /Users 都能成功，我必须添加一些额外的行。请看 [这里](https://gist.github.com/mattes/2d0ffd027cb16571895c#file-dockerfile-tmpl-L21) 。 就个人而言，我认为这是个很丑的 hack ，但此刻我找不到更好的办法了。（例如这个 /var/lib/boot2docker/bootsync.sh 文件，也许会是一个更好的选择。）

![alt](http://resource.docker.cn/boot2docker.png)

> 不幸的是，把 “Auto-mount” 设置成 “yes” 对我不起作用。我必须加上 [这些](https://gist.github.com/mattes/2d0ffd027cb16571895c#file-dockerfile-tmpl-L21) 。

建立这样的一个 Dockerfile 可以将 boot2docker.iso 保存到相应的 Docker 镜像。拷贝这样的 iso 文件到 host 系统，我经常看到：

```
# 这对我行不通
$ docker run -i -t --rm mattes/boot2docker-vbga > boot2docker.iso
```

这将返回一个空白文件或者是一个不能正常启动的 iso 的文件。我尝试用不同的 Docker 版本 和 Docker 1.0 很明显的得到这样的结果。对我来说能正常工作的是：

```
# 这个是可以的
$ docker run -i -t --rm mattes/boot2docker-vbga /bin/bash
# 在另外一个 shell 里运行
$ docker cp <Container-ID>:boot2docker.iso boot2docker.iso
```

一旦你在 host 系统上获得了崭新的 boot2docker.iso ，就在 ~/.boot2docker/boot2docker.iso 替换旧的：

```
$ boot2docker stop
$ mv ~/.boot2docker/boot2docker.iso ~/.boot2docker/boot2docker.iso.backup
$ mv boot2docker.iso ~/.boot2docker/bo
```

现在让 VirtualBox 知晓你要挂载的路径：

```
$ VBoxManage sharedfolder add boot2docker-vm -name home -hostpath /Users
```

这就完成了。我们来验证一下：

```
$ boot2docker up
$ boot2docker ssh "ls /Users"
Guest
Shared
mattes
```

接着，就可以运行：

```
docker run -i -t -v /Users/mattes/project1:/data/project1 ubuntu /bin/bash
```

--

想偷懒的话：简单的下载 http://static.dockerfiles.io/boot2docker-v1.0.1-virtualbox-guest-additions-v4.3.12.iso ， 将其移到 ~/.boot2docker/boot2docker.iso 。在做这之前，请确认先停止 boot2docker 。这儿是 [创建日志](https://gist.github.com/mattes/1eff3d69b6bc581afe03) 。

--

2014.6.20 编辑: 更新 boot2docker 到 1.0.1 版本。

***

#####这篇文章由 [Matthias Kadenbach](https://twitter.com/mkadenbach) 撰写，[Peter Zhang](https://github.com/duobei) 翻译。点击 [这里](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c) 阅读原文。

#####The article was contributed by [Matthias Kadenbach](https://twitter.com/mkadenbach) , click [here](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c) to read the original publication.
