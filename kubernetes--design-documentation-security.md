# Google Kubernetes 设计文档之安全篇

---

作者：Google Kubernetes

译者：何思玫

---

【摘要】：Kubernetes 是 Google 开源的容器集群管理系统，构建于 Docker 之上，为容器化的应用提供资源调度、部署运行、服务发现、扩容缩容等功能。 CSDN 联合浙江大学 SEL 实验室共同翻译其设计文档，本文为系列的第一篇：安全。

【编者按】Kubernetes 是 Google 开源的容器集群管理系统。它构建于 Docker 技术之上，为容器化的应用提供资源调度、部署运行、服务发现、扩容缩容等整一套功能，本质上可看作是基于容器技术的 mini-PaaS 平台。为帮助国内开发者了解 Kubernetes 技术， CSDN 联合浙江大学 SEL 实验室共同翻译 Kubernetes 的系列设计文档，本文为系列的第一篇：安全。

## 设计目标
 
本文讲述了 Kubernetes 的容器、 API 和基础设施在安全方面的设计原则。

- 保证容器与其运行的宿主机之间有明确的隔离；
- 限制容器对基础设施或者其它容器造成不良影响的能力；
- 最小特权原则——限定每个组件只被赋予了执行操作所必需的最小特权，由此确保可能产生的损失达到最小；
- 通过清晰地划分组件的边界来减少需要加固和加以保护的系统组件数量。

## 设计要点

#### 将 etcd 中的数据与 minion 节点和基础设施进行隔离

在 Kubernetes 的设计中，如果攻击者可以访问 etcd 中的数据，那么他就可以在宿主机上运行任意容器，获得存储在 volumes 或者 pods 的任何受保护信息（比如访问口令或者作为环境变量的共享密钥），通过中间人攻击来拦截和重定向运行中的服务流量，或者直接删除整个集群的历史信息。

**Kubernetes设计的基本原则是，对etcd中数据的访问权限应该只赋予某些特定的组件，这些组件或者需要对系统有完整的控制权，或者对系统服务变更请求能执行正确的授权和身份验证操作**。将来，etcd 会提供粒度访问控制，但这样的粒度要求有一个管理员能够深刻理解 etcd 中存储数据的schema，并按照schema设置相应的安全策略。管理员必须能够在策略层面上保证 Kubernetes 的安全性，而非实现层面；另外，随着时间推移，数据的 schema 可能产生变化，这样的状况应该被预先考虑以免造成意外的安全泄漏。

**Kubelet和Kube Proxy**都需要与它们特定角色相关的信息——对于 Kubelet ，需要的是运行的pods集合的信息；对于 Kube Proxy ，需要用以负载均衡的服务与端点集合信息。 Kubelet 同样需要提供运行的 pods 和历史终止数据的相关信息。 Kubelet 和 Kube Proxy 用于加载配置的方式是“ wait for changes ”的 HTTP 请求。因此，限制 Kubelet 和 Kube Proxy 的权限使其只能访问对应角色所需的信息是可行的。

**Replication controller 和其他 future controller的controller manager 经过用户授权可以代表其执行对 Kubernetes 资源的自动化维护**。 Controller manager 访问或修改资源状态的权限应该被严格地限定在它们特定的职责范围之内，而不能访问其他无关角色的信息。例如，一个 replication controller 只需要如下权限：创建已知 pods 配置的副本，设定已经存在的 pods 的运行状态，或者删除它创建的已存在的 pods ；而不需要知道 pods 的内容或者当前状态，亦不需要有访问挂载了 volume 的 pods 中的数据的权限。

Kubernetes pod scheduler 负责从 pod 中读取数据并将其注入pod所在集群的 minion 节点中。它需要的最低限度的权限有，查看 pod ID（用以生成 binding ）、 pod 当前状态、分配给 pod 的资源信息等。 Pod scheduler 不需要修改 pods 或查看其它资源的权限，只需要创建 binding 的权限。 Pod scheduler 不需要删除 binding 的权限，除非它接管了宕机机器上原有组件的重定位工作。在这样的情况下， scheduler 可能需要对用户或者项目容器信息的读取权限来决定重定位 pod 的优先位置。

原文链接：[Security in Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/design/security.md)（编译/何思玫 审校/孙宏亮 责编/周小璐）

译者简介：何思玫，浙江大学SEL实验室硕士研究生，利用业余时间编译了一些开源技术文章，希望对读者有所帮助。

---

转载来源：[CSDN](http://www.csdn.net/article/2014-12-10/2823062#0-tsina-1-34131-397232819ff9a47a7b7e80a40613cfe1) 

