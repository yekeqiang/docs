#Docker quicktip #3 – ONBUILD
#Docker quicktip #3 - ONBUILD

***

Docker 0.8 came out today, with it a slew of fantastic enhancements. Today we’ll be looking at one of them: ONBUILD.

今天， Docker0.8 发布了，新版增加了很多强大的特性。我们今天就来看其中一个：ONBUILD。

***

ONBUILD is a new instruction for the Dockerfile. It is for use when creating a base image and you want to defer instructions to child images. For example:

**ONBUILD** 是 **Dockerfile** 中新增的一个命令，**ONBUILD** 指定的命令在构建镜像时并不执行，而是在它的子镜像中执行。例如：

***

```
# Dockerfile
FROM busybox
ONBUILD RUN echo "You won't see me until later"


docker build -t me/no_echo_here .
 
Uploading context  2.56 kB
Uploading context
Step 0 : FROM busybox
Pulling repository busybox
769b9341d937: Download complete
511136ea3c5a: Download complete
bf747efa0e2f: Download complete
48e5f45168b9: Download complete
---> 769b9341d937
Step 1 : ONBUILD RUN echo "You won't see me until later"
---> Running in 6bf1e8f65f00
---> f864c417cc99
Successfully built f864c417cc9
```
***

Here the onbuild instruction is read, not run, but stored for later use.

可以看到，构建镜像时读到了 onbuild 指定的命令，但是命令并没有执行，而是稍后执行。

***

Here is the later use:

稍后就是这样执行的：

***

```
# Dockerfile
FROM me/no_echo_here


docker build -t me/echo_here .
Uploading context  2.56 kB
Uploading context
Step 0 : FROM cpuguy83/no_echo_here
 
# Executing 1 build triggers
Step onbuild-0 : RUN echo "You won't see me until later"
 ---> Running in ebfede7e39c8
You won't see me until later
 ---> ca6f025712d4
 ---> ca6f025712d4
Successfully built ca6f025712d4
```   

***

The ONBUILD instruction only gets run when building the cpuguy83/echo_here image.

ONBUILD 指令只在构建 cpuguy83/echo_here 镜像时才执行。

***

ONBUILD gets run just after the FROM and before any other instructions in a child image.

ONBUILD 指令在 FROM 命令后执行，但早于子镜像的任何命令。

***

You can also have multiple ONBUILD instructions.

你也可以指定多个 ONBUILD 指令。

***

