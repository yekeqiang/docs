Go，docker和持续集成服务器

前不久，我花了很长时间摆弄go和docker。我愿意花我一部分（也不是很多）业余时间在这两个技术上，是有原因的，不过我不打算谈这些原因，因为这不能给读者带来任何实践经验。这些原因只是我个人的想法，我也不确定这些原因对别人来说是否有意义。所以，本篇会以一个我不能开源的业余项目说起。这个项目是用go写的。当然，开始一个新项目，随之而来就会有类似的问题：

 - 我应该如何对项目进行持续集成？
 - 我应该如何发布项目？

通常，如果是ruby项目，我对这些问题有常用的答案，但是这些答案无法应用到全新的go项目上。一部分是因为go是个编译型语言（比如说，发布就包括“产生可执行程序并上传”这种步骤），一部分也因为go语言对我是个全新的世界。基于此，我希望在这个项目上达到下列小目标：

 - 我的私有仓库的架设必须便宜
 - 持续集成服务器的架设同样要便宜
 - 我的持续集成服务器不能安装任何与项目有关的依赖
 - 我的发布脚本必须快速而简短（以代码行数来算的简短）

记住这些，我做了一些调研，并且得到了如下的结果：

 - gitlab & gitlab-ci
 - mina
 - docker

就我看，[gitlab](https://www.gitlab.com/gitlab-ce/)和[gitlab-ci](https://www.gitlab.com/gitlab-ci/)是很明显的选择。我必须承认，我对这个选择有些怀疑，因为我第一次试用gitlab时印象并不深刻。实际情况是，这个项目已经有了巨大的变化，现在我找不到这个项目上的任何问题。安装过程非常简单（我在某些步骤上浪费了一些时间，现在这些不清的步骤已经被[这个改动](https://github.com/gitlabhq/gitlab-ci-runner/pull/78)修正了），而且界面速度很快，也很成熟。我不知道在重度试用时gitlab和gitlab-ci会有什么样的反应，但是我对我所看到的使用情况有非常好的印象。在代码存放和持续集成服务之间做集成感觉非常好，不用花太多时间做设置。还有一点，这两个应用可以流畅的跑在每月10刀的只有1GB内存的服务器上。

[mina](http://nadarei.co/mina/)类似capistrano，是个快速的发布工具。我对这个项目感兴趣，是因为它把速度作为设计目标。所以回答“如何发布项目”这个问题就变得很简单，而且我认为是mina让这个回答变得简单。只有一个地方需要些技巧，就是找到生成可执行文件并将其发送到生产机上去。我看到一些人使用类似s3的云存储，但是我并不喜欢这些方案，这些方案花费不少，但是没有新增任何价值。而且，我也没法控制我的发布流程的所有步骤。所以，我使用了简单粗暴的方法：rsync。假设我的项目叫做awesome-go-project，这是我现在的发布脚本：

	set :deploy_binary, lambda { "awesome-go-project-#{`git --no-pager log --format="%h" -1`.strip}" }

	task setup: :environment do
	  queue! %[mkdir -p "#{deploy_to}/shared/log"]
	  queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/log"]
	end

	desc 'Build a release'
	task :release do
	  sh 'make build-release'
	  sh "rsync --progress -avzp -e 'ssh -p #{port}' release/awesome-go-project #{user}@#{domain}:/tmp/#{deploy_binary}"
	end

我忽略了大部分无聊的配置设置和实际的发布任务，因为这个任务只是把要发布的可执行文件`deploy\_binary`链接到合适的文件名，然后执行`sudo service awesome-go-project restart`。只有一件事情需要多写几笔，就是我管理可执行文件的方法。因为只有执行文件，我在服务器上不需要源码管理软件，这样就无法立刻知道我在生产环境里执行的是哪个版本的程序，所以所有我对deploy\_binary做的就是使用一个包含提交的SHA1值的文件名。另外一个值得注意的细节是，因为我自己使用mac系统，而生产环境使用的ubuntu，所以我需要学习如何交叉编译go代码，当然这个利用[golang-crosscompile](https://github.com/davecheney/golang-crosscompile)工具可以轻易完成。

从docker开始，才是真正有意思的部分。当思考如何避免在持续集成服务器上安装项目依赖时，我意识到这是我第一次可以用docker做一些“Hello World”容器之外的事情。我的业余项目有一些依赖（比如强大的influxdb和mysql），但是为了不陷入争论，让我们把项目限制在早期状态，就是只依赖`go test -v`命令来执行测试的状态。首先，我创建了两个docker仓库，[go-lang](https://index.docker.io/u/lucapette/go-lang/)和[go-command](https://index.docker.io/u/lucapette/go-command/)。这有些感觉是在重复造轮子，因为已经有很多仓库这么做过了，不过他们都和我想要的不太一样。因为我是完全以学习的态度在进行这个业余项目，我不是*“接受现有的解决方案以便快速解决问题”*，而是*“我完全按自己的想法制造东西虽然会慢一些”*。这个体验很棒。开发Dockerfile是个有趣的过程，因为docker自己有很好的界面和很棒的文档，所以很容易学习。最后，`lucapette/go-lang`看上去是这样：

	FROM debian:jessie

	MAINTAINER lucapette <lucapette@gmail.com>

	RUN apt-get update && apt-get install -y \
	    build-essential \
	    curl \
	    git \
	    make

	# 获取并编译go
	RUN curl -s https://go.googlecode.com/files/go1.2.1.src.tar.gz | tar -v -C /usr/local -xz
	RUN cd /usr/local/go/src && ./make.bash --no-clean 2>&1
	ENV PATH /usr/local/go/bin:/go/bin:$PATH
	ENV GOPATH /go

需要注意的是我要使用的镜像。我遵从了[Docker最佳实践——第二部分](http://crosbymichael.com/dockerfile-best-practices-take-2.html)的建议，你也应该通读一遍（而且也遵守）。这个容器_只_提供给我们安装好的go。我下一步做的是使用`lucapette/go-lang`创建的镜像来创建`lucapette/go-command`：

	FROM lucapette/go-lang

	RUN mkdir -p /go/src

	ENTRYPOINT ["go"]

这个很简短，因为所有的魔法都包含在[ENTRYPOINT](http://docs.docker.io/en/latest/reference/builder/#entrypoint)命令里。简单来说，这个命令让容器执行一个命令，这里是`go`命令。现在，让我们继续假设我的项目叫做`awesome-go-project`，我的持续集成脚本看上去像这样：

	#!/bin/bash

	set -e

	docker run --rm=true -v `pwd`:/go/src/github.com/lucapette/awesome-go-project -w /go/src/github.com/lucapette/awesome-go-project  lucapette/go-command test -v

The script makes the assumption it will run from the root directory of the awesome-go-project. There are a few things happening, let’s dissect the docker command I’m running:
脚本假设从awesome-go-project的根目录执行。发生了一些事情，让我们解析一下我执行的docker命令：

 - `--rm=true`

    伟大的特性。一旦容器退出，docker会删除这个容器。

 - `-v \`pwd\`:/go/src/github.com/lucapette/awesome-go-project`

    这个命令将当前目录（比如，项目的源代码）挂接到冒号后指定的路径。

 - `-w /go/src/github.com/lucapette/awesome-go-project`

    这个命令将工作路径设为给定的目录

 - `test -v`

    因为我们有了`ENTRYPOINT`设置，这个只考虑执行go命令时的参数。

当我把代码推到master，gitlab-ci会执行脚本，运行docker命令并执行测试。在设计了这个流程后，我发现我真的可以在持续集成服务器里运行我的测试，而不用安装任何与测试代码相关的依赖，甚至连项目使用的语言都不用安装，因为服务器上所有的依赖就是docker本身。我认为这是个很棒的权衡。下一步，也是我正在做的，是让测试集运行的更稳定。这将导致更多的依赖，比如真实的influxdb服务器。进一步，我已经能预见到，在测试时在容器之间建立链接，是件很有趣的事情。