#通过代码来创建一个 Docker 镜像

#####作者：[Fernando Mayo](https://twitter.com/FernandoMayo) （ CTO & Co-founder of Tutum ）

#####译者：[Mark Shao](https://github.com/markshao)

***
[Heroku](https://www.heroku.com/) 最重要的开源项目之一是 [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)  。 Buildpacks 实际上是一系列脚本，可以自动侦测到应用的语言/框架，然后安装所运行所需要的解释器、库等等。

[Buildstep](https://github.com/progrium/buildstep) 是 [progrium](http://progrium.com/) 开发的一个非常优秀的工具，通过使用 buildpacks 把任意一些应用代码（被这些 buildpacks 支持）转换成一个自实现的 Dokcer 镜像。它也同样被用于这个作者开发的一个非常流行的项目 [Dokku](https://github.com/progrium/dokku) 。

为了充分利用这个神奇的工具，我们创建了一个名为 [tutum/buildstep](https://index.docker.io/u/tutum/buildstep/) 的基础镜像，可以在 Dockerfile 中直接使用，毫不费力地生成一个应用程序的镜像。


让我们看一个简单的例子

##使用 buildstep 把 Django 应用转换成 Docker 的镜像


假设一下我们有一个 Django 应用，我们希望把它转化成一个  Docker 镜像，从而可以方便我们在服务器间移动。在应用代码的目录中，我们要创建一个如下的 Dockerfie :

```
FROM tutum/buildstep
EXPOSE 80
CMD ["python", "manage.py", "runserver", "80"]
```

非常简单。运行 `tutum/buildstep` 仅需要“暴露的端口”以及“运行应用所需要的命令”这些信息。把应用代码添加进去，配置工作目录都是由 `tutum/buildstep` 使用 `ONBUILD` 指令自动完成的。

我们来构建一下:

```
$ docker build -t fermayo/myapp .
Uploading context 99.84 kB
Uploading context
Step 0 : FROM tutum/buildstep
# Executing 3 build triggers
Step onbuild-0 : RUN mkdir -p /app
 ---> Running in 0d65c9537e8f
 ---> 9d7c609c38ce
Step onbuild-1 : ADD . /app
 ---> 3b7ff5e4f126
Step onbuild-2 : RUN /build/builder
 ---> Running in 6e7a796d93c3
       Python app detected
-----> No runtime.txt provided; assuming python-2.7.6.
-----> Preparing Python runtime (python-2.7.6)
-----> Installing Setuptools (2.1)
-----> Installing Pip (1.5.4)
-----> Installing dependencies using Pip (1.5.4)
       Downloading/unpacking Django==1.6.2 (from -r requirements.txt (line 1))
       Installing collected packages: Django
       Successfully installed Django
       Cleaning up...

-----> Discovering process types
       Procfile declares types -> web
 ---> 8758f592da19
 ---> 8758f592da19
Step 1 : EXPOSE 80
 ---> Running in 3f10763973a8
 ---> db32c55e948b
Step 2 : CMD ["python", "manage.py", "runserver", "80"]
 ---> Running in 7d122de7f8d3
 ---> ed1a72bfa5d0
Successfully built ed1a72bfa5d0
Removing intermediate container 0d65c9537e8f
Removing intermediate container d79aa6530641
Removing intermediate container 6e7a796d93c3
Removing intermediate container 3f10763973a8
Removing intermediate container 7d122de7f8d3
```

现在我们的应用已经变成了一个叫 `fermayo/myapp` 的 Docker 镜像。运行一下:

```
$ docker run -d -p 80 fermayo/myapp
```

Django 应用跑起来了!

## 使用 buildstep 把 Heroku 应用转化成 Docker 镜像

Buildstep 也支持使用 [Procfile](https://devcenter.heroku.com/articles/procfile) 来定义应用的处理类型。我们的 Django 应用以前用到了一个简单的 `Procfile` 。

```
web: python manage.py runserver 80
```

通过使用这个命令而不是手动定义，我们可以创建下面这个 Dockerfile :

```
FROM tutum/buildstep
EXPOSE 80
CMD ["/start", "web"]
```

通过在 CMD 指令中输入要处理的类型的名字，我们指定了用来启动应用的命令。


##运行应用，无需通过 buildstep 来构建镜像

如果你的应用存储在一个 git 的代码库中，那么你可以在运行 `tutum/buildstep` 的时候传入一个指向项目地址的环境变量 `GIT_REPO` ，这样容器会在运行时克隆项目并且下载依赖:

```
# Without a Procfile
$ docker run -d -p 80 -e GIT_REPO=https://github.com/fermayo/hello-world-django.git tutum/buildstep python manage.py runserver 80
```

```
# With a Procfile (or relying on the default Procfile provided by the buildpack)
$ docker run -d -p 80 -e GIT_REPO=https://github.com/fermayo/hello-world-php.git tutum/buildstep /start web
```

所以无需构建任何东西。

## 试试看!

这是一个非常好的使用 Docker 的入门级方法。请让我知道它是否适用于你的语言/框架，以及你计划如何优化它。

谢谢阅读！

***

#####这篇文章由 [Fernando Mayo](https://twitter.com/FernandoMayo) 发表，点击 [这里](http://blog.tutum.co/2014/04/10/creating-a-docker-image-from-your-code/) 可阅读原文。 [Mark Shao](https://github.com/markshao) 翻译了本文，你可以在 [GitHub](https://github.com/markshao) 上与他交流。

#####The article was contributed by [Fernando Mayo](https://twitter.com/FernandoMayo) , click [here](http://blog.tutum.co/2014/04/10/creating-a-docker-image-from-your-code/) to read the original publication.
