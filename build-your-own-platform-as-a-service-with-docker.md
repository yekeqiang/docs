
#Build your own platform as a service with Docker

#使用Docker构建你自己的PaaS平台

[原文地址](http://serverascode.com/2014/06/16/build-your-own-paas-docker.html)

First off, let me be clear—Docker is not a “platform as a service” (PaaS) by itself. However, I do think it’s an important component, one that makes deploying a PaaS much simpler.

首先，我要澄清一点，[Docker](http://www.docker.com/) 自身并不是“平台即服务”（PaaS）, 而是其中重要的组件，它使得部署 PaaS 更为容易。

Second, for the most part I’m discussing the concept of building “your own PaaS” from my personal perspective, which is that I have a few blogs and Wordpress sites that I run for friends (ie. not a business venture or anything) and I have a single hardware server out there on the Internet that I use to do this. I thought it would be fun to use that server to think about how I one could provide a Wordpress PaaS that could potentially scale out to multiple hosts, even though I could never afford to or need to do that.

第二点，从我个人观点出发，我在这里谈到的所谓构建“属于自己的PaaS”是指比如我运行一些博客或是 Wordpress 站点，这些站点并不是为了商业或盈利只是为了在朋友间交流，并且我有一台在网上的服务器硬件设备来运行这些网站。我想我使用这个服务器如果能够提供一个 Wordpress 的 PaaS 服务，能够在需要的时候自动进行横向扩展那该是很有意思的实情，虽然也许我根本不需要这么做。

In a way, what I’m doing is creating a proof of concept (PoC) wordpress as a service, mostly for a single host which once in place could provide almost any application, and could potentially scale out to multiple container hosts.

从某种程度上来说，我所做的不过是为证明 wordpress 可以作为服务来提供，对于单个主机而言可以支持几乎所有的应用，并且有可以自动扩展到多主机的能力。

##Why not just use Dokku?

Dokku is a piece of software created by Jeff Lindsay that essentially creates a single-host PaaS.

>Docker powered mini-Heroku. The smallest PaaS implementation you’ve ever seen.

But I’m not going to use Dokku, even though that is probably the best way to go about this. Instead I’m going to mostly layout the minimum systems and components needed to create a low-fi PaaS. Also Deis, Flynn and other container management systems, in various states with differing features, exist.

##为什么不直接用 Dokku

[Dokku](https://github.com/progrium/dokku) 是一个由 [Jeff Lindsay](https://twitter.com/progrium) 写的在单主机环境下构建 PaaS 的软件。

>由Docker支持的迷你版Heroku。最小的 PaaS 实现工具。

但是我在这里不会去使用 Dokku，即使它可能是最好的实现工具。在这里我将会自己去实现这个迷你系统。同样我想说存在一些[其他](http://stackoverflow.com/questions/18285212/how-to-scale-docker-containers-in-production)的容器管理系统如 [Deis](http://deis.io/)、Flynn 他们都有其各自的特点。

##Components

These are the minimum components I think you would need to create your own PaaS.

1. Wildcard DNS entry
2. “Web router” (such as Hipache)
3. Docker – Application server/container provisioning and image creation
3. Application source code
5. Environment variables
6. Datastores, such as MySQL, NoSQL, object storage, etc
7. Something to tie them all together

##组件

以下是一些你在构建自己的PaaS时可能会用到的组件。

1. 泛域名解析(Wildcard DNS entry)
2. Web路由器(例如 [Hipache](https://github.com/dotcloud/hipache))
3. Docker - 应用服务器/容器，创建镜像
4. 应用源码
5. 环境变量
6. 数据存储，例如 MySQL、NoSQL、对象存储等
7. 把上边的组件组合在一起的东西

##Wildcard DNS entry

The first thing you need is a wildcard DNS entry for a domain name. This gist describes how to configure one using Namecheap, which happens to also be my registrar of choice (like a few other registrars, but not all, they support two factor authentication). So I bought a domain name like “somedomainapp.com” to run my “apps” and then configured a wildcard entry using Namecheap’s DNS service.

Obviously in a larger production environment you’d either manage your own nameservers or perhaps use something like Google’s DNS as a service, which I would love to try out, and some loadbalancers or similar.

At this point, at minimum, you have a wildcard DNS entry pointing to an IP on your server which apps will be able to use, such as *.yourdomainapp.com.

##泛域名解析

首先，你需要的是对一个域名的泛域名解析。[gist](https://gist.github.com/ngoldman/7287753)是如何使用Namecheap相关配置，Namecheap碰巧是我的注册服务商(其他一些注册管理服务商也类似，但不是所有的，他们支持双因子认证)。所以我购买了一个类似于“somedomainapp.com”这么一个域名来运行我的应用，并且使用一个Namecheap提供的泛域名解析服务。

很显然地是在一个大型的生产环境中，你必须管理自己的域名服务或是使用诸如[Google的DNS服务](https://gist.github.com/ngoldman/7287753)(我喜欢)，或是其他一些诸如负载均衡的设备。

至此，你有了一个泛域名解析到你指定的服务IP地址，例如 *.yourdomainapp.com

##Web router

##Web路由

<img src="https://raw.githubusercontent.com/curtisgithub/curtisgithub.github.com/master/img/somedomain.png"  alt="hipache non-existent domain page" />

(Above: hipache non-existent domain page)

(以上是 hipache 的 non-existent domain 页面)

I’m not sure what to call this layer. Heroku calls this HTTP routing. Web routing works for me.

我不知道改如何称呼这一层，Heroku称它为[HTTP路由](https://devcenter.heroku.com/articles/http-routing)，我想这个名字合适。

Essentially what his does is route incoming requests for apps to the right webserver, which in our case will be a docker container, or several docker containers. A request for someapp.somedomainapp.com would go to 127.0.0.1:49899 or 172.17.0.3:80 and such, which are docker containers.

从本质上讲，它的工作就是将输入请求路由到正确的 web 服务器，在我们的例子中也就是 docker 容器。一个请求了 someapp.somedomainapp.com 的请求可能被送到 127.0.0.1:49899 或是 172.17.0.3:80 或其它，这背后都是docker 容器。

In my situation I am using hipache which is backed by redis. This means you can add routes to hipache by entering them into redis, and hipache isn’t required to restart because it will query redis for domain configuration. By default hipache allows the use of wildcard domains, so it’ll route any configured entry, or send a default page if it doesn’t exist.

在我们的情形中，我使用 hipache ，它的后台使用的 redis。这也就是说你在 hipache 中添加路由也就是把这些规则添加到 redis 里边，并且 hipache 不需要重启，因为它可以查询 redis 以获取域配置。默认情况下 hipache 允许使用通配符域名，所以它可以路由任何请求并且如果目标不存在则发送到默认页面。

My PoC python script, which is called “wpd” (more on that later), can dump the keys setup in redis for hipache. The below output means hipache will randomly balance requests for someapp.yourdomainapp.com to the two containers listed in redis.

我的 PoC Python 脚本被称为“wpd”，它能够打印出在 redis 中存储的 hipache 键配置。以下的输出意味着 hipache 随机将对 someapp.yourdomainapp.com 的请求平均分布到两个容器之中，如下：

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

There are many other ways to do “web routing.” Dokku uses nginx. There is also vulcand which is backed by etcd and though it’s new it sounds exciting. Hipache does support SSL, but as of a few weeks ago Vulcand did not, though I think it’s on the roadmap (but I am a golang fanboy, so am biased).

有其他很多可以做web路由的方法。Dokku 使用 nginx。还有使用 [etcd](https://github.com/coreos/etcd) 的 [vulcand](https://github.com/mailgun/vulcand) 这是个很令人兴奋的新东西。Hipache 支持SSL，就在几周前 Vulcand 还不支持，但我想这肯定是在计划内的，因为我是 golang 的粉丝，所以相对有些偏心 ;)。

##Docker!

Again comparing Heroku to what we are doing here, I think that Docker would fill the roles of buildpack and dyno though perhaps in terms of the buildpack part not containing a “slug” of the application code, rather only the environment the application would run in. Perhaps it’s better to consider a Dockerfile a type of buildpack.

再来比较一下 Heroku 和我们正在做的实情，我认为 Docker 能够扮演 [buildpack](https://devcenter.heroku.com/articles/buildpacks) 和 [dyno](https://devcenter.heroku.com/articles/dynos) 的角色，虽然也许严格说来 buildpack 不包含应用代码，或者更确切的说只有应用运行所需的环境。或许更好地理解是 Dockerfile 是一种 buildpack。

Using my wordpress example, the Dockerfile would create the image which docker then runs to create a wordpress application container, for example using apache2 + php.

用我的 wordpress 作为例子，Dockerfile 文件可以创建 docker 镜像以用来生成一个运行 wordpress 应用的容器，举个例子包含 apache2 + php。

Docker manages the container and provides networking and network address translation to expose the apache2’s port to the web router.

docker 管理容器，并且提供网络以及网络地址转换以将 apache2 的端口暴露给 web 路由。

So Docker is doing quite a bit of the work for us. Without Docker, we would probably need a way to create virtual machines images programmatically, and a way to start up and instance and get it networked, which could be as simple as something like packer and libvirtd (perhaps with kvm or lxc) or other combinations such as packer and openstack. Certainly that would require a few more resources. (Interestingly packer can build docker images as well.)

所以 docker 为我们做了很多事情。没有 docker 的话，我们可能需要写代码以实现一个创建虚拟机镜像的方法，并且还得管理启动、网络、实例化等诸多的实情，这就跟写一个简单的 [packer](http://www.packer.io/) 或 libvirtd (kvm 或 lxc)一样了，就像openstack做的那样。毫无疑问那将耗费更多的资源。(有意思的是 packer 也能够创建 docker 镜像)

##Application source code

##应用源代码

In Dokku code is pushed to a git repository which kicks off other processes. This is also how Heroku works. Those processes somehow get the code into the application container.

在 Dokku 中，代码是被 push 到一个 git 容器中，并启动其他的进程，这也是 Heroku 的做法。这些进程将代码放置到应用容器之中。

However, in my wordpress example, the wordpress code is downloaded in the startup script. Once the container is started from the wordpress image, the startup script runs something like:

然而，在我们 wordpress 的例子中，wordpress 的代码是可通过起始脚本下载的。一旦容器从 wordpress 镜像启动，起始脚本如下运行：

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

to grab the code. The url used to download could be taken from an environment variable instead of just being static as in the example.

看看代码吐吐槽，用来下载的 url 应该从环境变量中获取而不是如上例直接写在代码里边。

The git push/recieve style probably makes more sense in a PaaS, but I haven’t looked into what it takes to do that. Again Jeff Lindsay has gitrecieve and Flynn (a Jeff Lindsay project) has gitrecieved. Also he has execd and more. Busy guy!

git 的 push/receive 风格可能在 PaaS 中更有效，但是我还没有深入去研究那是怎么做到的。Jeff Lindsay 有一个工具[gitreceive](https://github.com/progrium/gitreceive)并且Flynn(另一个 Jeff 的项目)有[gitreceived](https://github.com/flynn/gitreceived)。他还有 [execd](https://github.com/progrium/execd)和其他项目，真是大忙人！

Obviously there are a lot of ways to get code into the container to run it. If there is any one important thing a PaaS does, it’s run your code.

显然，有很多方法可以讲代码放到容器中去执行。如果要说 PaaS 有什么重要的事情，那么就是运行代码了。

##Environment variables

##环境变量

I think Docker images should be fairly generic. Also you don’t want to commit configuration information to the image, such as passwords, so they should come from environment variables, and those variables need to get injected into the containers environment somehow.

我认为 docker 镜像应该会变得相当普及。因此你不会想将敏感的配置信息诸如密码等放到镜像之中，所以它们应该从环境变量中获取，并且这些变量需要通过某种方式注入容器环境中。

For my wordpress example, I set environment variables with Docker. The Docker run command has the “-e” option which allows setting environment variables which will be exposed in the container. Below is an example.

在我的 wordpress 例子中，我设定了 docker 的环境变量。docker 可以用“-e”参数运行命令，以使得设定环境变量的方式暴露出来，来看下面的例子：

```
$ docker run -e FOO=bar -e BAR=foo busybox env
HOME=/
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=6cf2d6e8acb3
FOO=bar
BAR=foo
```

My wordpress startup script checks for a few environment variables

我的 wordpress 启动脚本检查以下几项环境变量：

```
DB_NAME=${DB_NAME:-"wordpress"}
DB_USER=${DB_USER:-"wordpress"}
DB_PASSWORD=${DB_PASSWORD:-"wordpress"}
DB_HOST=${DB_HOST:-$1}
```

and uses them to create the wordpress configuration file with the right database settings.

Later on I’ll talk about “tying the whole room together” which I do with a python script, and in it I set the environment variables for the container.

Below is a snippet of python code used to start a container with environment variables.

并且使用它们和正确的数据库设置创建 wordpress 的配置文件。

稍后我会谈谈“如何将所有部分整合”，我是通过一个 Python 脚本完成的，在这个脚本中，我设置了容器的环境变量。

下面的是 python 代码的一些片段来用环境变量初始化容器：

```
env = {
  'DB_HOST': MYSQL_HOST,
  'DB_NAME': dbname,
  'DB_USER': dbname,
  'DB_PASSWORD': dbpass,
}
container = dockerConn.create_container(image, detach=True, environment=env)
```

Docker and python go well together using docker-py.

Another method is using a shared configuration system. Previously I mentioned etcd.

可以使用 [docker-py](https://github.com/dotcloud/docker-py) 来联合使用 docker 和 python。

另一种方法是是使用共享配置系统，诸如我之前提到的 etcd 。

> [etcd is] a highly-available key value store for shared configuration and service discovery.

> [etcd 是] 一个高可用的键值存储系统，用于共享配置和服务发现。

etcd can store configuration information and confd is a configuration management agent which can query etcd for information and generate configuration files that applications can use, and also restart the services that use those configuration files.

Given that I’m suggesting environment/configuration variables are a key part to PaaS, things like etcd, confd, and consul are going to be important projects. But again, with my limited wordpress PaaS I’m just working on a very simplified PoC, where environment variables come at container runtime. However, I would imagine most larger PaaS or PaaS-like systems will use something like consul or etcd.

etcd 能够存储配置信息。而 [confd](https://github.com/kelseyhightower/confd) 是一个配置管理代理软件，能够查询 etcd 以生成针对应用的配置文件，并且能够使用这些配置文件重启服务。

说了这么多，我认为 环境/配置 变量是 PaaS 的核心部分，诸如 etcd、confd 和 [consul](https://github.com/hashicorp/consul) 都将是重要的项目组件。但是，对于本文所说的 wordpress 例子而言，我们只是做一个简单的验证系统，环境变量从容器运行时中获取。然而，我非常建议大型的 PaaS 或是其他 类-PaaS 系统能够使用 consul 或 etcd 这样的组件。

##Datastores

##数据存储

If your application needs to persist data, then it’s got to put that data somewhere and the application container itself is not a good place. In general, I think there are two approaches:

如果你的应用需要持久化数据，那么将数据放在某些地方就是必然的了，但是使用应用容器来存储显然不是一个好选择。通常来说，我认为有两种解决方案。

1. Another container 
2. A separate service

1. 另一个容器
2. 一个单独的服务

In terms of “a separate service” I’m talking about something like Amazon RDS or OpenStack Trove (both of those being Database as a service) or object storage like OpenStack Swift. In short I mean a service that is managed by someone else, perhaps the same provider that either runs Docker or the server that Docker is running on.

对于“一个单独的服务”而言，我指的是诸如亚马逊 [RDS](http://aws.amazon.com/rds/) 或 [OpenStack Trove](https://wiki.openstack.org/wiki/Trove) (两者都是数据库即服务)或例如 [OpenStack Swift](http://docs.openstack.org/developer/swift/) 那样的对象存储系统。简而言之就是有第三方管理的服务，也许同样的服务提供商也是使用 docker 来提供服务的。

The other option is another container. Again using the wordpress example, instead of running one application container, perhaps I would run a second container that has a MySQL server running (or both services could even be in a single container). Maybe that MySQL service is a container, maybe it’s a hardware server configured with Ansible. Who knows. Docker also recommends the volumes from approach, which works great when the data doesn’t have to be distributed across multiple containers, which would be the case in a MySQL or OpenStack Swift container.

另一个选择是使用“另一个 docker 容器”。再拿 wordpress 来做例子，我可能不只是启动一个应用容器，而是启动第二个包含 MySQL 服务器的容器(或者两个服务运行在一个容器中)。也许 MySQL 的服务是一个容器，也许那是一个通过 Ansible 配置的硬件服务器，谁知道呢。docker 同样也推荐使用卷([volume](https://docs.docker.com/userguide/dockervolumes/))，尤其是当数据不会分布到多个容器中的时候，如果数据分散，那么就是用 MySQL 或是 openstack swift 吧。

My feeling is that either is Ok, but I prefer to run a separate service. So in my wordpress example there is a single MySQL server that all wordpress apps will connect to, each having their own database. Perhaps that separate service is using Docker too.

我认为这几种方式都OK，但是我会选择一个单独的服务。所以在我的例子中，存在一个单独的 MySQL 服务器，所有的 wordpress 应用都会连接它，每个应用都有其自己的数据库。或许这个单独的服务也是用 docker 来完成的。

##Something to tie them all together

##把上面讲的内容都串起来

I’m working on a python script called “wpd” that ties all of this together, and what it does is:

我用一个名为 wpd 的脚本来串起所有环节：

1. Creates a site entry in the wpd MySQL database
2. Creates a database datastore that the site can use for wordpress
3. Creates couple of wordpress containers
4. Provides those containers with environment variables regarding how to connect to a worpress database
5. Adds that site to redis so that hipache can route/loadbalance requests to those containers

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

As you can see it has a few options, such as “addsite” and “deploysite”. It’s not complete yet. Adding a site just enters it into the wpd database, and deploying it means starting up containers and adding the information to redis so that hipache can route http requests to them.

正如你所见，有一些选项，诸如“addsite”和“deploysite”，这还没有完全弄完。添加站点仅仅是将其放到 wpd 数据库中，部署站点意味着启动容器，并且向 redis 添加了信息以便 hipache 能够将 http 请求路由给它们。

What this would look like in a larger system…I’m not sure. It seems like a user management system more than anything else—users have sites, sites have names, containers, images and datastores.

这看起来想是一个大型系统...我不太确定哈。看起来更像是一个用户管理系统，因为用户能够拥有站点、站点能够有名字、容器、镜像和数据存储等。

##Issues

##问题

There are a few issues that I’d like to mention (though I’m probably not covering them all).

这里有几个问题我必须提一提(也可能我没有全概括到)。

Logging – Getting logs out of docker is still a bit of an problem. At this point likely you’ll need to configure syslog in the container to ship logs to a centralized system. I expect that eventually docker will have more advanced ways of dealing with logs, if they don’t already.

日志 - 从docker之外获取日志仍旧无解。所以在这里你可能需要在容器中配置 syslog 将日志记录到一个中心系统中。我期望 docker 这一边能够想办法解决日志的问题。

File systems – Wordpress is a good example of a web application that is difficult to scale because it relies a lot on the file system for data persistence, such as media uploaded by users. In order to scale out a file system across multiple docker hosts you’ll need a distributed file system, which is a huge pain and can also increase the size of your failure domain dramatically. I suggest not using file systems to store files and instead use object storage such as OpenStack Swift, which isn’t as hard to deploy as some might think. What’s more Swift doesn’t think it can do consistency and availability at the same time.

文件系统 - Wordpress 是一个很好的 Web 应用示例，它很难扩展，因为它依赖文件系统做数据存储，例如用户上传的多媒体文件。为了跨多个docker主机扩展文件系统你需要一个分布式的文件系统，这个文件系统必须能够支持动态的扩展，这很令人头疼。所以我建议不用文件系统而是使用对象存储，例如 OpenStack Swift，其实它并不是这么难搭。但是 Swift 并不能同时保证一致性和可用性。

Datastore credentials – I’m not sure what the best way to securely store these is. Credentials and other important configuration information will need to be injected into the container somehow, and thus will need to be stored in a database or etcd or similar. Something to look into.

数据安全 - 我不确定什么是最好的保证数据安全的做法。一些认证或其他重要的配置信息都需要注入容器中，并且需要存储在诸如 etcd 的系统中，可能会被看到。

##Conclusion

In the end, I think that docker is a great system to use as a component in PaaS and that, to me, it simplifies rolling your own small platform as a service host. All you need is a web router, a Dockerfile and a docker host, a way to get the app into the container, and pow now you’re cookin’ with PaaS. Just remember to make your containers fairly generic, pipe in environment variables for configuration (or get them from somewhere else), and avoid file systems if possible.

##小结

在最后，我认为 docker 用来做 PaaS 组件是非常棒的，对我来说，它简化了将自己的小型平台改造成服务提供者的过程。你所需要做的就是 web 路由、dockerfile、 docker主机、将应用放到容器内的方法，准备好之后你就能够做自己的 PaaS 了。请记住将你的容器做的尽量通用，多用环境变量和配置变量(或是从其他什么地方获取到相关信息)，并且尽可能避免使用文件系统。

##The Future!

Obviously I’m leaving a lot out. There’s a huge difference between toying around with a PoC like this and running a production PaaS. I haven’t even mentioned the terms “service discovery” or “container scheduling”, which theoretically things like etcd and libswarm could take care of respectively, though I’m not sure libswarm will turn into a container scheduler. Recently Google released Kubernetes which is as docker cluster manager of some kind, though it currently only runs on GCE. Apache Mesos also working on its Deimos project. Further, CoreOS has fleet. Neither have I discussed limiting resources, such as container memory and CPU and a about a billion other things, but I look forward to learning more.

##未来展望！

很显然我拉下了一些东西。搭建一个验证系统和实际弄一个生产用的 PaaS 还是有非常大的区别的。我还没有提到诸如“服务发现”和“容器调度”，这些偏理论的东西可以用 etcd 或是 libswarm 来帮助你解决，虽然我不清楚 libswarm 是否能做容器调度器。最近 Google 开源了 [Kubernetes](https://news.ycombinator.com/item?id=7873897) - 一个 docker 集群的管理工具，但是它现在仍然只能运行在 GCE 上。Apache 的 Mesos 同样也只能运行在它的 [Deimos](https://github.com/mesosphere/deimos) 项目上。长远来看，CoreOS 有[fleet](https://github.com/coreos/fleet)。同样我也没有谈到诸如资源限制问题，如容器所用内存、CPU等其他的细节，但我正计划深入研究。