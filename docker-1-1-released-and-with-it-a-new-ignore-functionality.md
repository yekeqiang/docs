
#Docker 1.1 新增 "ignore" 功能

***

作者：[ALEX WILLIAMS](http://thenewstack.io/docker-1-1-released-and-with-it-a-new-ignore-functionality/)

译者：[moonatcs](https://code.csdn.net/moonatcs)

***

![](http://thenewstack.io/wp-content/uploads/2014/04/homepage-docker-logo.png)

在今年6月份 Docker 1.0 版本被发布之前， dotCloud 的 CEO 及创始人 Solomon Hykes 就曾经对我说，
在 Docker 1.1 版本中将增加 .dockerignore 功能。
目前，[Docker 1.1 版本](http://blog.docker.com/2014/07/announcing-docker-1-1/)已经被发布，
Solomon Hykes 所说的话也被验证成真。
现在，开发人员可以在 `Dockerfile` 旁边创建一个 `.dockerignore` 文件，
当给 docker daemon 发送构建上下文的命令时，
Docker 就会忽略 `.dockerignore` 中列出的具体文件和路径。
具体情况，请看[详细示例](https://github.com/docker/docker/blob/master/.dockerignore)。
其他方面的更新包括：

* commit提交一个运行的容器时，该容器会被暂停
* 对容器的日志文件，开发人员可以从指定的位置进行跟踪
* tar 归档文件可以作为 `docker build` 的上下文

我们写了很多关于 Docker 的文章。当然还有其他的主题，但是对轻量级容器技术的欲望几乎是贪得无厌。
在初期，开发人员在创建应用时所具有的经验，让它表现的很好。
这是预料之中的事情，但是那些在兴奋之中常常被忽视的问题，也会被越来越多的人关注。

无论是开发人员之间的交流还是在 GitHub 上关于当前项目的评论，都提出了一些问题，特别是
上传到 Docker 开发环境中的文件如何进行管理以及对生产环境中容器进行管理的复杂性。



[Space Monkey](https://www.spacemonkey.com/) 的 [Murphy Randle](http://murphyrandle.svbtle.com/vittles-for-developing-nodejs-apps-in-docker)
在博客中描述了如何避免 NFS mounts/volumes 带来的痛苦，在使用 Docker 开发应用时通常需要这样。
这时，他决定专门为小一点的应用创建一个开发环境，但是不能用于较大的应用。
问题来了，使用 Dockerfile 构建 Docker image 的过程是[自动的](https://www.digitalocean.com/community/articles/docker-explained-using-dockerfiles-to-automate-building-of-images)。
但是 Randle 写到， 当加载多个 node_modules 目录时，速度会变得相当慢。

Randle 在五月份被采访时说，当 `docker build` 命令被发出，
 docker client 会把  docker file 所在文件夹中的全部内容 上传 到 docker daemon。
“当只有少量源码时，这个过程还是很快的。但是当应用依赖数以兆计的 node_modules 时，
整个过程就需要花费大量时间。” 这个问题在[GitHub](https://github.com/docker/docker/issues/2224) 上有一长串的讨论，最终导致了 `.dockerignore `
的产生。讨论的导火索是这样的：

![](http://thenewstack.io/wp-content/uploads/2014/06/dockerignore.png)

 在Docker的整个生态系统中， Docker 都做了改进，包括对 Docker Engine , Docker Hub 以及它的文档都做了更新。
 总体来说，这并不是主要版本更新，但更多的是对这个仍处于发展阶段的开源技术的评述。
 
 
