#Docker Remote API


>注：这篇文章是系列教程中的一篇，本篇文章由 [Flux7 Labs](https://twitter.com/Flux7Labs) 发布，原文地址 [Docker Tutorial Series, Part 8: Docker Remote API](http://blog.flux7.com/blogs/docker/docker-tutorial-series-part-8-docker-remote-api)

在前面的文章中，作为正在进行的 Docker 教程系列的一部分，我们讨论了  [Docker Hub and Docker Registry API](http://flux7.com/blogs/docker/docker-tutorial-series-part-7-ultimate-guide-for-docker-apis/)。 在今天的文章中，让我们深入探讨 Docker Remote API。

---

Docker Remote API 是一个 REST 风格的 API，它用于代替远程的命令行接口 - rcli。为了达到本教程的目的，我们使用一个命令行工具 cURL 来处理我们所有的 url 操作，它帮助我们构造请求，获取和发送数据，并且获取信息。

- **List containers** - 用以下命令获取所有容器的列表：
 ```
 GET /containers/json
 ```
 ![alt](http://resource.docker.cn/get-all-containers.png)
 
- **Create a new container** - 一个新的容器被创建：
 ```
 POST /containers/create
 ```
 ![alt](http://resource.docker.cn/docker-create-container.png)

- **Inspect Container** - 这个命令用于返回指定 id 的容器的低层级的信息：
```
GET /containers/(id)/json
```
![alt](http://resource.docker.cn/docker-inspect-a-container.png)

- **Process List** - 获取一个容器中正在运行的所有进程： 
```
GET /containers/(id)/top
```
![alt](http://resource.docker.cn/docker-container-top.png)

- **Container Logs** - 从容器中收集 stdout 和 stderr 的日志：
```
GET /containers/(id)/logs
```
![alt](http://resource.docker.cn/docker-container-logs.png)

- **Export Container ** - 用以下命令导出容器的内容：
```
GET /containers/(id)/export
```
![alt](http://resource.docker.cn/docker-export-container.png)

- **Start a container** - 用以下命令启动容器：
```
POST /containers/(id)/start
```
![alt](http://resource.docker.cn/docker-start-container.png)

- **Stop a container** - 用以下命令停止容器：
```
POST /containers/(id)/stop
```

![alt](http://resource.docker.cn/docker-stop-a-container.png)

**Restart a Container** - 用以下命令重起一个容器：
```
POST /containers/(id)/restart
```
![alt](http://resource.docker.cn/docker-restart-a-container.png)

- **Kill a container** - 用以下命令 kill 一个容器：
```
POST /containers/(id)/kill
```
![alt](http://resource.docker.cn/docker-kill-a-container.png)

现在我们已经进行了你下一阶段的 Docker API 之旅，下个星期我们将开启关于 Docker OAuth 方面的内容。这是在每周四你所能发现的正在进行的 Docker 教程系列的所有部分。

下次再见，查看 Flux7 的 [Docker Assessment Program](http://flux7.com/docker-assessment-package/)，它被设计于通过使用 #Docker 来检查和解决我们的开发工作流的优化工作。为了学习更加多的评定和使用 Docker，只需要简单的通过 info@flux7.com 或者点击[这里](http://flux7.com/docker-assessment-package/) 订阅，或者是直接访问 [我们](http://flux7.com/docker-solution/) 。

## 这个系列的其他教程

[Part 1: An Introduction](http://flux7.com/blogs/docker/docker-tutorial-series-part-1-an-introduction/)

[Part 2: The 15 Commands](http://flux7.com/blogs/docker/docker-tutorial-series-part-2-the-15-commands/)

[Part 3: Automation is the word using DockerFile](http://flux7.com/blogs/docker/docker-tutorial-series-part-3-automation-is-the-word-using-dockerfile/)

[Part 4: Registry & Workflows](http://flux7.com/blogs/docker/docker-tutorial-series-part-4-registry-workflows/)

[Part 5: Docker Security](http://flux7.com/blogs/docker/docker-tutorial-series-part-5-docker-security/)

[Part 6: The Next 15 Commands](http://flux7.com/blogs/docker/docker-commands/)

[Part 7: Ultimate Guide for Docker APIs](http://flux7.com/blogs/docker/docker-tutorial-series-part-7-ultimate-guide-for-docker-apis/)

[Part 9: 10 Docker Remote API Commands for Images](http://blog.flux7.com/blogs/docker/docker-tutorial-series-part-9-10-docker-remote-api-commands-for-images)


  [1]: http://flux7.com/
  [2]: http://flux7.com/blogs/docker/docker-tutorial-series-part-8-docker-remote-api/?utm_source=Docker%20News&utm_campaign=f90a032721-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-f90a032721-235715137
  [3]: http://flux7.com/blogs/docker/docker-tutorial-series-part-7-ultimate-guide-for-docker-apis/
  [4]: http://flux7.com/wp-content/uploads/get-all-containers.png
  [5]: http://flux7.com/wp-content/uploads/docker-create-container.png
  [6]: http://flux7.com/wp-content/uploads/docker-inspect-a-container.png
  [7]: http://flux7.com/wp-content/uploads/docker-container-top.png
  [8]: http://flux7.com/wp-content/uploads/docker-container-logs.png
  [9]: http://flux7.com/wp-content/uploads/docker-export-container.png
  [10]: http://flux7.com/wp-content/uploads/docker-start-container.png
  [11]: http://flux7.com/wp-content/uploads/docker-stop-a-container.png
  [12]: http://flux7.com/wp-content/uploads/docker-restart-a-container.png
  [13]: http://flux7.com/wp-content/uploads/docker-kill-a-container.png
  [14]: http://flux7.com/docker-assessment-package/
  [15]: http://flux7.com/docker-assessment-package/
  [16]: http://flux7.com/docker-solution/
  [17]: http://flux7.com/blogs/docker/docker-tutorial-series-part-1-an-introduction/
  [18]: http://flux7.com/blogs/docker/docker-tutorial-series-part-2-the-15-commands/
  [19]: http://flux7.com/blogs/docker/docker-tutorial-series-part-3-automation-is-the-word-using-dockerfile/docker/docker-tutorial-series-part-2-the-15-commands/
  [20]: http://flux7.com/blogs/docker/docker-tutorial-series-part-4-registry-workflows/
  [21]: http://flux7.com/blogs/docker/docker-tutorial-series-part-5-docker-security/
  [22]: http://flux7.com/blogs/docker/docker-commands/
  [23]: http://flux7.com/blogs/docker/docker-tutorial-series-part-7-ultimate-guide-for-docker-apis/
