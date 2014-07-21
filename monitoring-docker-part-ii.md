#监控Docker -- 第二部分

Docker发展迅速，功能日趋完善，但是Docker监控却刚刚起步。大多数已有的监控解决方案只能监控container资源的基本使用情况，它会告诉你在container层级用了多少内存、CPU和硬盘资源。这些是有用的，但是当部署规模变大时，能起到的作用显得比较小。在Docker系列的第二部分中，我们一起来看看还可以用哪些属性来更好的监控Docker。

![](http://blog.logscape.com/wp-content/uploads/2014/07/Selection_552.png)

在[第一部分](http://blog.logscape.com/2014/06/monitoring-docker-mongo-cluster-part-1/)中，我们已经演示过了用docker和netcat命令从Logscape中获取数据，以此监控docker的资源使用情况。大规模部署的时候需要更多更详细信息。想象一下这个场景，你用了大量流行的技术，运行Nginx、MongoDB、HAProxy和Node.js多个服务。当用一个image生成多个container，不同的container运行了同一个应用的不同版本，监控它们就会比较困难。

我们需要把部署信息等联系起来，构造出可管理的基础架构视图。我们需要进程ID，它所属的container ID，基础的image名称。我们获取到这些信息和其关联的度量指标，我们就能控制大规模的部署，不至于手足无措了。下面是我们数据的一个表格视图：
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

##获取container的PID

Docker采用cgroups来隔离资源。要想知道一个进程是否采用了Docker控制组，在proc文件系统中打开下面的文件：
```
/proc/$PID/cgroup
```

还可以用ps命令来显示进程表中每个pid的控制组，类似于下面的命令：
```
ps-wweouname,pid,cgroup
```

把*docker ps*运行的结果和container ID与image对应的pid等关联起来，用任何脚本语言来实现都并不是很困难。DockerApp采用的dockerpids.groovy脚本就是干这事的，下面的一个样例输出：
```
16-Jul-201414:54:52 BST, CID=31dc0a17d71d,image=logscape/forwarder:latest,names=trusting_hawking,cmd=/bin/sh,pcpu=0.0,pmem=0.0,rss=2040,comm=bash
16-Jul-201414:54:52 BST, CID=e93abb421dda,image=logscape/forwarder:latest,names=insane_hoover,cmd=bash,pcpu=0.0,pmem=0.0,rss=24112,comm=java
16-Jul-201414:54:52 BST , CID=e93abb421dda,image=logscape/forwarder:latest,names=insane_hoover,cmd=bash,pcpu=0.5,pmem=0.2,rss=124876,comm=java
```

该脚本输出pid、资源利用率和Docker相关的元信息。

##搜索

下面这个部分和这个系列的最后一个部分都会详细的研究DockerApp-1.0。它提供的服务是基于[第一部分](http://blog.logscape.com/2014/06/monitoring-docker-mongo-cluster-part-1/)用的命令，还有从cgroups和proc文件系统收集信息的脚本。采用上面展示过的示例信息，我们来看看几个搜索的结果。

**找出系统中哪个容器占用了最多的内存，是非常有意思的事情。下面的搜索语句给出了container ID占用的平均内存。**
```
| _type.equals(d_pids) rss.avg(CID,rss) chart(line-zero)
```

搜索结果还会展示内存是如何在不同image之间分布的。

![](http://blog.logscape.com/wp-content/uploads/2014/07/docker-mem.png)

我们能看到container：[e93abb421dda](https://bitbucket.org/logscape/blog-docker/commits/e93abb421dda)和[31dc0a17d71d](https://bitbucket.org/logscape/blog-docker/commits/31dc0a17d71d)在1345附近开始，然后就迅速提升。这两个container属于image：[logscape/fowarder](https://bitbucket.org/logscape/blog-docker/commits/e93abb421dda)，都运行了相对比较大型的Java程序。下一个搜索语句会给出我们系统中运行的所有进程的heatmap表：
```
 | _type.equals(d_pids) comm.by(pid,command) rss.max(pid,rss)  cpuPct.max(pid,cpu)  memPct.max(pid,mem)  image.by(pid,)  CID.by(pid,) chart(table) buckets(1)
```

![](http://blog.logscape.com/wp-content/uploads/2014/07/docker-table.png)

 我们来看看系统中所有的container PID的CPU利用率，然后找出所属的image和container ID：
```
|_filename.contains(d_pids.out) cpuPct.max(CID,rss) chart(line-zero)
```

![](http://blog.logscape.com/wp-content/uploads/2014/07/docker-cpu.png)

看一看输出结果，我们就能发现系统中占用CPU最多的进程，它们所属于的image和container。

##结论

下一篇博客会关注比较大型的系统，这些系统比较强大，运行了不同的服务器和数据库系统。我们会看到不同的团队和部门是如何管理这些系统的，能够监控到的系统健康程度，还有不依赖docker事件的告警。

*在第三部分，我们会提供DockerApp-1.0.zip的下载，继续关注我们吧，或者[订阅一下](http://eepurl.com/XinRT)~~*

![](http://blog.logscape.com/wp-content/uploads/2014/07/docker-dashboard-1024x442.png)

原文链接：<http://blog.logscape.com/2014/07/monitoring-docker-part-ii>