# 基于 Docker 构建 Flume   --   Part 1

##### 作者：[Alex Wilson](https://twitter.com/pr0bablyfine)

##### 译者：[叶可强](http://weibo.com/1224591704)

***

在 Unruly ，我们使用 [Apache Flume](https://flume.apache.org/) 作为事件流架构的一部分。在源端和 sinks 端，它使非常容易建立以及丢弃的。在我的创新时间，我尝试创立一些 Flume 技术来获得 Docker 和集装箱运输的知识。

## 建立一个基础镜像

Docker 有镜像的概念，在这个镜像中我们能运行一个容器。因此，第一步就是创建一个预安装了 Flume 的镜像。Flume 仅仅依赖 java （它自身也是一个 java 项目），我在一个 Ubuntu 基础镜像的基础上创建了它，创建它需要执行以下步骤：

 - 安装 java 和 wget
 - 下载和解压 flume 工程到 /opt/flume 目录下
 - 设置 JAVA_HOME 和把 flume-ng 添加进 PATH 

如下所做，创建一个 dockerfile ：

```
FROM ubuntu

# install wget + java
RUN apt-get update -q
RUN DEBIAN_FRONTEND=noninteractive apt-get install \
  -qy --no-install-recommends \
  wget openjdk-7-jre

# download and unzip Flume
RUN mkdir /opt/flume
RUN wget -qO- \
  https://archive.apache.org/dist/flume/stable/apache-flume-1.4.0-bin.tar.gz \
  | tar zxvf - -C /opt/flume --strip 1

# set environment variables
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
ENV PATH /opt/flume/bin:$PATH
```

通过这个 Dockerfile 构建一个镜像（使用 ```docker build -t flume .``` ），将给我们一个基础镜像，以使 Flume 容器可用。你可以在 [Docker index](https://index.docker.io/u/probablyfine/flume/) 找到它。

## 一个基础的 Flume  拓扑

一个 Flume 拓扑由 agent 组成，它包括3个核心概念：sources 、channels 和 sinks。

我们从 sources 接收数据，把它放入一个或是多个的 channel 里，被 sinks 读取和加工。大部分的基础拓扑由一个节点组成，我们建立了以下一个由 Docker 创建的节点，使用：

 - 一个 NetcatSource ，从一个端口读取数据并且变成事件
 - 一个 MemoryChannel ，存放在内存中的事件 buffering 。
 - 一个 LoggerSink ，记录它接收到的事件
 
这个拓扑的配置文件，我们称之为 flume-example.conf 。配置文件如下：

```
docker.sinks = logSink
docker.sources = netcatSource
docker.channels = inMemoryChannel

docker.sources.netcatSource.type = netcat
docker.sources.netcatSource.bind = 0.0.0.0
docker.sources.netcatSource.port = 44444
docker.sources.netcatSource.channels = inMemoryChannel

docker.channels.inMemoryChannel.type = memory
docker.channels.inMemoryChannel.capacity = 1000
docker.channels.inMemoryChannel.transactionCapacity = 100

docker.sinks.logSink.type = logger
docker.sinks.logSink.channel = inMemoryChannel
```

我们使用这个配置文件创建一个新容器，并且启动 docker agent 。

```
FROM probablyfine/flume

ADD flume-example.conf /var/tmp/flume-example.conf

EXPOSE 44444

ENTRYPOINT [ "flume-ng", "agent",
  "-c", "/opt/flume/conf", "-f", "/var/tmp/flume-example.conf", "-n", "docker",
  "-Dflume.root.logger=INFO,console" ]
```

将 ```ENTRYPOINT``` 块中的 ```flume-ng``` 的命令在一个启动的容器中运行（需要配置文件的目录、配置文件、 以及 agent 名称），并且 ```EXPOSE``` 指令使得端口在运行期是可用的。 NetcatSource 在监听这个端口。

一旦我们创建了这个新镜像（我们叫做 flume-example），我们就可以通过使用命令 ```docker run -p 444:44444 -t flume-example``` 启动该容器。 ```p 444:44444``` 指令将让容器中的 ```4444``` 端口和本机的 ```444``` 端口做映射。现在我们可以通过 ```echo foo bar baz | nc localhost 444``` 给它发送消息了，然后查看被记录的事件。

```
...
2014-05-05 19:26:13,218 (SinkRunner-PollingRunner-DefaultSinkProcessor)
  [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:70)]
  Event: { headers:{} body: 66 6F 6F 20 62 61 72 20 62 61 7A foo bar baz }
...
```

简直酷毙了！现在我们有一个运行良好的 Flume agent 来获取和处理数据了。

下一篇文章，我将展示更多的有趣的 Flume 技术，以及介绍如何轻松地把 Docker 的特性（比如共享一个卷和只读的挂载）和 Flume 整合起来。

***

##### 这篇文章由 [Alex Wilson](https://twitter.com/pr0bablyfine) 撰写， [叶可强](http://weibo.com/1224591704) 翻译。点击 [这里](http://probablyfine.co.uk/2014/05/05/using-docker-with-apache-flume-1/) 可阅读原文。

##### The article was contributed by [Alex Wilson](https://twitter.com/pr0bablyfine), click [here](http://probablyfine.co.uk/2014/05/05/using-docker-with-apache-flume-1/) to read the original publication.
