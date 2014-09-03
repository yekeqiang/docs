***

# 使用 Boot2Docker 和 NSEnter

***

##### 作者：[rhuss](https://github.com/rhuss)
##### 译者：[moonatcs](http://blog.yege.me/)

***
使用 [NSEnter](https://github.com/jpetazzo/nsenter)  连接到运行的容器，是一个比较好的方法， 
本文介绍了一个同时使用  `nsenter` 和  [Boot2Docker](https://github.com/boot2docker/boot2docker) 的脚本。

随着经验的不断积累，新的模式以及反模式的应用，使 Docker 存在着一些小问题。
在 image 内部使用 SSH daemon 进行调试、备份以及排查故障，就是这些反模式中的一个。
Jérôme Petazzoni 的[文章](https://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/)对此作了很好的解释。
另外，文章还为通常使用 SSH 的一些用例提供了适当的解决方案。

然而，我一直还是不可抗拒的去登录到容器内部，这样做只是为了检查一下环境。
庆幸的是，Jérôme 提供了一个满足这个需求的解决办法：[nsenter](https://github.com/jpetazzo/nsenter) 。它允许你进入(**enter**)容器的命名空间(**n**ame**s**pace)。
在 GitHub Page 上，你可以找到在 Linux 主机上安装和使用 `nsenter` 的方法。

如果你要在 装有 Boot2Docker 的 OS X 上使用它，
你需要登录到 运行 Docker daemon 的 VM 上，然后再连接到运行的容器上。

就像 [NSenter README](https://github.com/jpetazzo/nsenter#docker-enter-with-boot2docker) 里描述的那样，
可以用简单的别名，透明的实现它:

 	docker-enter() {
      boot2docker ssh '[ -f /var/lib/boot2docker/nsenter ] || docker run --rm -v /var/lib/boot2docker/:/target jpetazzo/nsenter'
      boot2docker ssh -t sudo /var/lib/boot2docker/docker-enter "$@"
    }

更好的方法是用 bash 实现它，
只需要 [docker-enter](https://gist.github.com/rhuss/a8a40bd143001fd5c83c#file-docker-enter) 中的 shell 脚本安装到 OS X 上的 path 。 
它需要的参数包括：容器的 id 或容器的 name 以及 要在容器中执行的命令。 
如果 boot2docker VM 上没有安装 nsenter ， 这个脚本会自动安装 `nsenter` ：

	10:20 [~] $ docker ps -q
    5bf8a161cceb
    
    10:20 [~] $ docker-enter 5bf8a161cceb bash
    
    Unable to find image 'jpetazzo/nsenter' locally
    Pulling repository jpetazzo/nsenter
    Installing nsenter to /target
    Installing docker-enter to /target
    
    root@5bf8a161cceb:/#

如果想要 bash completion (bash自动补齐功能)，可以将安装 bash completion(bash自动补齐功能) 的脚本  
[docker-enter_commands](https://gist.github.com/rhuss/a8a40bd143001fd5c83c#file-docker-enter_commands)
( 需要 [Docker's bash completion](https://github.com/docker/docker/blob/master/contrib/completion/bash/docker) 支持) 
添加到 `~/.bash_completion_scripts/` 这个路径( completion scripts 所在的路径 )。
这样就会自动补全 `docker-enter` 的容器 name 和容器 id 参数。

*P.S. After writing this post,
 I’ve found out, that this topic has been already covered in another [blog post](http://blog.sequenceiq.com/blog/2014/07/05/docker-debug-with-nsenter-on-boot2docker/) previously by Lajos Papp. 
 That’s also where the shell function definition in the `nsenter` README originates from.
 Give credit to whom it’s due.*
 
 ***

#####这篇文章由 [rhuss](https://github.com/rhuss) 撰写，[moonatcs](http://blog.yege.me) 翻译。点击 [这里](http://ro14nd.de/NSEnter-with-Boot2Docker/) 阅读原文。

#####The article was contributed by [rhuss](https://github.com/rhuss) , click [here](http://ro14nd.de/NSEnter-with-Boot2Docker/) to read the original publication.
 
