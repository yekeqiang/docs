# Docker发布1.2.0版本，并宣布DockerCon Europe

---

##### 作者：[Carlos Sanchez](http://www.infoq.com/cn/author/Carlos-Sanchez)

##### 译者：[马德奎](http://www.infoq.com/cn/author/%E9%A9%AC%E5%BE%B7%E5%A5%8E)

---

Docker 发布了1.2.0版本，其中包括为容器指定重启策略、容器权限的细粒度控制等特性。该公司还将于今年12月在阿姆斯特丹主持召开它在欧洲的第一次正式会议 DockerCon Europe 。

## 新特性

### 重启策略

Docker 1.2.0新增重启策略特性，允许容器在退出时重启以及从可能发生的容器故障中恢复。

docker run 命令增加了一个新的参数 `—restart` ，用来指定以下三种策略中的一种：

- no ：如果容器宕掉，不自动重启。这是默认行为。
- on-failure ：如果退出代码不是0，重启容器。该参数有一个可以指定最大重启次数的可选项（比如：on-failure：5）。
- always ：不管返回的退出代码是什么，总是重启。

有了该参数，Docker 守护进程上的 `—restart` 参数就废弃了。

例如，使 Redis 在容器退出时无限次尝试重启：

```docker run --restart=always redis```

### 细粒度的容器功能

在先前的版本中，Docker 不建议在生产环境中使用 `—privileged` ，因为它允许容器访问主机资源。在这个版本中，docker run 可以使用 --cap-add 和 —cap-drop 参数控制授予特定容器的功能。

例如，更改容器接口状态：

```docker run --cap-add=NET_ADMIN ubuntu sh -c "ip link eth0 down"```

### 禁止在容器中使用 chown：

```docker run --cap-drop=CHOWN ...```

### 在无特权的容器中挂载设备

容器挂载设备不再需要特权。该版本在 docker run 命令中引入了 `—device` 参数，使容器可以使用特定的设备，而不需要 `—privileged` 参数。

例如，在容器内使用声卡：

```docker run --device=/dev/snd:/dev/snd ...```

### 可写的hosts、hostname和resolve.conf文件

/etc/hosts、/etc/hostname和/etc/resolve.conf文件现在可以在容器运行期间编辑，允许执行像bind那样可能向这些文件写入内容的服务。不过，对这些文件的更改只在运行时有用，在构建容器镜像时并不保留。

## DockerCon Europe 2014

继今年6月举行的 [DockerCon 旧金山大会](http://www.infoq.com/news/2014/06/dockercon2014) 之后，在欧洲组织的第一次正式会议将于12月4日到5日在阿姆斯特丹的 NEMO 科学中心举行。会议演讲者包括 Docker 公司 CEO [Ben Golub](https://twitter.com/golubbe) 和联合创始人兼 CTO [Solomon Hykes](https://twitter.com/solomonstre) 。大会的前一天，[Jérôme Petazzoni](https://twitter.com/jpetazzo) 将主持开展一场“介绍 Docker ”的培训，该课程将介绍 Docker 平台，内容贯穿安装、集成和运行。

另外， Docker 已经宣布 [与 VMware 建立合作伙伴关系](http://blog.docker.com/2014/08/docker-vmware-1-1-3/)，致力于保证 Docker 运行在 VMware 的虚拟解决方案上、创建可互操作的管理工具以及就 Docker 社区的核心技术标准进行协作，尤其是 libcontainer 和 libswarm 的流程互操作技术。

查看英文原文：[Docker Announces Version 1.2.0 and DockerCon Europe](http://www.infoq.com/news/2014/09/docker-1.2-dockercon-eu)

---
本文原载于 [InfoQ] ，我们得到授权后将其转载。您可以阅读 [原始版本](http://www.infoq.com/cn/news/2014/09/docker-investment) 。
