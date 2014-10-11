# 构造有效的 Dockerfile —— Node.js


##### 作者：[David Weisnstein](https://twitter.com/insitusec)
##### 译者：[李兆海](https://twitter.com/googollee)

***

在安装完所有的依赖，还没有**添加**你自己的代码到容器时，使用下面的代码片段（或者稍作修改）。这样每次重新构造容器时，不需要重新构造你自己的模块。如果 `package.json` 有变化，你的模块才会重新构造。完整的例子在这个 [gist](https://gist.github.com/dweinstein/9550188) 里。

```

	# 将这段代码插到 Dockerfile 里，在所有依赖之后，但在自己的代码之前。

	ADD package.json /tmp/package.json
	RUN cd /tmp && npm install
	RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/
```

## 为了模块使用缓存层

这篇文章是关于如何高效使用docker的中间层的；同时我们会看到如何降低 docker 容器里 node.js 应用的开发和调试时间。随着把开发环境下所有事情迁移到 docker ，总有些别扭产生，主要还是“边修改边测试”的交互式工作方式。

过去，每当我对应用做了一点小修改，都要花时间等待重新构造 docker 容器，经常是在等待重新安装所有的模块。我花在等待编译依赖的时间，要长于真正在修正问题时的时间。我希望这篇文章能帮助其他人从这种工作循环里解脱出来。

Docker 作为一个新技术，如何写出高效的 `dockerfile` 是使用这个新技术乐趣的一部分。 **Docker 强迫你用不同的方式思考。一旦这种思考方式成型，你会发现新诀窍。**

关键是理解 docker 存储层是如何工作的。这份[文档](http://docs.docker.io/en/latest/terms/layer/)中的一张图，演示了 docker 如何包含了多个存储层。 `dockerfile` 里的命令会创建多个新的存储层。如果可能， docker 会尝试使用已经缓存的、之前创建过的存储层。你应该尽量利用存储层的这个优点，并根据这个特性重新安排 dockerfile 的命令顺序。我们一会儿会深入看看如何构造你应用里的 node 模块，并充分利用这个特性。

首先是描述 node 程序的依赖文件的例子：

```
	{
	  "name": "myApp",
	  "description": "This is my awesome app...",
	  "version": "0.0.1",
	  "private": true,
	  "scripts": {
	    "start": "node server.js"
	  },
	  "dependencies": {
	    "docker.io": "*",
	    "redis": "*",
	    "restify": "*"
	  }
	}
```

这里是我们要插入到旧 dockerfile 的代码：

```
	# 将这段代码插到 Dockerfile 里，在所有依赖之后，但在自己的代码之前。

	ADD package.json /tmp/package.json
	RUN cd /tmp && npm install
	RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/
```

这段代码通常应该在你应用的所有依赖安装好之后，但是你的代码还没有加入到容器之前跑。

一个不好的 `dockerfile` 看起来是这样：

```
	FROM ubuntu
	 
	RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
	RUN apt-get update
	RUN apt-get -y install python-software-properties git build-essential
	RUN add-apt-repository -y ppa:chris-lea/node.js
	RUN apt-get update
	RUN apt-get -y install nodejs
	 
	WORKDIR /opt/app
	 
	ADD . /opt/app
	RUN npm install
	EXPOSE 3001
	 
	CMD ["node", "server.js"]
```

不好的原因是，在 [第12行](https://gist.github.com/dweinstein/9550778#file-bad-dockerfile-L12) 我们拷贝 app 的工作目录——这个目录包含了 package.json 文件——到我们的容器的工作目录 `.` ，然后重新构造了所有的模块。这导致每次我们在目录 `.` 里改动文件，都会重新构建所有的模块。

下面这个完整的例子向我们展现了好的实现：

```
	FROM ubuntu
	MAINTAINER David Weinstein <david@bitjudo.com>
	 
	# 安装依赖和 nodejs
	RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
	RUN apt-get update
	RUN apt-get -y install python-software-properties git build-essential
	RUN add-apt-repository -y ppa:chris-lea/node.js
	RUN apt-get update
	RUN apt-get -y install nodejs
	 
	# 当我们改变程序在 nodejs 的依赖时，改 变package.json 会强制 docker 不使用已有的缓存
	ADD package.json /tmp/package.json
	RUN cd /tmp && npm install
	RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/
	 
	# 这里开始载入我们的应用代码，因此之前的 docker 存储层缓存在可能时会被用到
	WORKDIR /opt/app
	ADD . /opt/app
	 
	EXPOSE 3000
	 
	CMD ["node", "server.js"]
```

这里的想法是，如果 `package.json` 文件被改了（[第14行](https://gist.github.com/dweinstein/9550188#file-dockerfile-L14)），那么 docker 会重新执行 `npm install` 那一系列的命令（[第15行](https://gist.github.com/dweinstein/9550188#file-dockerfile-L15)），其他情况下， docker 会使用已有的缓存，跳过那部分的执行。

下面的日志展示了，我们的 docker 容器现在在利用之前的 dockerfile 构建时，在模块依赖那步使用了缓存。

```
	Uploading context 4.608 kB
	Uploading context 
	Step 0 : FROM ubuntu
	 ---> 9cd978db300e
	Step 1 : MAINTAINER David Weinstein <david@bitjudo.com>
	 ---> Using cache
	 ---> 67aeca8f12ae
	Step 2 : RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
	 ---> Using cache
	 ---> be8f73b1204f
	Step 3 : RUN apt-get update
	 ---> Using cache
	 ---> 70395f80789a
	Step 4 : RUN apt-get -y install python-software-properties git build-essential
	 ---> Using cache
	 ---> 58821e45ea25
	Step 5 : RUN add-apt-repository -y ppa:chris-lea/node.js
	 ---> Using cache
	 ---> 79afb0c0539a
	Step 6 : RUN apt-get update
	 ---> Using cache
	 ---> 18fc6aa866d8
	Step 7 : RUN apt-get -y install nodejs
	 ---> Using cache
	 ---> 1f1f41f47329
	Step 8 : ADD package.json /tmp/package.json
	 ---> Using cache
	 ---> 0331fd81b4c8
	Step 9 : RUN cd /tmp && npm install
	 ---> Using cache
	 ---> 95ee8b27b72b
	Step 10 : RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/
	 ---> Using cache
	 ---> 40102f5ce4f1
	Step 11 : WORKDIR /opt/app
	 ---> Using cache
	 ---> 6a1ad0dca915
	Step 12 : ADD . /opt/app
	 ---> 6b9bdaa0e7a2
	Step 13 : EXPOSE 3000
	 ---> Running in 722c8f0b88e2
	 ---> d97a3d372bda
	Step 14 : CMD ["node", "server.js"]
	 ---> Running in 3309a2dab1cc
	 ---> a0b19d7625d3
	Successfully built a0b19d7625d3
```

所有的例子都在这个 [gist](https://gist.github.com/dweinstein/9550188) 里，你可以完整重复我所做的事情。

假设你之前构建过容器（比如 `docker build -t testProject .` ），然后去掉例子的 server.js 里第七行的注释（模拟修改了应用程序的逻辑），再看看重新构建容器，日志会提示发生了什么。看看日志，在[第32行](https://gist.github.com/dweinstein/9550105#file-gistfile1-txt-L32)使用了缓存，而[第38行](https://gist.github.com/dweinstein/9550105#file-gistfile1-txt-L38)则没有使用缓存。

## 结论

由于现在缓存了模块，所以每次修改应用程序代码时，不会再重新构建这些模块。这会大大提高测试和调试 nodejs 应用程序的速度。而且，这种缓存技巧也可以用于 ruby gems ，就像我们在另一篇文章里讨论的那样。

***

##### 这篇文章由 [David Weisnstein](https://twitter.com/insitusec) 发表，[李兆海](https://twitter.com/googollee)，点击 [这里](http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js) 可阅读原文。

##### The article was contributed by [David Weisnstein](https://twitter.com/insitusec) , click [here](http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js) to read the original publication.
