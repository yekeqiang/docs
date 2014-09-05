#使用 Docker 构建你自己的 PaaS 平台

#####作者：[serverascode](https://twitter.com/serverascode)
#####译者：[邵靖](http://www.weibo.com/dysj4099)

***

首先，我要澄清一点，[Docker](http://www.docker.com/) 自身并不是“平台即服务”（PaaS），而是其中重要的组件，使得部署 PaaS 更为简单。


其次，从我个人观点出发，我在这里谈到的所谓构建“属于自己的 PaaS ”是指像我这样运行一些博客或是 Wordpress 站点。这些站点并非为了商业或盈利目的，单纯用于朋友间交流，并且我有一台联网的服务器硬件设备来运行这些网站。我觉得这台服务器如果提供一个 Wordpress 的 PaaS 服务，能够在需要的时候自动进行横向扩展那该是很有意思的实情。


从某种程度上来说，我所做的不过是为证明 wordpress 可以作为服务来提供，对于单个主机而言可以支持几乎所有的应用，并且有可以自动扩展到多主机的能力。


##为什么不直接用 Dokku

[Dokku](https://github.com/progrium/dokku) 是一个由 [Jeff Lindsay](https://twitter.com/progrium) 写的在单主机环境下构建 PaaS 的软件。

>注： Dokku 是由 Docker 支持的迷你版 Heroku ，最小的 PaaS 实现工具。

虽说 Dokku 可能是最好的实现工具，但我不会用它，而是自己设计最小的系统以及创建 low-fi PaaS 所需的组建。而诸如 [Deis](http://deis.io/) 、Flynn 和 [其它](http://stackoverflow.com/questions/18285212/how-to-scale-docker-containers-in-production) 的容器管理系统也都各具特色。


##组件

以下是一些你在构建自己的 PaaS 时可能会用到的组件。

1. 泛域名解析（ Wildcard DNS entry ）
2. Web 路由器（例如 [Hipache](https://github.com/dotcloud/hipache) ）
3. Docker - 应用服务器/容器，创建镜像
4. 应用源码
5. 环境变量
6. 数据存储，例如 MySQL 、NoSQL 、对象存储等
7. 把上边的组件组合在一起的东西


##泛域名解析

首先，你需要的是对一个域名的泛域名解析。 [这篇文档](https://gist.github.com/ngoldman/7287753) 介绍了如何使用 Namecheap 相关配置。 Namecheap 是我的注册服务商（其他一些注册管理服务商也有类似服务，支持双因子认证），我购买了一个类似于“ somedomainapp.com ”这么一个域名来运行我的应用，并且使用一个 Namecheap 提供的泛域名解析服务。

很显然，在一个大型的生产环境中，你必须管理自己的域名服务或是使用诸如 [Google 的 DNS 服务](https://gist.github.com/ngoldman/7287753)（我喜欢），或是其他一些诸如负载均衡的设备。

至此，你有了一个泛域名解析到你指定的服务IP地址，例如 *.yourdomainapp.com


###Web 路由

![alt](http://resource.docker.cn/somedomain.png)

（上图为 hipache 的 non-existent domain 页面）

我不知道改如何称呼这一层， Heroku 称它为 [HTTP 路由](https://devcenter.heroku.com/articles/http-routing) ，我想这个名字合适。

本质上讲，它的工作就是将输入请求路由到正确的 web 服务器，在我们的例子中也就是 docker 容器。一个请求了 someapp.somedomainapp.com 的请求可能被送到 127.0.0.1:49899 或是 172.17.0.3:80 或其它，这背后都 是docker 容器。

在我们的案例中，我使用 hipache ，它后台使用 redis 。这也就是说你在 hipache 中添加路由也就是把这些规则添加到 redis 里边，并且 hipache 不需要重启，因为它可以查询 redis 以获取域配置。默认情况下 hipache 允许使用通配符域名，所以它可以路由任何请求并且如果目标不存在则发送到默认页面。

我的 PoC Python 脚本被称为“ wpd ”，它能够输出在 redis 中存储的 hipache 键配置。以下的输出意味着 hipache 随机将对 someapp.yourdomainapp.com 的请求平均分布到两个容器之中，如下：

```
$ wpd listkeys
someapp.yourdomainapp.com
===> http://127.0.0.1:49156
===> http://127.0.0.1:49157
$ redis-cli lrange someapp.yourdomainapp.com 0 -1
1) "someapp.yourdomainapp.com"
2) "http://127.0.0.1:49156"
3) "http://127.0.0.1:49157"
```

有其他很多可以做 web 路由的方法。 Dokku 使用 nginx ，还有使用 [etcd](https://github.com/coreos/etcd) 的 [vulcand](https://github.com/mailgun/vulcand) ，这个新东西着实让人兴奋。 Hipache 支持 SSL ，不过几周前 Vulcand 还不支持，但我想这肯定是在计划内的，因为我是 golang 的粉丝，所以相对有些偏心 ;) 。

###Docker!

再来比较一下 Heroku 和我们正在做的事情，我认为 Docker 能够扮演 [buildpack](https://devcenter.heroku.com/articles/buildpacks) 和 [dyno](https://devcenter.heroku.com/articles/dynos) 的角色，虽然也许严格说来 buildpack 不包含应用代码，或者更确切的说只有应用运行所需的环境。把 Dockerfile 看作是一种 buildpack 也许更容易理解。

以我的 wordpress 为例，Dockerfile 文件可以创建 docker 镜像以用来生成一个运行 wordpress 应用的容器，比如使用 apache2 + php 。

Docker 管理容器，并且提供网络以及网络地址转换以将 apache2 的端口暴露给 web 路由。

所以 docker 为我们做了很多事情。没有 docker 的话，我们可能需要写代码以实现一个创建虚拟机镜像的方法，并且还得管理启动、网络、实例化等诸多的实情，这就跟写一个简单的 [packer](http://www.packer.io/) 或 libvirtd ( kvm 或 lxc ) 一样了，就像 openstack 做的那样。毫无疑问那将耗费更多的资源。（有意思的是 packer 也能够创建 docker 镜像）


###应用源代码

在 Dokku 中，代码是被 push 到一个 git 容器中，并启动其他的进程，这也是 Heroku 的做法。这些进程将代码放置到应用容器之中。

然而，在我的 wordpress 案例中， wordpress 的代码是可通过起始脚本下载的。一旦容器从 wordpress 镜像启动，起始脚本就开始运行：

```
if [ ! -e /app/wp-settings.php ]; then
        cd /app
        curl -O http://wordpress.org/latest.tar.gz
        tar zxvf latest.tar.gz
        mv wordpress/* /app
        rm -f /app/wordpress
        chown -R www-data:www-data /app
        rm -f latest.tar.gz
fi
```

看看代码吐吐槽，用来下载的 url 应该从环境变量中获取而不是如上例直接写在代码里边。

git 的 push/receive 风格可能在 PaaS 中更有效，但是我还没有深入去研究那是怎么做到的。 Jeff Lindsay 有一个工具 [gitreceive](https://github.com/progrium/gitreceive) ，并且Flynn （ Jeff 的另一个项目）有 [gitreceived](https://github.com/flynn/gitreceived) 。他还有  [execd](https://github.com/progrium/execd) 和其他项目，真是大忙人！

显然，有很多方法可以讲代码放到容器中去执行。如果要说 PaaS 有什么重要的事情，那么就是运行代码了。

###环境变量

我认为 docker 镜像应该会变得相当普及。同时你不想将敏感的配置信息诸如密码等放到镜像之中，所以它们应该从环境变量中获取，并且这些变量需要通过某种方式注入容器环境中。

在我的 wordpress 例子中，我设定了 docker 的环境变量。 Docker 可以用“ -e ”参数运行命令，以使得设定环境变量的方式暴露出来，来看下面的例子：

```
$ docker run -e FOO=bar -e BAR=foo busybox env
HOME=/
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=6cf2d6e8acb3
FOO=bar
BAR=foo
```

我的 wordpress 启动脚本检查以下几项环境变量：

```
DB_NAME=${DB_NAME:-"wordpress"}
DB_USER=${DB_USER:-"wordpress"}
DB_PASSWORD=${DB_PASSWORD:-"wordpress"}
DB_HOST=${DB_HOST:-$1}
```

并且使用它们和正确的数据库设置创建 wordpress 的配置文件。

稍后我会谈及“如何将所有部分整合”。我通过一个 Python 脚本完成，在这个脚本中，我设置了容器的环境变量。

下面的是 python 代码的一些片段，用环境变量初始化容器：

```
env = {
  'DB_HOST': MYSQL_HOST,
  'DB_NAME': dbname,
  'DB_USER': dbname,
  'DB_PASSWORD': dbpass,
}
container = dockerConn.create_container(image, detach=True, environment=env)
```

可以使用 [docker-py](https://github.com/dotcloud/docker-py) 来联合使用 docker 和 python。

另一种方法是是使用共享配置系统，诸如我之前提到的 etcd 。

>注： etcd 是一个高可用的键值存储系统，用于共享配置和服务发现。

etcd 能够存储配置信息； [confd](https://github.com/kelseyhightower/confd) 则是一个配置管理代理软件，能够查询 etcd 以生成针对应用的   配置文件，并且能够使用这些配置文件重启服务。

说了这么多，我认为 环境/配置 变量是 PaaS 的核心部分，诸如 etcd 、confd 和 [consul](https://github.com/hashicorp/consul) 都将是重要的项目组件。但是，对于本文所说的 wordpress 例子而言，我们只是做一个简单的验证系统，环境变量从容器运行时中获取。然而，我非常建议大型的 PaaS 或是其他类 PaaS 系统能够使用 consul 或 etcd 这样的组件。

###数据存储

如果你的应用需要存留数据，那么将数据放在某个地方就是必然的了，但是使用应用容器来存储显然不是一个好选择。通常来说，我认为有两种解决方案。

1. 另一个容器
2. 一个单独的服务

对于“一个单独的服务”而言，我指的是诸如亚马逊 [RDS](http://aws.amazon.com/rds/) 或 [OpenStack Trove](https://wiki.openstack.org/wiki/Trove) （二者都是数据库即服务）或诸如 [OpenStack Swift](http://docs.openstack.org/developer/swift/) 那样的对象存储系统。简而言之就是有第三方管理的服务，或者 Docker 也运行其上的服务器。

另一个选择是使用“另一个 docker 容器”。再拿 wordpress 来做例子，我可能不只是启动一个应用容器，而是启动第二个包含 MySQL 服务器的容器（或者两个服务运行在一个容器中）。也许 MySQL 的服务是一个容器，也许那是一个通过 Ansible 配置的硬件服务器，谁知道呢。 docker 同样也推荐使用卷（ [volume](https://docs.docker.com/userguide/dockervolumes/) ），尤其是当数据不会分布到多个容器中的时候；如果数据分散，那么就是用 MySQL 或是 openstack swift 吧。

我认为这几种方式都 OK ，但是我更倾向于使用一个单独的服务。正因如此，在我的例子中，存在一个单独的 MySQL 服务器，所有的 wordpress 应用都会连接它，每个应用都有其自己的数据库。或许这个单独的服务也是用 docker 来完成的。


###把上面讲的内容都串起来

我用一个名为 wpd 的脚本来串起所有环节：

1. 在 wpd 数据库中传经一个站点记录
2. 为这个 wordpress 站点创建一个数据库存储
3. 创建多个 wordpress 容器
4. 将环境变量传送给这些容器，让它们知道该如何连接数据库
5. 将站点添加到 redis 以便 hipache 能够做路由/负载均衡

```
$ wpd -h
usage: wpd [-h] {listkeys,addsite,listsites,addimage,deploysite,dumpsite} ...

positional arguments:
  {listkeys,addsite,listsites,addimage,deploysite,dumpsite}
    listkeys            list all the keys in redis
    addsite             add a site to the database
    listsites           list all the sites in the database
    addimage            add a docker image to the database
    deploysite          startup a sites containers
    dumpsite            show all information about a site

optional arguments:
  -h, --help            show this help message and exit
```

正如你所见，有一些选项，诸如“ addsite ”和“ deploysite ”还没有完全弄完。添加站点仅仅是将其放到 wpd 数据库中；部署站点意味着启动容器，并且向 redis 添加了信息以便 hipache 能够将 http 请求路由给它们。

这看起来想是一个大型系统…… 我不太确定哈。看起来更像是一个用户管理系统，因为用户能够拥有站点，站点能够有名字、容器、镜像和数据存储等。

##问题

这里有几个问题我必须提一提（也可能我没有全概括到）。

- 日志 

从 docker 之外获取日志仍旧无解。所以在这里你可能需要在容器中配置 syslog 将日志记录到一个中心系统中。我期望 docker 这边能够想办法解决日志的问题。

- 文件系统 

Wordpress 是一个很好的 Web 应用示例，它很难扩展，因为它依赖文件系统做数据存储，例如用户上传的多媒体文件。为了跨多个 docker 主机扩展文件系统你需要一个分布式的文件系统，这个文件系统必须能够支持动态的扩展，这很令人头疼。所以我建议不用文件系统而是使用对象存储，例如 OpenStack Swift ，其实它并不是这么难搭。但是 Swift 并不能同时保证一致性和可用性。

- 数据安全 

我不确定什么是最好的保证数据安全的做法。一些密码或其他重要的配置信息都需要注入容器中，并且需要存储在诸如 etcd 的系统中，可能会被看到。

##小结

在最后，我认为 docker 用来做 PaaS 组件是非常棒的，对我来说，它简化了将自己的小型平台改造成服务提供者的过程。你所需要做的就是 web 路由、 dockerfile 、 docker 主机、将应用放到容器内的方法，准备好之后你就能够做自己的 PaaS 了。请记住将你的容器做的尽量通用，多用环境变量和配置变量（或是从其他什么地方获取到相关信息），并且尽可能避免使用文件系统。

##未来展望！

很显然我拉下了一些东西。搭建一个验证系统和实际弄一个生产用的 PaaS 还是有非常大的区别的。我还没有提到诸如“服务发现”和“容器调度”，这些偏理论的东西可以用 etcd 或是 libswarm 来帮助你解决，虽然我不清楚 libswarm 是否能做容器调度器。最近 Google 开源了 [Kubernetes](https://news.ycombinator.com/item?id=7873897) ——一个 docker 集群的管理工具，但是它现在仍然只能运行在 GCE 上。 Apache 的 Mesos 同样也只能运行在它的 [Deimos](https://github.com/mesosphere/deimos) 项目上。长远来看，CoreOS 有 [fleet](https://github.com/coreos/fleet) 。同样我也没有谈到诸如资源限制问题，如容器所用内存、CPU 等其他的细节，不过我计划深入研究。

***

#####这篇文章由 [serverascode](https://twitter.com/serverascode) 撰写， [邵靖](http://www.weibo.com/dysj4099) 翻译。点击 [这里](http://serverascode.com/2014/06/16/build-your-own-paas-docker.html) 可阅读原文。

#####The article was contributed by [serverascode](https://twitter.com/serverascode), click [here](http://serverascode.com/2014/06/16/build-your-own-paas-docker.html) to read the original publication.