Why would you want this? It turns out it’s pretty darn awesome, and powerful. I have a demo github repo setup for this: [Docker ONBUILD Demo](https://github.com/cpuguy83/docker-onbuild_demo)

为什么要这样呢？其实 **ONBUILD** 是非常棒的一个命令，而且非常强大！我在 Github 上放了一个 [ONBUILD 的 Demo](https://github.com/cpuguy83/docker-onbuild_demo)

***

Before diving into this, I just want to say I’ve probably used ONBUILD a bit excessively here in order to get the point across for what ONBUILD does and what it can do, it’s up to you how to use it in your projects.

在看 Demo 之前，我先声明，为了演示ONBUILD的功能，我在 Demo 中可能有点儿过度使用 ONBUILD 了。大家在自己的项目中要酌情使用。

***

```
# Dockerfile
FROM ubuntu:12.04
 
RUN apt-get update -qq && apt-get install -y ca-certificates sudo curl git-core
RUN rm /bin/sh && ln -s /bin/bash /bin/sh
 
RUN locale-gen  en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8
 
RUN curl -L https://get.rvm.io | bash -s stable
ENV PATH /usr/local/rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
RUN /bin/bash -l -c rvm requirements
RUN source /usr/local/rvm/scripts/rvm && rvm install ruby
RUN rvm all do gem install bundler
 
ONBUILD ADD . /opt/rails_demo
ONBUILD WORKDIR /opt/rails_demo
ONBUILD RUN rvm all do bundle install
ONBUILD CMD rvm all do bundle exec rails server
```

This Dockerfile is doing some initial setup of a base image. Installs Ruby and bundler. Pretty typical stuff. At the end are the ONBUILD instructions.

在这个 Dockerfile 中，完成了一个基础镜像的初始化设置，安装了 Ruby 和 bundler ，都是一些典型的软件。最后是 ONBUILD 的命令。

```
ONBUILD ADD . /opt/rails_demo

```

Tells any child image to add everything in the current directory to /opt/rails_demo. Remember, this only gets run from a child image, that is when another image uses this one as a base (or FROM). And it just so happens if you look in the repo I have a skeleton rails app in rails_demo that has it’s own Dockerfile in it, we’ll take a look at this later.

这个命令让子镜像把当前目录的所有内容复制到 /opt/rails_demo 目录中。请记住，这些命令只会在子镜像中执行（所谓的子镜像，就是通过 FROM 指令从当前镜像衍生出来的镜像）。如果你看一下 rails-demo 这个 git 库，就会发现，其中有一个 rails 程序框架，它有自己的 Dockerfile，我们随后再看这个 Dockerfile 文件。

***

```
ONBUILD WORKDIR /opt/rails_demo
```

***

Tells any child image to set the working directory to /opt/rails_demo, which is where we told ADD to put any project files

这条命令让子镜像把工作目录设置到 /opt/rails_demo ，也就是之前我们用 ADD 命令拷贝文件的目标目录。

***

```
ONBUILD RUN rvm all do bundle install
```

***

Tells any child image to have bundler install all gem dependencies, because we are assuming a Rails app here.

这条命令，会让 bundler 在子镜像中安装所有必须的依赖，因为我们要在子镜像中安装一个 Rails 程序。

***

```
ONBUILD CMD rvm all do bundle exec rails server
```

***

Tells any child image to set the CMD to start the rails server

这条命令让子镜像中调用 CMD 命令来开启 rails 服务。

***

Ok, so let’s see this image build, go ahead and do this for yourself so you can see the output.

OK，我们来看看这个镜像的构建，请大家来执行这些命令，看看输出的结果。

***

```
git clone git@github.com:cpuguy83/docker-onbuild_demo.git
cd docker-onbuild_demo
docker build -t cpuguy83/onbuild_demo .
 
Step 0 : FROM ubuntu:12.04
 ---> 9cd978db300e
Step 1 : RUN apt-get update -qq && apt-get install -y ca-certificates sudo curl git-core
 ---> Running in b32a089b7d2d
# output supressed
ldconfig deferred processing now taking place
 ---> d3fdefaed447
Step 2 : RUN rm /bin/sh && ln -s /bin/bash /bin/sh
 ---> Running in f218cafc54d7
 ---> 21a59f8613e1
Step 3 : RUN locale-gen  en_US.UTF-8
 ---> Running in 0fcd7672ddd5
Generating locales...
done
Generation complete.
 ---> aa1074531047
Step 4 : ENV LANG en_US.UTF-8
 ---> Running in dcf936d57f38
 ---> b9326a787f78
Step 5 : ENV LANGUAGE en_US.UTF-8
 ---> Running in 2133c36335f5
 ---> 3382c53f7f40
Step 6 : ENV LC_ALL en_US.UTF-8
 ---> Running in 83f353aba4c8
 ---> f849fc6bd0cd
Step 7 : RUN curl -L https://get.rvm.io | bash -s stable
 ---> Running in b53cc257d59c
# output supressed
---> 482a9f7ac656
Step 8 : ENV PATH /usr/local/rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
 ---> Running in c4666b639c70
 ---> b5d5c3e25730
Step 9 : RUN /bin/bash -l -c rvm requirements
 ---> Running in 91469dbc25a6
# output supressed
Step 10 : RUN source /usr/local/rvm/scripts/rvm && rvm install ruby
 ---> Running in cb4cdfcda68f
# output supressed
Step 11 : RUN rvm all do gem install bundler
 ---> Running in 9571104b3b65
Successfully installed bundler-1.5.3
Parsing documentation for bundler-1.5.3
Installing ri documentation for bundler-1.5.3
Done installing documentation for bundler after 3 seconds
1 gem installed
 ---> e2ea33486d62
Step 12 : ONBUILD ADD . /opt/rails_demo
 ---> Running in 5bef85f266a4
 ---> 4082e2a71c7e
Step 13 : ONBUILD WORKDIR /opt/rails_demo
 ---> Running in be1a06c7f9ab
 ---> 23bec71dce21
Step 14 : ONBUILD RUN rvm all do bundle install
 ---> Running in 991da8dc7f61
 ---> 1547bef18de8
Step 15 : ONBUILD CMD rvm all do bundle exec rails server
 ---> Running in c49139e13a0c
 ---> 23c388fb84c1
Successfully built 23c388fb84c1
```

***

Now let’s take a look at that Dockerfile in the rails_demo project:
现在我们来看看 rails_demo 项目的 Dockerfile ：

```
# Dockerfile
FROM cpuguy83/onbuild_demo

```

WAT?? This Dockerfile is a grand total of one line. It’s only one line because we setup everything in the base image. The only pre-req is that the Dockerfile is built from within the Rails project tree. When we build this image, the ONBUILD commands from cpuguy83/onbuild_demo will be inserted just after the FROM instruction here.

什么？ Dockerfile 总共只有那么牛叉闪闪的一行？确实如此，只有一行，因为我们把所有设置都放在父镜像中了。子镜像只需要保证 Dockerfile 是从 Rails 项目树中衍生出来的即可。当我们构建这个镜像时， cpuguy83/onbuild_demo 镜像中的 ONBUILD 命令就会插入到 FROM 指令后面。

***

> 译者注： Rails 项目树应该是指作者的 rails-demo 的镜像及其子镜像所形成的一个树形结构，只要 Dockerfile 中 FROM 指定的父镜像是这个树形结构中的任何一个节点。由于 rails-demo 中指定了 ONBUILD 指令，所以衍生的子镜像会执行 ONBUILD 所指定的命令。

***

Remember, this aggressive use of ONBUILD may not be optimal for your project and is for demo purposes… not to say it’s not ok 

请注意，这种过度使用 ONBUILD 的方式不一定适合你的项目，我在这里也是为了 Demo ，当然，也并不是说这种方式一定不 OK 。嘿嘿

***

So let’s run this:

好了，我们来执行构建命令吧：

***

```
cd rails_demo
docker build -t cpuguy83/rails_demo .

Step onbuild-0 : ADD . /opt/rails_demo
---> 11c1369a8926
Step onbuild-1 : WORKDIR /opt/rails_demo
---> Running in 82def1878360
---> 39f8280cdca6
Step onbuild-2 : RUN rvm all do bundle install
---> Running in 514d5fc643f1
# output supressed
Step onbuild-3 : CMD rvm all do bundle exec rails server
---> Running in df4a2646e4d9
---> b78c1813bd44
---> b78c1813bd44
Successfully built b78c1813bd44
    
```    

***

Then we can run the rails_demo image and have the rails server fire right up

构建完成后，我们就可以启动 rails_demo 镜像了，其中的 rails 服务器也随之启动了。
   
***   
   
```
docker run -i -t cpuguy83/rails_demo
 
=> Booting WEBrick
=> Rails 3.2.14 application starting in development on http://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
[2014-02-06 11:53:20] INFO  WEBrick 1.3.1
[2014-02-06 11:53:20] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-06 11:53:20] INFO  WEBrick::HTTPServer#start: pid=193 port=3000
    
```    

***

TLDR; ONBUILD… awesome. Use it to defer build instructions to images built from a base image. Use it to more easily build images from a common base but differ in some way, such as different git branches, or different projects entirely.

此处省略两万五千字... ONBUILD ，嗯，很棒的命令，可以用它把镜像中指定的命令延迟到子镜像中去执行。用 ONBUILD 可以从一个通用的父镜像中更方便的衍生出某些方面有所不同的子镜像，例如子镜像可以有不同分支，或者整个项目都是全新的。

***

With great power comes great responsibility.

力量越大，责任越大，这就是 ONBUILD 。

***
