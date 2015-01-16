# Flocker 0.3.1 新特性 - 兼容 Fig 配置文集，及支持多个云平台

---

##### 作者： Rob Haswell

##### 译者： Fiona Feng

---


> 译者按：ClusterHQ 发布了 Flocker 0.3.1 ，集成了两大功能，以下是对官方博客文章的全文翻译。


自从8月份发布了 Flocker ，我们一直忙于添加新功能，从而为运维团队提供他们在 Docker 容器内运行数据库、队列、键值存储及其它服务时所需的工具。今天我们高兴地宣布 Flocker 0.3.1 引入两个新功能。


## 兼容 Fig 配置文集

我们非常喜欢 Fig ，它是 Docker 的一款轻松上手的工具，用来创建隔离的部署环境。 Fig 能让开发者将自己的应用描述为一套关联的容器，然后部署在单个节点，用于测试和部署。 Fig 使用的 YAML 格式易于理解，与 Flocker 自己的应用部署格式非常接近。


Docker 正在努力将 Fig 的理念与 Docker 紧密结合，因此我们决定兼容 Fig 文集，这样开发者们无需掌握两种区别不大的方法来描述一个复杂应用。

Here is an example of a Fig yml file for an ElasticSearch-Logstash-Kibana application:

以下是 Fig yml 文件应用于 ElasticSearch-Logstash-Kibana 应用的示例：


```
elasticsearch:
  image: clusterhq/elasticsearch
  ports:
   - "9200:9200"
  volumes:
   - /var/lib/elasticsearch
logstash:
  image: clusterhq/logstash
  ports:
   - "5000:5000"
  links:
   - elasticsearch:es
kibana:
  image: clusterhq/kibana
  ports:
   - "80:8080"
```
   
阅读更多：[Using Fig and Flocker to build, test, deploy and migrate multi-server Dockerized apps](https://clusterhq.com/blog/fig-flocker-multi-server-docker-apps/).

## 支持多个云平台


尽管我们继续积极完善 Flocker，不过仍建议不要将其应用于生产环境。许多人要求我们demo展现的“在 Vagrant 环境外部安装 Flocker ”的操作指南。

从现在起，你可以在 [AWS](http://docs.clusterhq.com/en/0.3.1/gettingstarted/installation.html#using-amazon-web-services) 、 [Rackspace](http://docs.clusterhq.com/en/0.3.1/gettingstarted/installation.html#using-rackspace) 和 [DigitalOcean](http://docs.clusterhq.com/en/0.3.1/gettingstarted/installation.html#using-digital-ocean) 上安装 Flocker，将 Fedor20 用作宿主操作系统。希望使用别的供应商或者操作系统，[提交建议让我们知晓](https://github.com/ClusterHQ/flocker/issues)。

对多个云平台的支持最酷的一点就是你现在能够使用 Flocker 在整个数据中心或者供应商之间迁移数据库和别的状态服务，而不仅仅是同一供应商的不同服务器之间。请注意：由于 Flocker 仍处于早期开发阶段，我们的网络代码可能不能很好地应对不稳定的互联网连接。随着 Flocker 能用于生产，在数据中心和供应商之间迁移数据库将会实现真正的应用移植，避免被锁死。


我们提供完整的操作指南，方便用户在 [AWS](http://docs.clusterhq.com/en/0.3.1/gettingstarted/installation.html#using-amazon-web-services) 、 [Rackspace](http://docs.clusterhq.com/en/0.3.1/gettingstarted/installation.html#using-rackspace) 和 [DigitalOcean](http://docs.clusterhq.com/en/0.3.1/gettingstarted/installation.html#using-digital-ocean) 上安装 Flocker ，同时执行跨云平台的容器迁移。

That’s it for the new features of Flocker 0.3.1. We’d love your feedback on what else you’d like to do with Flocker. Talk to us on #clusterhq on freenode.net, the Flocker users google group, or submit an issue on Github.

以上就是 Flocker 0.3.1 的新特性。我们希望得到你的反馈，你希望 Flocker 能做什么。请在 freeonde.net 的 #clusterhq 频道和我们交谈，或使用 [Flocker 用户邮件列表](https://groups.google.com/forum/#!forum/flocker-users) 和我们交流，你也可以在 [GitHub](https://github.com/ClusterHQ/flocker) 上提交错误报告。

---

本文翻译自 [ClusterHQ](https://clusterhq.com/blog/flocker-0-3-1/) 的官方博客，原文地址：[Flocker 0.3.1 - Docker Fig support & multi-cloud deployments](https://clusterhq.com/blog/flocker-0-3-1/)。

本文由 Fiona Feng 为 [docker.cn](www.docker.cn) 翻译，如需转载请注明出处。 