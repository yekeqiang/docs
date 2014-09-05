#####作者：[Peter Parente](https://twitter.com/parente)
#####译者：[Mark Shao](https://github.com/markshao)

***

Docker 0.8 发布了 Dockerfile 的一个新的指令 ONBUILD。我必须承认，当我在阅读 [发布公告](http://blog.docker.io/2014/02/docker-0-8-quality-new-builder-features-btrfs-storage-osx-support/) 时，我忽视了这个新特性。现在我有充分的时间 [再读一遍它](http://docs.docker.io/en/latest/reference/builder/#onbuild) ，我发现了它在构建环境上的潜力。


现在我通过把 reveal.js 容器化来向您展示这个新特性。


### 基础镜像



[Reveal.js](http://lab.hakim.se/reveal-js/#/) 把自己定义为“使用 HTML 技术来展示漂亮幻灯片的一个简易框架”。当我们直接从文件系统装载幻灯片的时候，reveal.js 的大多数特性是正常工作的。但是有些特性，比如 Markdown 的支持，需要我们通过一个 Web 服务器来装载幻灯片。 Docker 可以把所有东西装载到一个容器里，方便我们迁移到任何地方并执行。

我们可以写一个 **Dockerfile** 让它在构建的时候安装 **NodeJS** , 下载 **reveal.js** 并使用 **npm** 安装它的依赖以及把我们的 **Markdown** 幻灯片添加到容器中。接下来我们可以在任何我们要开发或者执行幻灯片服务的地方来复制并构建这个 **Dockerfile**。 我们也可以可以通过重构的方式或者 **SCP** 的方式来不断地把新的 **Markdown** 文件加入到容器中。 或者我们可以构建一个通用的镜像，当我们执行的时候，需要把主机上的幻灯片通过挂载的方式共享到容器中。

**ONBUILD** 介绍了一种新的方法。通过它我们可以构建一个基本镜像来完整地配置 **reveal.js** 和 **web** 服务器，但是延迟插入我们的幻灯片的内容直到我们构建一个子的 **Dockerfile**。 **Dockerfile** 中的关键部分如下

```
FROM ubuntu:13.10
MAINTAINER Peter Parente <parente@cs.unc.edu>

# elided dependency setup, then ...

ONBUILD ADD slides.md /revealjs/md/

# elided final setup
```

我们可以一次构建一个基础镜像并且通过 docker push 的方式把它存储到 Docker Registry 服务中，或者建立一个可信的构建。（我已经 [实现了后者](https://index.docker.io/u/parente/revealjs/) ，如果你感兴趣，可以查看完整地Dockerfile ）。


### 幻灯片展示镜像

后来当我们有一个新的幻灯片的时候，我们可以写一个只有一行的 Dockerfile

```
FROM parente/revealjs
```

和 slides.md 在共同的目录下：

```
# Docker + reveal.js

### A reveal.js Docker Base Image with ONBUILD

---

## Write more slides
```

并且执行一次构建

```
$ docker build -t slideshow .
```

在这次构建的过程中，Docker 执行延迟了的 ONBUILD ADD slides.md 、revealjs/md/ 的指令，把本地的 slides.md 文件拉到子镜像中。执行的结果是一个可移植的 Docker 镜像，可以在任何有 Docker 的地方运行我们的幻灯片服务器。

```
$ docker run -d -P myslides
```

当然它支持所有 docker run 的标准选项：一个固定的端口、容器命名、在前台运行等等。


### 一个开发故事

上面的构建和执行的步骤可以完美的生成一个完整的幻灯片展示容器。然而，在开发幻灯片的过程中，构建和执行的循环却不怎么理想。

使用混合技术可以用来减轻这个问题。在开发过程中，我们可以通过把主机上的幻灯片通过挂载的方式在一个基础镜像的容器上共享。这样当我们在主机上对 slides.md 进行修改的时候，通过刷新浏览器，我们可以立刻看到发生的变化，而无需把服务关掉、重新构建、重新执行容器。

```
$ docker run -d -P -v `pwd`:/revealjs/md parente/revealjs
```

当我们对结果满意时，就可以构建一个最终版本，一个可以用来迁移和部署的完整镜像。

```
$ docker build -t slideshow .
```

更多的细节，可以查看 [源代码](https://github.com/parente/dockerfiles/tree/master/revealjs) 或者 [parente/revealjs repository](https://index.docker.io/u/parente/revealjs/) 。

---
#####这篇文章由 [Peter Parente](https://twitter.com/parente) 发表， [Mark Shao](https://github.com/markshao) 翻译。你可以点击 [这里](http://mindtrove.info/a-reveal.js-docker-base-image-with-onbuild/) 阅读原文。

#####The article was contributed by [Peter Parente](https://twitter.com/parente), click [here](http://mindtrove.info/a-reveal.js-docker-base-image-with-onbuild/) to read the original publication.
