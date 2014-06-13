# Monitor Docker with Datadog 使用 Datadog 监控 Docker

Docker is an emerging platform to build and deploy software using lightweight, pared-down virtual machines known as containers. By delivering easy-to-provision recipes for developers and bit-for-bit compatibility between environments, Docker is a popular solution to solve continuous delivery in modern infrastructure.
Docker 是一个构建部署应用的新兴平台，它使用被称之为轻量级虚拟机的容器。通过提供对开发者易于部署和不同环境的兼容，Docker成为在现代基础设施之上解决持续交付问题的流行方案。

Like virtual machines before them, containers require a new monitoring approach. Luckily, if you are a Datadog user, you can now take advantage of our newest integration: Docker.
如何在它们之前的虚拟机一样，这些容器需要一种新的途径来监控。幸运的事，如果你是一个 Datadog 用户，将可以利用 Datadog 对 Docker 的监控集成。

With our Docker integration you can monitor containers by running version 4.3.1 of the Datadog agent. The integration configuration is, like all other agent-based integrations, a simple YAML file.
使用我们对 Docker 的监控集成，你将可以运行 4.3.1 版本的 Datadog 代理来监控容器。而这个监控配置，就如同其他基于代理的集成一样，是一个简单的 YAML 文件。

## How Docker monitoring works Docker 监控是怎么工作的

The simplest way to monitor Docker containers is to run the Datadog Agent on the host, where it can access container statistics. This is especially true if you are deploying Docker on existing, full-fledged Host OSes, along existing applications such as databases.
通过在主机上运行 Datadog 代理是监控 Docker 容器最简单的方式，在这种情况下它可以直接获取容器的数据。当你将 Docker 部署到延用已经存在应用（例如数据库）的主机上时，情况更是如此。

![one](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage1.png)

Since Docker uses existing kernel constructs (namespaces and cgroups) in order to run containers, the Datadog Agent uses the native cgroup accounting metrics to gather CPU, memory, network and I/O metrics of the containers every 15 seconds before they are forwarded to Datadog.
由于 Docker 使用了内核结构（namespaces和cgroups）来运行容器，因此，Datadog 代理可以直接使用cgroup的指标报告每十五秒收集一次CPU、内存、网络和IO数据。

![two](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage2_Screenboard.png)

## Monitor many containers efficiently with tags 使用标签有效监控多个容器

With easy-to-use, lightweight containers, you will likely dial up several times more running containers than the number of underlying physical or virtual hosts in your infrastructure. How do you then keep track and monitor them without spending time chasing after every single one of them? With tags.
凭借易于使用的轻量级容器，你很有可能连接着数倍于底层物理机或虚拟机的容器。那你怎么才可以不为每一个容器单独花费时间而能够追踪和监控它们呢？答案就是使用标签。

Tags are the key to monitoring a lot of containers without additional effort. By default, the agent will monitor your containers and turn the Docker "name", "image" and "command" attributes into a "tag".
标签是不花费额外努力来监控大量容器的关键。默认情况下，代理会监控你的容器，并且将 Docker 的名字、镜像以及命令属性转换成标签。

![three](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage3_tags.png)

## Graph specific metrics with tags 带标签的图像化指标

In Datadog, you define the metrics shown in dashboards and graphs based on one or many tags. This allows you to track specific metrics for many containers in aggregate. Using tags, you can easily create a graph for a metric drawn from all containers running a given image.
在 Datadog 中，你可以基于一个或多个标签来定义显示在仪表盘和图上的指标。这允许你可以全局上追踪众多容器的特定指标。使用标签，可以很轻松地创建一个运行着指定镜像的所有容器的指标图。

In the example below, we are showing the amount of CPU consumed, broken down by image.
在下面的例子中，我们根据镜像划分，显示 CPU 的使用量。

![four](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage4_graph_by_image.png)

## Alerts 警报

Tags are also very useful to define alerts that span clusters of containers. For instance, lt us say that you are running a cluster of Redis containers and you want to be alerted when one of the containers is running out of memory.
标签在定义跨容器集群的警报时非常有用。例如，让我们说你在运行着 Redis 的容器集群，而你想要当其中有一个容器发生内存溢出时接收到警报。

Instead of defining one alert per container, you only have to create a multi-alert on the docker.mem.rss metric and Datadog will trigger an alert if any container misbehaves.
免去每个容器单独定义警报，你只需要在 docker.mem.res 指标上创建一个“多重警报”，这样 Datadog 就能在任何一个容器产生不正常行为时触发警报。

You can also mix and match tags to express more complex conditions. For instance, you can monitor all Redis containers running the redis2.8 image that run on host alq-docker with a simple tag selection:
你也可以混合匹配多个标签来表达更复杂的条件。例如，你可以使用一个简单标签选择监控所有在 alq-docker 主机上运行 redis2.8 镜像的容器。

![five](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage5-multi-alert.png)

## Monitor your containers' lifecycles 监控你的容器生命周期

Since containers are designed to be as short-lived (or long-lived) as traditional OS procsses, it can be very useful to track particular containers throughout their lifecycles.
由于容器时被设计成与传统 OS 进程一般短期存活（或长期存活），那么代理可以用在整个生命周期中追踪特定的容器。

Much like any other meaningful event in your infrastructure, you can search for Docker container create/start/stop/destroy events using the Events Stream. Simply use "sources:docker" as the search filter.
就如同在你的基础设施中其他有意义的事件，你可以使用 Events Stream （事件流）来搜索 Docker 容器创建／启动／停止／销毁事件。只需要使用“source:docker"作为搜索过滤器。

![six](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage6_Events.png)

You can also apply the same search to any TimeBoard to visualize Docker container events in the context of Docker and non-Docker metrics. In the following example, we overlay continers starting and stopping over memory and CPU metrics.
你可以在 Docker 的上下文和非 Docker 指标中应用同样的搜索到 TimeBoard（时间板）来可视化 Docker 容器的事件，在下面的例子中，我们覆盖了容器开始到结束的内存和CPU指标。

![seven](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage7_Correlations.png)

## Explore Docker metrics 浏览 Docker 指标

To explore the Docker metrics that are available, you can use the Metrics Explorer in Datadog and type "docker" in the first drop-down.
想浏览可用的 Docker 指标，你可以使用在 Datadog 中的 Metrics Explorer（指标浏览器），在第一个下拉框中输入“docker”。

![eight](https://www.datadoghq.com/wp-content/uploads/2014/06/DockerImage8_metrics.png)

You can find detailed descriptions about all the metrics in Docker's Runtime Metrics guide.
你可以在 Docker's Runtime Metrics guide（Docker 运行时指标手册）中找到所有指标的详细描述。

If you would like to easily visualize and alert on Docker metrics, try out Datadog for free with a 14-day trial. Metrics for the Docker engine, containers and underlying hosts will be immediately available after installing the Datadog agent.
如果你想要更容易可视化 Docker 指标、接收到相关警报，可以尝试14天免费试用期的 Datadog。在你安装完 Datadog 代理后，你马上可以获得 Docker 引擎、容器以及容器以下的主机的指标。
