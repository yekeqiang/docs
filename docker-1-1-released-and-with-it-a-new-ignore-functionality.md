
#Docker 1.1 新增 "ignore" 功能

***

#####作者：[Alex Williams](https://twitter.com/alexwilliams)

#####译者：[moonatcs](https://code.csdn.net/moonatcs)

***

![](http://resource.docker.cn/homepage-docker-logo.png)

在今年6月份 Docker 1.0 版本被发布之前， CEO 及创始人 Solomon Hykes 就曾经对我说，
在 Docker 1.1 版本中将增加 .dockerignore 功能。
目前，[Docker 1.1 版本](http://blog.docker.com/2014/07/announcing-docker-1-1/) 已经发布，
Solomon Hykes 所说的话也被验证成真。
现在，开发人员可以在 `Dockerfile` 旁边创建一个 `.dockerignore` 文件，
当给 docker daemon 发送构建上下文的命令时，
Docker 就会忽略 `.dockerignore` 中列出的具体文件和路径。
具体情况，请看 [详细示例](https://github.com/docker/docker/blob/master/.dockerignore) 。

其他方面的更新包括：

* commit 提交一个运行的容器时，该容器会被暂停
* 对容器的日志文件，开发人员可以从指定的位置进行跟踪
* tar 归档文件可以作为 `docker build` 的上下文

我们写了很多关于 Docker 的文章。虽有其他主题，但人们对轻量级容器技术的关注还是日渐增加。创建应用时， Docker 的活力与开发人员的经验相得益彰。这在我们的意料之中，同时原本被忽略的问题，也引起越来越多的关注。

无论是与开发人员交流，还是在 GitHub 上阅读关于当前项目的评论，都提出了一些问题，特别是如何对上传到 Docker 开发环境的文件进行管理，以及在生产环境中管理容器的复杂性。

来自 [Space Monkey](https://www.spacemonkey.com/) 的 [Murphy Randle](http://murphyrandle.svbtle.com/vittles-for-developing-nodejs-apps-in-docker) 在博客中描述了如何避免在使用 Docker 开发应用时 NFS mounts/volumes 带来的痛苦。于是他决定创建一个开发环境，对于小型应用具有实用价值，并不适用于大型应用。问题渐渐浮出水面。使用 Dockerfile 构建 Docker image 的过程是 [自动的](https://www.digitalocean.com/community/articles/docker-explained-using-dockerfiles-to-automate-building-of-images) ，但是 Randle 注意到当加载多个 node_modules 目录时，速度会变得相当慢。

Randle 在五月份接受采访时说，当 `docker build` 命令被发出， docker client 会把  docker file 所在文件夹中的全部内容上传到 docker daemon。“当只有少量源码时，这个过程还是很快的。但是当应用依赖数以兆计的 node_modules 时，整个过程就需要花费大量时间。” 这个问题在 [GitHub](https://github.com/docker/docker/issues/2224)  上有一长串的讨论，最终导致了 `.dockerignore ` 的产生。讨论的导火索是这样的：

![](http://resource.docker.cn/docker-ignore.png)

Docker  has made improvements throughout the Docker ecosystem, including updates to Docker Engine, Docker Hub, and its documentation. Overall, not a major release, but more so a commentary on a still developing open source technology.

Docker 已经对整个 Docker 生态系统中做了改进，包括更新了 Docker Engine 、 Docker Hub 以及文档。总体来说，这并非主要版本更新，更多的是对这个仍处于发展阶段的开源技术的评述。

***

#####这篇文章由 [Alex Williams](https://twitter.com/alexwilliams) 撰写，[moonatcs](https://code.csdn.net/moonatcs) 翻译。点击 [这里](http://thenewstack.io/docker-1-1-released-and-with-it-a-new-ignore-functionality/) 阅读原文。

#####The article was contributed by [Alex Williams](https://twitter.com/alexwilliams) , click [here](http://thenewstack.io/docker-1-1-released-and-with-it-a-new-ignore-functionality/) to read the original publication.

 
 
