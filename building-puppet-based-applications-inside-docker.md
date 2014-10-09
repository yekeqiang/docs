# 在 Docker 中构建基于 Puppet 的应用

#####作者：[James Turnbull](https://twitter.com/kartar)

#####译者：[叶可强](http://weibo.com/yeziyu)

---

通过创建一个 [Docker](http://www.docker.io/) 的 ```Dockerfile``` 来构建一个应用程序是非常容易的。但是如果你已经有了大量的 Puppet 模块（或者是 Chef cookbooks），你想把这些模块用于构建你的应用程序，你应该怎么做？我们将看到利用 Dockerfile[^1] 构建是多么的容易。

我们首先要构建一个 Docker 应用镜像来安装 Puppet。我们将把 Tim Sharpe 的一个非常酷的工具 [Librarian-Puppet](http://librarian-puppet.com/) 添加到镜像中。Librarian-Puppet 是一个 Puppet 模块打包工具，你可以使用它从 GitHub 或者是 [Puppet Labs Forge](http://forge.puppetlabs.com/) 选择和安装模块。

让我们创建一个 ```Dockerfile``` 来构建我们的 Puppet[^2] 镜像。

```
FROM ubuntu:12.10
MAINTAINER James Turnbull "james@lovedthanlost.net"

RUN apt-get -y update
RUN apt-get -y install rubygems 
RUN echo "gem: --no-ri --no-rdoc" > ~/.gemrc
RUN gem install puppet librarian-puppet
```

这个 ```Dockerfile``` 使用基于 Ubuntu 的镜像，然后通过 RubyGems 来安装 Puppet 和 Librarian-Puppet。

我们运行如下命令来构建这个镜像：

```
$ sudo docker build -t="jamtur01/puppetbase" .
```

我们已经构建了一个名为 ```jamtur01/puppetbase``` 的新镜像。我们将使用这个镜像来作为我们新的应用程序镜像的基础镜像。

下一步我们需要创建一个 ```Puppetfile```， Librarian-Puppet 使用它来安装需要的 Puppet 模块。如下例，我们将安装一个 Nginx 基础服务。

```
mod "nginx",
  :git => "https://github.com/jfryman/puppet-nginx"
```

这个 ```Puppetfile``` 告诉 Librarian-Puppet 从 GitHub 中安装 ```puppet-nginx``` 模块。

现在我们需要为我们的应用程序镜像创建另外一个 ```Dockerfile ```。

```
FROM jamtur01/puppetbase
MAINTAINER James Turnbull "james@lovedthanlost.net"

RUN apt-get -y -q install wget git-core
ADD Puppetfile /
RUN librarian-puppet install
RUN puppet apply --modulepath=/modules -e "class { 'nginx': }"
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx"]
```

这个 ```Dockerfile``` 使用我们刚刚构建的 ```jamtur01/puppetbase``` 镜像，它我们本地的 ```Puppetfile``` 添加到镜像的 root 目录，然后运行 ```librarian-puppet install``` 来安装要求的模块（默认安装在 /modules 目录）。

然后我们通过 ```puppet-nginx``` 模块使用 ```puppet apply``` 命令来安装 Nginx 。这个在本地主机上运行 Puppet （i.e. 不需要安装一个 Puppet 客户端）。

在这个镜像中，我们安装了 Nginx 。我们还可以安装虚拟主机或者是一个 Web 应用程序或者是 Nginx 模块支持的任何东西。

我们现在可以像这样构建我们的应用程序镜像：

```
$ sudo docker build -t="jamtur01/nginx" .
```

最后通过它启动一个容器：

```
$ sudo docker run -P -d jamtur01/nginx
fd461a1418c6
```
我们已经启动了一个 ID 为 ```fd461a1418c6``` 的容器，在后台运行，并且告诉它暴露任意的端口，我们的例子中，我们在 ```Dockerfile``` 中暴露了 80 端口，让我们检查容器，并且看看其在 映射的 nginx 端口是。

```
$ sudo docker port fd461a1418c6 80
0.0.0.0:49158
```

现在让我们访问端口 ```49158```，看 nginx 是否正在运行。

![alt](http://resource.docker.cn/nginx.png)

欧耶！我们已经通过 Puppet 安装了 Nginx。你可以重复这个步骤安装任何基于 Puppet 的应用或者是基础设施[^3]。

***

#####这篇文章由 [James Turnbull](https://twitter.com/kartar) 撰写，[叶可强](http://weibo.com/yeziyu) 翻译。点击 [这里](http://kartar.net/2013/12/building-puppet-apps-inside-docker/) 阅读原文。

#####The article was contributed by [James Turnbull](https://twitter.com/kartar), click [here](http://kartar.net/2013/12/building-puppet-apps-inside-docker/) to read the original publication.

---

  [^1]:This is a somewhat short-term hacky implementation. When Docker is more pluggable this will be a lot easier. Expect to see that sort of plugin support in the 1.0 release
  
  [^2]:We could easy do the same thing with Chef too
  
  [^3]:For other thoughts on Docker and CM see this
  


 
