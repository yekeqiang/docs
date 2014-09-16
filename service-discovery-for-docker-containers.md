#Docker 容器的服务发现


#####作者：[Des Drury](https://twitter.com/DesDrury)

#####译者：[Bo Wen](http://weibo.com/u/2537862844)

***
Docker 是一个封装运行时环境和可部署代码的应用程序的奇妙工具，但是它没有提供在分布式架构下容器间交流的功能。 Docker 的 links 功能只能帮助托管在单个节点的容器。一些模式和实现的如雨后春笋般出现弥补这个差距。我将在这篇博客中指出我自己的分布式 Docker 容器解决方案。

我用到的技术有：

**Docker** ： 必须的。

**Etcd** ：一个高可用的为了共享配置和服务发现的键值存储系统。

**Skydns** ：一个用于发现 etcd 的 DNS 服务工具

**HA Proxy** ：一个可靠的，高性能 TCP/IP 负载均衡工具.

**Custom Scripts** ：一些脚本集，用来自动化服务发布，服务注册，服务发现以及自动化配置。

下面的图标为构建这个例子所用的应用栈。

![diagram](http://resource.docker.cn/stack-1.png)

###Docker容器

每个 Docker 容器中都有一些额外的进程。我使用了 Supervisord 作为进程管理工具。下面图表展示了容器的结构。

![diagram](http://resource.docker.cn/docker-container-2.png)


如上所示，其余的容器都从一个基本容器继承而来。基本容器包括了 SSHD 。 Docker 容器通常只运行服务进程，把 SSH 引入进来的目的是为了调试。

announce-service 和 listen-path 的后台进程描述如下：

`announce-service` ：这个脚本发布 Etcd 中提供服务的细节。

`listen-path` ：这个脚本监听一个或者更多的 Etcd 路径并且为每个 listen-service 启动一个实例。

`listen-service` ：每个容器都要运行的脚本。


###发布一个服务

announce-service 能发现一系列的环境变量。如下面这些变量:

`ANNOUNCE_PATH` ： Etcd的路径，服务会将自身的详细信息写到这里。路径前缀为 /announce 。

`PORT` ：容器暴露的服务端口。

`TYPE` ：服务类型，可以是 HTTP 或者 TCP 。

下面是一个 PostgreSQL 容器启动的例子，它将在 Etcd 中发布自己。

```bash
docker run -d -t -P -e ANNOUNCE_PATH=/internal/app1/dev/db/ghost.example -e PORT=5432 -e TYPE TCP --name postgresql registry/postgresql
```

当容器启动时，一个JSON字符将被写入到Etcd中下面的路径。 */announce/internal/app1/dev/db/ghost.example/444a99dbf0c6* 

上面的路径中最后一部分是 Docker 指派给容器的主机名。

服务发布后，它的 TTL 变成15秒。 ```announce-service``` 脚本每隔5秒将尝试去更新路径。如果更新失败，它会自动将路径删除。

下面是写入到路径的 JSON 字符串示例。

```javascript
{"ID":"444a99dbf0c6",
"APP":"app1",
"ENV":"dev",
"SERVICE":"db",
"NAME":"ghost.main",
"PORT":"5432",
"TYPE":"TCP"}
```

`ID` ：Docker指派给容器的主机名。

`APP` ：应用名称。

`ENV` ：环境名称。

`SERVICE` ：服务名称。

`NAME` ：可选的环境名称。后面你会发现，这个变量对于在一个主要的环境中，比如DEV，创建多个命名环境提供了极大的灵活性和便利。

`PORT` ：容器暴露的服务端口。

`TYPE` ：服务类型。

下面的图表展示了服务发布的过程。

![diagram](http://resource.docker.cn/announce-service-2.png)

###注册一个服务

一旦一个服务被发布，它需要立即注册。这个额外的步骤的原因在于容器不知道 Docker 指派给自己的外部 IP 地址或者端口。

在每个包含 Docker daemon 的主机上都运行着 register-service ，它们在同一个 Etcd 集群中。一旦探测到 /announce 路径有修改，容器中运行的主机就会往 /services 路径写入一个新的 JSON 字符串。可以在下面的图中看到这个过程。

![diagram](http://resource.docker.cn/register-service-2.png)


下面的例子里，路径这次用的是 Django 容器。

*/services/internal/app1/dev/web/ghost.example/0ce60d000eac*

和前面一样，服务的注册的 TTL 时间为 15 妙。每次 announce-service 脚本在容器中运行时，将会引起 register-service 脚本重新将服务在 /services 路径注册。

下面是写入到该路径中的一个 JSON 字符串示例。

```javascript
{"ID":"0ce60d000eac",
"APP":"app1",
"ENV":"dev",
"SERVICE":"web",
"NAME":"ghost.example",
"IP":"10.250.136.243",
"EXT_PORT":"49178",
"TYPE":"HTTP"}
```

`IP` ：主机的 eth01 适配器的 IP 地址。

`EXT_PORT` ： Docker 指派给容器暴露出的外部端口。

在 /services 注册服务的同时， register-service 脚本还会在 /skydns 路径下为服务注册一个 DNS 入口。


用 Django 作为例子，下面的路径将被写入。

*/skydns/internal/app1/dev/web/example/ghost*

如下是一个写到路径的实例JSON字符串。

```javascript
{"host":"x.x.x.x"}
```

这将注册下面的 DNS 入口。

```
ghost.example.web.dev.app1.internal
```

注意：在我当前版本的脚本中， DNS 入口的 IP 地址被硬编码为包含 web 前端 HA proxy 容器的 AWS 服务器的动态 IP 地址。在将来的版本中这部分会被修改的更加动态。

###监听一个服务

如果容器运行了 ```listen-path``` 脚本，之后它将会监听一个或者更多的 Etcd 路径更改。路径用如下的环境变量定义。

`LISTEN_PATH` ：逗号分隔的 Etcd 路径集。

下面是一个启动 web 前端 HA Proxy 容器请求 DEV 和 SIT 命名环境中的服务的例子。

```bash
docker run -d -t -p 80:80 -p 2200:22 -e LISTEN_PATH=/services/internal/app1/dev,/services/internal/app1/sit --name web_frontend registry/haproxy
```

![diagram](http://resource.docker.cn/listen-path.png)


这个 ```listen-service``` 脚本用于配置一个本地的 HA Proxy 服务器，监听路径上的 SET 或者 EXPIRE 行为。一旦检测到修改，相关的 JSON 字符串被读入，同时 HA Proxy 配置文件被创建/修改/删除。

下面是启动自我发布和监听 PostgreSQL 容器的 Django 容器示例。

```bash
docker run -d -t -P -e ANNOUNCE_PATH=/internal/app1/dev/web/ghost.example -e PORT=80 -e TYPE=HTTP -e  LISTEN_PATH=/services/internal/app1/dev/db/ghost.example --name django registry/django
```

由于 Django 容器在路径 */internal/app1/dev/web/ghost.example* 上发布自己，它很快就会被提前启动的 HA Proxy 容器检测到， HA Proxy 容器监听的路径之一为 */services/internal/app1/dev* 。

之后， ```launch-service``` 将修改 ```haproxy.cfg``` 文件中的 ```http-in``` 前端代码块，于是 Django 容器发布的内容就被知道了，如下。

```bash
acl ghost.example.web.dev.app1.internal hdr_dom(host) -i ghost.example.web.dev.app1.internal

use_backend ghost.example.web.dev.app1.internal if ghost.example.web.dev.app1.internal
```

`launch-service` 脚本还会创建一个针对具体环境的配置的包含 `backend` 的代码块，如下。

```bash
backend ghost.example.web.dev.app1.internal
  log global
  server 0ce60d000eac 10.250.136.243:49176
```

如果多余的 Django 容器启动发布给相同的路径，那么它们也会被添加到 ```backend``` 代码块，如下。

```bash
backend ghost.example.web.dev.app1.internal
  log global
  server 0ce60d000eac 10.250.136.243:49176
  server 7874ghGD121s 10.250.136.221:49188
```

前面的例子描述了配置 HTTP 服务，这对 TCP 也同样适用。一个例子是 Django 容器在监听它的 PostgreSQL 容器。一旦 PostgreSQL 容器被检测到，它会在 Django 容器的 ```127.0.0.1:5432``` 上变得可用。这意味着 Django 容器不用修改在任何环境中移动。唯一需要修改的是环境变量的值。

###优点

一旦 Docker 容器能够使用服务发现交流，会带来很多优点。

- Docker 容器可以在分布在多个主机

- 通过更改少量的环境变量，同样的 Docker 容器可以在多个环境间切换。

- 一个完全的多层可以在数秒中启动。

-  在一个主环境下可以有多个命名环境。每个开发人员都可以拥有自己的环境或者为每次分支提交都创建一个环境。

Docker 容器的服务发现的问题解决后，下一个需求就是在多个主机间协调容器的启动/停止，这个是 CoreOS 的 Fleet 的功能。

***

#####这篇文章由 [Des Drury](https://twitter.com/DesDrury) 撰写，[Bo Wen](http://weibo.com/u/2537862844) 翻译。点击 [这里](http://desdrury.com/service-discovery-for-docker-containers/) 阅读原文。

#####The article was contributed by [Des Drury](https://twitter.com/DesDrury), click [here](http://desdrury.com/service-discovery-for-docker-containers/) to read the original publication.
