# DockerCon 上露脸的开源项目

2014年的 DockerCon 落幕了，看过之后最大的感慨就是：这是一场开源的盛会，以及卖萌的盛会。

在这个为期两天的大会上出了不少开源的好东西，且看我们一一列举：

## Kubernetes 

公司： Google 

https://github.com/GoogleCloudPlatform/kubernetes

容器集群管理工具，基于 GCE 平台。

## cAdvisor

公司： Google

https://github.com/google/cadvisor

全称 Container Advisor ，为运行容器的用户提供出色的资源使用和性能特征。这是一个运行守护进程，能够搜集、集料、处理和导出运行中的容器的信息。特别需要指出，每个容器都有资源隔离参数、历史资源使用、以及完整历史数据的柱状图。

cAdvisor 目前支持 lmctfy 容器和 Docker 容器。

## lmctfy

公司： Google

https://github.com/google/lmctfy

全称 Let Me Contain That For You ，是 Google 自己的容器栈的开源版本。这些容器能够隔离在单个机器上运行的多个应用之间的资源。这些容器能够创建并管理自己的子容器。

lmctfy 同时发布了 C++ 库 和 CLI 。

## dotCI

公司： Groupon

https://github.com/groupon/DotCi

轻松配置诸如 travisci 这样的云 CI 系统，简化运行 docker 到 Jenkins 时的配置、环境和执行时间。

## Centurion

公司： New Relic

https://github.com/newrelic/centurion

为 Docker 开发的部署工具。从 Docker registry 中抓取容器，运行在其它主机上，同时保证环境变量、主机容积映射、端口映射正确。支持滚动部署，也让输出容器到 Docker 服务器更轻松。


## Libcontainer

公司： Docker

https://github.com/docker/libcontainer

一款操作系统沙盒的标准化界面。它使用 Go 实现本地使用 Linux 命名空间、联网和管理，无需外部依赖，也不会对主机系统造成影响。

## Libchan

公司： Docker

https://github.com/docker/libchan

小型、轻量的通讯协议，为分布式计算提供库。 Libchan 支持大量非常规传输，包括 Unix socket 、 Raw TCP 、 TLS 和  HTTP2/SPDY 。

## Libswarm

公司： Docker

https://github.com/docker/libswarm

一套极简的编配和组织网络服务的工具包，适用于分布式系统。 Libswarm 能让用户组织复杂架构，从可重复使用的构建模块，到避免因与其它服务器交换而被厂商锁定。 Libswarm 自带一个包含服务的拓展库，你也可以使用简单的 API 来自己写一个。

