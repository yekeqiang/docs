# 不安装直接使用 Fig

---

##### 作者：Larry Cai

---

## Fig 是什么？

[Fig](http://fig.sh) 是一个简单的编配工具，用于管理多个docker容器，目前该公司是由docker公司收购了。

使用很简单，你把配置写在 [`fig.yml`](https://github.com/larrycai/codingwithme-ansible/blob/master/fig.yml)，代码如下：

```
ansible:
  image: dockerfile/ansible
  volumes: 
   - /home/docker/codingwithme-ansible:/data
  links: 
   - haproxy
   - web1
   - web2
   - database
haproxy:
  image: larrycai/ubuntu-sshd
web1:
  image: larrycai/ubuntu-sshd
web2:
  image: larrycai/ubuntu-sshd
database:
  image: larrycai/ubuntu-sshd
```

然后执行fig run ansible，你将启动你配置的docker容器：上面的示例是启动5个docker的容器，最后进入ansible容器的shell窗口。

更多的 Fig 知识可以阅读 [深入浅出Docker（五）：基于Fig搭建开发环境](https://docker.cn/p/docker-build-development-environment-based-on-fig)

## 运行环境的问题

使用fig之前，大多需要安装它，如果你查看[官方的安装指导](http://www.fig.sh/install.html)，你会发现它仅支持OS X（与boot2docker 1.3）和其他Linux系统，现在你不能安装在Windows（boot2docker 环境）。

此外，如果你开始玩CoreOS，你会发现这也很难安装，在官方文件中，建议使用fleet（coreos工具）管理docker的服务。 那样，你需要运行 [fig2coreos](https://github.com/centurylinklabs/fig2coreos) ，它不是那么的小巧。

怎样才能解决呢？

一个命令就可以了，懂得人可以不用看后面解释了。

```
docker run larrycai/fig
```

### 用docker中的docker使用fig容器

解决的办法其实并不难，我们可以简单的用一个fig容器内来启动docker，肯定它需要docker in docker的技术。

看代码 [Dockerfile](https://github.com/larrycai/docker-images/blob/master/fig/Dockerfile)

```
## docker run -v /var/run/docker.sock:/docker.sock -v <figapp>:/app larrycai/fig

FROM ubuntu:latest
MAINTAINER Larry Cai "larry.caiyu@gmail.com"
ENV REFREST_AT 20141015

RUN apt-get update && apt-get install -y curl make

RUN \
    curl -L https://get.docker.io/builds/Linux/x86_64/docker-latest -o /usr/local/bin/docker  && \
    chmod +x /usr/local/bin/docker && \

    # see http://www.fig.sh/install.html 
    curl -L https://github.com/docker/fig/releases/download/1.0.0/fig-`uname -s`-`uname -m`  -o /usr/local/bin/fig && \
    chmod +x /usr/local/bin/fig 

ENV DOCKER_HOST unix:///docker.sock

WORKDIR /app

# set initial command

ENTRYPOINT ["/usr/local/bin/fig"]
CMD ["-v"]
```

它会在里面安装最新的docker客户端，也跟着安装Fig 1.0.0版本 （方法详见[官方](http://www.fig.sh/install.html)）。

### 如何使用它？

像通常的docker container一样，直接运行它，看看 fig 的帮助（这是默认设置）：

```
docker@boot2docker:~$ docker run larrycai/fig
Punctual, lightweight development environments using Docker.

Usage:
  fig [options] [COMMAND] [ARGS...]
  fig -h|--help
....
```

现在，如果克隆我的 [codingwithme-ansible sample code](https://github.com/larrycai/codingwithme-ansible) 示例代码到本地/home/docker (见上面的代码fig.yml），那么我就可以运行。

```
docker@boot2docker:~/codingwithme-ansible$ docker run -it \
 -v /var/run/docker.sock:/docker.sock \
 -v /home/docker/codingwithme-ansible:/app \
 larrycai/fig run ansible
```

然后，它会启动Web堆栈容器（haproxy/web/database)，第一次它会花些时间下载docker的镜像），并运行到ansible容器中。

`-v /var/run/docker.sock:/docker.sock` is used to pass the docker daemon socket into docker container so docker inside can community outside `-v /home/docker/codingwithme-ansible:/app` is to share the host folder inside. 

> 注：作者原文中本段有错误，需勘误，修正后会提供最新译文


### 提醒

只是提醒一件事，因为这些docker的命令实际在主机上运行，请在volumes配置中指定绝对路径，不要使用相对路径.。

```
volumes: - /home/docker/codingwithme-ansible:/data
```

我猜这可能是使用fig docker容器唯一的限制。

## 摘要

现在使用Fig的docker 镜像，你不需要手动安装了，这就是 docker 的核心价值，一切都是service（业务）。 它在Windows中（boot2docker 1.3以上版本）和CoreO都能很好动作。

享受docker的美好生活 v5。

---

本文原载于作者的 [博客](http://larrycaiyu.com/)，原文地址：[不安装直接使用 Fig](http://larrycaiyu.com/2014/10/30/running-fig-without-installation-in-docker-host.html)，欢迎通过 [微博](http://weibo.com/124565421?sudaref=larrycaiyu.com) 与作者交流。