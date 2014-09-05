#Google 云平台对容器支持的最新进展

#####作者：[Eric Brewer](https://twitter.com/eric_brewer) （ Google 基础架构部副总裁 ）

#####译者：[Chris Liu](mailto:chris.z.liu@emc.com) （ EMC 公司软件研发经理）

***

从搜索到 Gmail ， Google 所有的服务其实都封装以及运行在 Linux 容器中。在全球的 Google 数据中心，每周我们会创建超过 20 亿个容器实例，而这些容器为我们提供了更可靠的服务，更有效率的更高的扩展性。现在我们将更进一步，为所有的开发者提供这些新的特性。

##在 Google App Engine (GAE) 中支持 Docker image ##

上个月，我们改进了 [Google Compute Engine](https://developers.google.com/compute/docs/containers) （ Google 云计算引擎） 对 Docker image 的支持。现在，我们基于已有的成果，又扩展了 GAE 对 Docker image 的支持，使得 GAE 开发者可以在 [托管虚拟机](https://developers.google.com/cloud/managed-vms) 上创建和部署 Docker image。开发者利用这些扩展特性，可以方便的访问 Docker 丰富且与日俱增的 image 库。这样，  Docker 社区就可以轻松的将容器部署到托管的虚拟机环境中，并迅速开始访问例如 Cloud Datastore 这样的服务。如果你想尝试这些新特性，请 [填表注册](https://docs.google.com/a/google.com/forms/d/1_RfwC8LZU4CKe4vKq32x5xpEJI5QZ-j0ShGmZVv9cm4/viewform "") 。

## Kubernetes  –  一个开源的容器管理系统 
基于在 Google 内部运行 Linux 容器的经验，我们充分了解到在整个互联网规模上有效地调度管理容器集群的重要性。在 Google 内部我们使用 [Omega](http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/41684.pdf "") 进行集群调度管理，但是对于互联网开发者需要一个更轻巧适度的集群管理系统。正因为如此，我们发布了 Kubernetes ，一个更轻巧却不失强大的开源容器管理系统。 Kubernetes 可以在机群上部署容器集群，提供容器的健康状态管理以及复制功能，这样使得容器之间互联以及对外提供服务更为便捷（满足一下大家的好奇心， Kubernetes 念做koo-ber-nay-tace ，是希腊语里的舵手的意思）。

Kubernetes 从开始就被定位为一个可扩展的基于社区开发支持的项目。你可以在 [GitHub](https://github.com/GoogleCloudPlatform/kubernetes "") 上查看源码和文档，以及通过 [邮件列表](https://groups.google.com/forum/#!forum/google-containers "") 来沟通你的想法。我们将和 Docker 社区合作，持续完善各种功能，将 Kubernetes 里面的各种好的想法并入 Docker 。

## 容器栈的优化 
我们还发布了一个名为 [cAdvisor](http://github.com/google/cadvisor "") 的开源工具，用于提供详细的容器集群资源使用情况。这个工具能够跟踪统计多种资源的实时和历史使用情况，处理嵌套的容器（容器之内的容器）以及支持 Google 的 LMCTFY 容器以及 Docker 的 libcontainer 。 cAdvisor 是使用 Go 语言开发的，这样如果有需要的话，我们可以方便的将这些工具集成进 libcontainer 。

## 对开放式容器标准的承诺
最后，我很荣幸我已经被任命为 Docker 咨询管理委员会的委员，将继续和 Docker 社区一起持续的为开放式容器标准做贡献。容器技术曾是 Google 的基础，我们和 Docker 联手，把容器技术打造为所有云应用的基石。

***

#####这篇文章由 [Eric Brewer](https://twitter.com/eric_brewer) 撰写， [Chris Liu](mailto:chris.z.liu@emc.com) 翻译。点击 [这里](http://googlecloudplatform.blogspot.fr/2014/06/an-update-on-container-support-on-google-cloud-platform.html) 可阅读原文。

#####The article was contributed by [Eric Brewer](https://twitter.com/eric_brewer), click [here](http://googlecloudplatform.blogspot.fr/2014/06/an-update-on-container-support-on-google-cloud-platform.html) to read the original publication.

