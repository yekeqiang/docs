#监控 Docker -- 第二部分

#####作者：[Logscape](http://go.docker.com/e/44082/logscape/756b/21481063)
#####译者：[陈菊英](http://weibo.com/u/1716255775)

***

Docker 发展迅速，功能日趋完善，但是 Docker 监控却刚刚起步。大多数已有的监控解决方案只能监控 container 资源的基本使用情况，它会告诉你在 container 层级用了多少内存、 CPU 和硬盘资源。这些功能虽然有用，但随着部署规模变大时，功效也捉襟见肘。在 Docker 系列的第二部分中，我们一起来看看还可以用哪些属性来更好的监控 Docker 。

![alt](http://resource.docker.cn/selection-552-300x129.png)

在 [第一部分](http://blog.logscape.com/2014/06/monitoring-docker-mongo-cluster-part-1/) 中，我们已经演示过了用 docker 和 netcat 命令从 Logscape 中获取数据，以此监控 docker 的资源使用情况。大规模部署的时候需要更多更详细信息。想象一下这个场景，你用了大量流行的技术，运行 Nginx 、 MongoDB 、 HAProxy 和 Node.js 等多个服务。如果用一个 image 生成多个 container ，不同的 container 运行同一个应用的不同版本，监控它们就会比较困难。

我们需要把部署信息等联系起来，构造出可管理的基础架构视图。我们需要进程 ID 、它所属的 container ID 、基础 image 名称。我们获取到这些信息和其关联的度量指标，我们就能控制大规模的部署，不至于手足无措了。下面是我们数据的一个表格视图：
```
pid | command |  containerID |  image |
----------------------------------------------
1334 |java   | 23423abc23 | risk/appserver:1.2
1331 |java   | 23423dbc33 | risk/appserver:1.2
1331 |nginx  | 23423dbc33 | risk/nginx:0.9
1334 |mongod | 20023dbc33 | risk/mongo-repl:0.5
1882 |mongod | 13423dbc33 | risk/mongo-repl:0.5
1222 |mongod | 13023dbc33 | risk/mongo-repl:0.5
1332 |java   | 21123abc23 | risk/appserver:1.2
```

##获取 container 的 PID

Docker 采用 cgroups 来隔离资源。要想知道一个进程是否采用了 Docker控制组 ，在 proc 文件系统中打开下面的文件：
```
/proc/$PID/cgroup
```

还可以用ps命令来显示进程表中每个pid的控制组，类似于下面的命令：
```
ps-wweouname,pid,cgroup
```

把 *docker ps* 运行的结果和 container ID 与 image 对应的 pid 等关联起来，用任何脚本语言来实现都并不是很困难。DockerApp 采用的 dockerpids.groovy 脚本就是干这事的，下面是一个示例：

```
16-Jul-201414:54:52 BST, CID=31dc0a17d71d,image=logscape/forwarder:latest,names=trusting_hawking,cmd=/bin/sh,pcpu=0.0,pmem=0.0,rss=2040,comm=bash
16-Jul-201414:54:52 BST, CID=e93abb421dda,image=logscape/forwarder:latest,names=insane_hoover,cmd=bash,pcpu=0.0,pmem=0.0,rss=24112,comm=java
16-Jul-201414:54:52 BST , CID=e93abb421dda,image=logscape/forwarder:latest,names=insane_hoover,cmd=bash,pcpu=0.5,pmem=0.2,rss=124876,comm=java
```

该脚本输出 pid 、资源利用率和 Docker 相关的元信息。

##搜索

下面这个部分和这个系列的最后一个部分都会详细的研究 DockerApp-1.0 。它提供的服务是基于 [第一部分](http://blog.logscape.com/2014/06/monitoring-docker-mongo-cluster-part-1/) 使用的命令，还有从 cgroups 和 proc 文件系统收集信息的脚本。采用上面展示过的示例信息，我们来看看几个搜索的结果。

**找出系统中哪个容器占用了最多的内存，是非常有意思的事情。下面的搜索语句给出了 container ID 占用的平均内存。**
```
| _type.equals(d_pids) rss.avg(CID,rss) chart(line-zero)
```

搜索结果还会展示内存是如何在不同 image 之间分布的。

我们能看到 container：[e93abb421dda](https://bitbucket.org/logscape/blog-docker/commits/e93abb421dda) 和 [31dc0a17d71d](https://bitbucket.org/logscape/blog-docker/commits/31dc0a17d71d) 在 1345 附近开始，然后就迅速提升。这两个 container 属于 image：[logscape/fowarder](https://bitbucket.org/logscape/blog-docker/commits/e93abb421dda) ，都运行了相对比较大型的 Java 程序。下一个搜索语句会给出我们系统中运行的所有进程的 heatmap 表：
```
 | _type.equals(d_pids) comm.by(pid,command) rss.max(pid,rss)  cpuPct.max(pid,cpu)  memPct.max(pid,mem)  image.by(pid,)  CID.by(pid,) chart(table) buckets(1)
```

![alt](http://resource.docker.cn/docker-mem.png)

我们来看看系统中所有的 container PID 的 CPU 利用率，然后找出所属的 image 和 container ID ：
```
|_filename.contains(d_pids.out) cpuPct.max(CID,rss) chart(line-zero)
```

![alt](http://resource.docker.cn/docker-table.png)

看一看输出结果，我们就能发现系统中占用 CPU 最多的进程，以及它们所属的 image 和 container 。

##结论

下一篇博客会关注比较大型的系统，这些系统比较强大，运行了不同的服务器和数据库系统。我们会看到不同的团队和部门是如何管理这些系统的，能够监控到的系统健康程度，还有不依赖docker事件的告警。

*在第三部分，我们会提供 DockerApp-1.0.zip 的下载，继续关注我们吧，或者 [订阅一下](http://eepurl.com/XinRT) ~~*

![alt](http://resource.docker.cn/docker-cpu.png)


***

#####这篇文章由 [Logscape](http://go.docker.com/e/44082/logscape/756b/21481063) 撰写，[陈菊英](http://weibo.com/u/1716255775) 翻译。点击 [这里](http://blog.logscape.com/2014/07/monitoring-docker-part-ii) 阅读原文。

#####The article was contributed by [Logscape](http://go.docker.com/e/44082/logscape/756b/21481063), click [here](http://blog.logscape.com/2014/07/monitoring-docker-part-ii) to read the original publication. 
