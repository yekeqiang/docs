
##New Relic开源Docker部署工具Centurion

>注:作者[Carlos Sanchez](http://www.infoq.com/author/Carlos-Sanchez), 原文发于[infoQ](http://www.infoq.com/news/2014/06/docker-deployment-tool-centurion)

[New Relic](http://newrelic.com/)开源了Centurion，一个用于在内部产品环境中运行的部署工具。[Centurion](http://github.com/newrelic/centurion)从Docker registry拿到容器，用正确的环境变量，主机容量映射，端口映射将它们运行在一队主机上，支持容器外的滚动部署。

Karl Matthias，New Relic的网站工程经理，在[DockerCon期间宣布](https://twitter.com/relistan/status/476166527138279425)以MIT license的授权形式将Centurion开源。项目以Ruby gem的形式发布并且读取一个配置文件，该配置文件用类似Rake语法的内建DSL实现，不过很快它就会支持从etcd中读取配置。Centurion的DSL包括定义部署的镜像，每个容器的容量和端口的指令，同时它还支持定义多个环境比如Staging和production。

Centurion包括多个任务,用于分布式容器环境:


- 滚动部署到一队Docker服务器```rolling_deploy```:每次启动和停止一个容器，保证应用在负载均衡服务的可用性。随着部署的进行，每个容器都会受到健康检查，以保证应用正确启动。默认情况下，它会向应用的根路径发出一个GET请求，并期望endpoint能返回有效的20x状态码。

- 部署到一队Docker服务器```deploy```: 硬停机，然后在所有指定的主机上启动容器。对于endpoint始终要可用的应用服务来说，不推荐使用。

- 在主机上部署一个bash命令行```deploy_console```: 在容器中已存在的环境中启动一个命令行shell。Dockerfile中的```CMD```指令被替换为```/bin/bash```，使用主机列表中第一个主机。

- 列出为特定项目运行的服务器的所有标签```list:running_container_tags```: 返回服务运行机器的所有当前标签列表。给出所有主机的唯一标签列表，对于验证部署的状态很有用，以防在部署过程中出问题。

- 列出当前项目所有运行的容器```list:running_containers```: 返回配置中每个Docker服务器上运行项目的所有容器的列表。

- 列出registry镜像```list```: 返回项目registry上所有的镜像。

该项目正在添加一些新的功能，如为了配置和服务发现集成etcd，证书验证或服务器池的动态主机分配。
