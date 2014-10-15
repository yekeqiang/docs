# 深入浅出Docker（四）：Docker的集成测试部署之道

---

##### 作者：[肖德时](http://www.infoq.com/cn/author/%E8%82%96%E5%BE%B7%E6%97%B6)

---

【编者按】Docker是PaaS供应商dotCloud开源的一个基于LXC 的高级容器引擎，源代码托管在 GitHub 上, 基于Go语言开发并遵从Apache 2.0协议开源。Docker提供了一种在安全、可重复的环境中自动部署软件的方式，它的出现拉开了基于云计算平台发布产品方式的变革序幕。为了更好的促进Docker在国内的发展以及传播，我们决定开设《[深入浅出Docker](http://www.infoq.com/cn/dockers)》专栏，邀请Docker相关的布道师、开发人员、技术专家来讲述Docker的各方面内容，让读者对Docker有更深入的了解，并且能够积极投入到新技术的讨论和实践中。另外，欢迎加入InfoQ Docker技术交流群交流Docker的最佳实践，QQ群号：365601355。

## 1. 背景

敏捷开发已经流行了很长时间，如今有越来越多的企业开始践行敏捷开发所提倡的以人为中心、迭代、循序渐进的开发理念。在这样的场景下引入Docker技术，首要目的就是使用Docker提供的虚拟化方式，给开发团队建立一套可以复用的开发环境，让开发环境可以通过Image的形式分享给项目的所有开发成员，以简化开发环境的搭建。但是，在没有Docker技术之前就已经有类如Vagrant的开发环境分发技术，软件开发者一样可以创建类似需求的环境配置流程。所以在开发环境方面，Docker技术的优势并不能很好的发挥出来。笔者认为Docker的优点在于可以简化CI（持续集成）、CD（持续交付）的构建流程，让开发者把更多的精力用在开发上。

每家公司都有自己的开发技术栈，我们需要结合实际情况对其进行持续改进，优化自己的构建流程。当我们准备迈出第一步时，我们首先要确立一张构建蓝图，做到胸有成竹，这样接下来的事情才会很快实现。

![alt](http://resource.docker.cn/timechart.png)

这张时序图概括了目前敏捷开发流程的所有环节。结合以上时序图给出的蓝图框架，本文的重点是讲解引入Docker技术到每个环节中的实践经验。

## 2. 创建持续发布的团队

开发团队在引入Docker技术的时候，最大的问题是没有可遵循的业界标准。大家常常以最佳实践为口号，引入多种工具链，导致在使用Docker的过程中没有侧重点。涉及到Docker选型，又在工具学习上花费大量时间，而不是选用合适的工具以组建可持续发布产品的开发团队。基于这样的场景，我们可以把“简单易用”的原则作为评判标准，引入到Docker技术工具选型的参考中。开发团队在引入Docker技术的过程中，首先需要解决的是让团队成员尽快掌握Docker命令行的使用。在熟悉了Docker命令行之后，团队需要解决几个关键问题具体如下：


1）Base Image的选择, 比如 [phusion-baseimage](https://phusion.github.io/baseimage-docker/)

2）配置管理Docker镜像的工具的选择，比如 [Ansible](http://www.ansible.com/home) 、 [Chef](http://www.getchef.com/chef/) 、 [Puppet](http://puppetlabs.com/)

3）Host主机系统的选择，比如 [CoreOS](https://coreos.com/) 、 [Atomic](http://www.projectatomic.io/) 、 [Ubuntu](http://docs.docker.com/installation/ubuntulinux/)

**Base Image** 包括了操作系统命令行和类库的最小集合，一旦启用，所有应用都需要以它为基础创建应用镜像。Ubuntu作为官方使用的默认版本，是目前最易用的版本，但系统没有经过优化，可以考虑使用第三方有划过的版本，比如如phusion-baseimage。对于选择RHEL、CentOS分支的Base Image，提供安全框架SELinux的使用、块级存储文件系统devicemapper等技术，这些特性是不能和Ubuntu分支通用的。另外需要注意的是，使用的操作系统分支不同，其裁剪系统的方法也完全不同，所以大家在选择操作系统时一定要慎重。

**配置管理 Docker 镜像的工具**主要用于基于Dockerfile创建Image的配置管理。我们需要结合开发团队的现状，选择一款团队熟悉的工具作为通用工具。配置工具有很多种选择，其中 [Ansible](http://www.ansible.com/home) 作为后起之秀，在配置管理的使用中体验非常简单易用，推荐大家参考使用。

**Host 主机系统**是Docker后台进程的运行环境。从开发角度来看，它就是一台普通的单机OS系统，我们仅部署Docker后台进程以及集群工具，所以希望Host主机系统的开销越小越好。这里推荐给大家的Host主机系统是 [CoreOS](https://coreos.com/) ，它是目前开销最小的主机系统。另外，还有红帽的开源 [Atomic](http://www.projectatomic.io/) 主机系统，有基于 [Fedora](http://www.projectatomic.io/download/) 、 [CentOS](http://www.projectatomic.io/blog/2014/06/centos-atomic-host-sig-propposed/)、 [RHEL](http://rhelblog.redhat.com/2014/07/10/going-atomic-with-the-red-hat-enterprise-linux-7-high-touch-beta/) 多个版本的分支选择，也是不错的候选对象。另外一种情况是选择最小安装操作系统，自己定制Host主机系统。如果你的团队有这个实力，可以考虑自己定制这样的系统。

## 3. 持续集成的构建系统

当开发团队把代码提交到Git应用仓库的那一刻，我相信所有的开发者都希望有一个系统能帮助他们把这个应用程序部署到应用服务器上，以节省不必要的人工成本。但是，复杂的应用部署场景，让这个想法实现起来并不简单。

首先，我们需要有一个支持Docker的构建系统，这里推荐 [Jenkins](http://jenkins-ci.org/) 。它的主要特点是项目开源、方便定制、使用简单。Jenkins可以方便的安装各种第三方插件，从而方便快捷的集成第三方的应用。

![alt](http://resource.docker.cn/jenkins.png)

通过Jenkins系统的Job触发机制，我们可以方便的创建各种类型的集成Job用例。但缺乏统一标准的Job用例使用方法，会导致项目Job用例使用的混乱，难于管理维护。这也让开发团队无法充分利用好集成系统的优势，当然这也不是我们期望的结果。所以，敏捷实践方法提出了一个可以持续交付的概念 [DeploymentPipeline](http://martinfowler.com/bliki/DeploymentPipeline.html)（管道部署）。通过Docker技术，我们可以很方便的理解并实施这个方法。

Jenkins的管道部署把部署的流程形象化成为一个长长的管道，每间隔一小段会有一个节点，也就是Job，完成这个Job工作后才可以进入下一个环节。形式如下：

![alt](http://resource.docker.cn/deployment-pipeline.png)

image source: google image search

大家看到上图中的每一块面板在引入Docker技术之后，就可以使用Docker把任务模块化，然后做成有针对性的Image用来跑需要的任务。每一个任务Image的创建工作又可以在开发者自己的环境中完成，类似的场景可以参考下图：

![alt](http://resource.docker.cn/scene.png)

image source: google image search

所以，使用Docker之后，任务的模块化很自然地被定义出来。通过管道图，可以查看每一步的执行时间。开发者也可以针对任务的需要，为每一个任务定义严格的性能标准，已作为之后测试工作的参考基础。

## 4.最佳的发布环境

应用经过测试，接下来我们需要把它发布到测试环境和生产环境。这个阶段中如何更合理地使用Docker也是一个难点，开发团队需要考虑如何打造一个可伸缩扩展的分发环境。其实，这个环境就是基于Docker的私有云，更进一步我们可能期望的是提供API接口的PaaS云服务。为了构建此PaaS服务，这里推荐几款非常热门的工具方便大家参考，通过这些工具可以定制出企业私有的PaaS服务。

### 4.1 [Apache Mesos](https://mesosphere.io/2013/09/26/docker-on-mesos/) + [marathon](https://mesosphere.github.io/marathon/)

Apache Mesos系统是一套资源管理调度集群系统，生产环境使用它可以实现应用集群。此系统是由Twitter发起的Apache开源项目。在这个集群系统里，我们可以使用Zookeeper开启3个Mesos master服务，当3个Mesos master通过zookeeper交换信息后会选出Leader服务，这时发给其它两台Slave Messos Master上的请求会转发到Messos master Leader服务。Mesos slave服务器在开启后会把内存、存储空间和CPU 资源信息发给Messos master。Mesos是一个框架，在设计它的时候只是为了用它执行Job来做数据分析。它并不能运行一个比如Web服务Nginx这样长时间运行的服务，所以我们需要借助marathon来支持这个需求。marathon有自己的REST API，我们可以创建如下的配置文件Docker.json：

```
{
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "libmesos/ubuntu"
    }
  },
  "id": "ubuntu",
  "instances": "1",
  "cpus": "0.5",
  "mem": "512",
  "uris": [],
  "cmd": "while sleep 10; do date -u +%T; done"
}
```

然后调用

```
curl -X POST -H "Content-Type: application/json" http://<master>:8080/v2/apps -d@Docker.json
```

我们就可以创建出一个Web服务在Mesos集群上。对于Marathon的具体案例，可以参考 [官方案例](https://mesosphere.github.io/marathon/) 。

![alt](http://resource.docker.cn/marathon.png)

image source: google image search

### 4.2  [Google Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)

Google的一个容器集群管理工具，它提出两个概念：

- Pods，每个Pod是一个容器的集合并部署在同一台主机上，共享IP地址和存储空间，比如Apache，Redis之类分为一组容器集合。
- Labels，提供服务标签，方便Pod容器之间的调用协作。

通过官方 [架构设计](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/DESIGN.md) 文档的介绍，可以详细的了解每个组件的设计思想。这是目前业界唯一在生产环境部署经验的基础上推出的开源容器方案，可以预见到未来会成为容器管理系统的行业参考标准。

![alt](http://resource.docker.cn/kubernetes.png)

image source: google image search

### 4.3  [Panamax](http://panamax.io/)

在琳琅满目的集群管理工具面前，如何管理单机的Docker容器也是一个需要解决问题。因为Docker占用内存小，在单机服务器上部署成百上千个容器也不足为奇。Panamax提供人性化的Web管理界面用来安装软件让部署变得更简单。并且，Panamax还提供丰富的 [容器模板](https://github.com/CenturyLinkLabs/panamax-contest-templates) ，让在线创建服务成为可能。比如到DigitalOcean申请一台主机，安装一套Panamax启动为后台服务。然后通过Panamax Web界面安装Nginx、Mysql、Redis等服务镜像，这样可以快速搭建生产环境的应用场景。所有的操作都是在Web界面上完成，开发者只需要关注开发本身即可。

![alt](http://resource.docker.cn/panamax.png)

## 5. 结论

Docker的集成部署方案，是一套灵活简单的工具集解决方案。它克服了之前集群工具复杂、难用的困境，使用统一的Docker应用容器的概念部署软件应用。通过引入Docker技术，开发团队在面对复杂的生产环境中，可以结合自己团队的实际情况，定制出适合自己基础架构的配套软件发布方案。

## 6. 作者简介

肖德时, Red Hat Engineering Service/HSS 内部工具组Team Leader. Nodejs开源项目nodejs-cantas Lead Developer。擅长企业内部工具的设计以及实现。开源课程Rails Starter的发起人。rubygem: lazy_high_charts的Maintainer。twitter账号：xds2000，邮箱：xiaods@gmail.com

## 7.下期预告

Docker在生产环境的应用实践已经给大家介绍，下期我将给大家介绍如何基于Docker快速构建开发环境，敬请期待！

感谢 [郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE) 对本文的审校和策划。

---

本文原载于作者在 InfoQ 的专栏，我们在得到作者与 InfoQ 的授权后将其转载。
