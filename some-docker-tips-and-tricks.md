# 一些使用 Docker 的技巧与秘诀

##### 作者：[Wouter Danes](https://twitter.com/wouterdanes)

##### 译者：[叶可强](http://weibo.com/1224591704)

***

Docker 可能一开始让你望而生畏，但确实是一个非常伟大的工具。用 Shells 工作令人烦扰，并且有陷阱。鉴于我花了很多时间才弄明白它，所以我想写这篇文章来避免你们做无用功。这篇文章列举了一些快速技巧、秘诀和单行脚本，帮助你更有效地使用 Docker 。

## 移除所有的容器和镜像（大扫除）

用一行命令大扫除：

```
  docker kill $(docker ps -q) ; docker rm $(docker ps -a -q) ; docker rmi $(docker images -q -a) 
```

> ##### 译者注：shell 中的 $() 和 `` 类似，会先执行这里面的内容，上面的脚本会出现如下 docker kill "pids" ; docker kill 在 docker 中用于停止容器，docker rm 用户删除容器， docker rmi 删除镜像

当没有运行的容器或者是根本没有容器的时候，这只是个警告。不过当你想尝试运行的时候，这就是个非常好的单行命令。如果你想删除所有容器，可以运行如下命令：

```
docker kill $(docker ps -q) ; docker rm $(docker ps -a -q) 
```

## 退出时删除容器

如果你仅仅想在一个容器中快速地运行一个命令，然后退出，并且不用担心容器状态，那请把 ```--rm``` 参数加入 ```run``` 命令后面，这将结束很多你保存了的容器，并且清理它们。

示例：```docker run --rm -i -t busybox /bin/bash```

## 不在 Shell 上运行命令

如果你使用需要 Shell 的扩展项的 ```docker run``` 命令处理事情，比如 ```docker run --rm busybox ls '/var/log/*'``` ，这个命令将失败。这个失败的原因我花了工夫才弄明白。这个陷阱在这里：你原来没有 Shell ， 而 ```*``` 是 Shell 的扩展项，因此你需要一个能使用的 Shell 。正确方法为：

```
docker run --rm busybox sh -c 'ls /var/log/*'
```

## Boot2Docker 和 LapTops 处理 DNS 问题的方法

我在三个不同的地方使用我的便携式电脑，并且他们有不同的 ISPs。由于这个原因， [Boot2Docker](https://github.com/boot2docker/boot2docker-cli) 会占用 DNS 服务器很长一段时间。当你在尝试创建镜像的时候，可能会得到离谱的错误。如果你在 Ubuntu 或者 CentOS 上看到下面的提示：

```
cannot lookup archive.ubuntu.com
```

聪明的做法是停止，然后启动 boot2docker 。

```
boot2docker-cli down && boot2docker-cli up
```

这样问题就解决了。

## Volumes 解决 ```docker logs``` 和 ```docker copy``` 问题

如果你想在一个容器中监控另一个容器中的日志文件和文件的使用，你可以看看 [volumes](http://docs.docker.io/use/working_with_volumes/#volume-def) ，例如，检查 tomcat 是否启动：

```
tomcat_id=$(docker run -d -v /var/log/tomcat6 wouterd/tomcat6)
# Give Tomcat some time to wake up...
sleep 5
while ! docker run --rm --volumes-from ${tomcat_id} busybox /bin/sh -c "grep -i -q 'INFO: Server startup in' /var/log/tomcat6/catalina*.log" ; do
    echo -n "."
    sleep 5
done
```

你还可以在一个 ```Dockerfile```中指定 volumes ，这个在我前面的 [博客文章](http://www.wouterdanes.net/2014/04/11/continuous-integration-using-docker-maven-and-jenkins.html) 写过了。

## Docker Inspect 结合 Go Templates 的好处

命令 ```docker inspect``` 允许使用 [Go Templates](http://golang.org/pkg/text/template/) 来格式化 inspect 命令的输出信息。如果你擅长这个，你能获取很多 docker 容器命令行的脚本输出信息。这是一个获取正在运行的容器 IP 的示例：

```
container_ip=$(docker inspect --format '{{.NetworkSettings.IPAddress}}' ${container_id}) 
``` 
这里也有个笨办法去得到匹配所有暴露的端口 host:port ，并且把他们输入一个 java properties 文件：

```
sut_ip=${BOOT_2_DOCKER_HOST_IP}
template='{{ range $key, $value := .NetworkSettings.Ports }}{{ $key }}='"${BOOT_2_DOCKER_HOST_IP}:"'{{ (index $value 0).HostPort }} {{ end }}'
tomcat_host_port=$(docker inspect --format="${template}" ${container_id})
for line in ${tomcat_host_port} ; do
    echo "${line}" >> ${work_dir}/docker_container_hosts.properties
done
```

> ###### 译者注：如需转载，请注明出处。由于个人的翻译水平问题，同时欢迎各位读者修正翻译的错误，并且就 Docker 问题进行讨论。

***

##### 这篇文章由 [Wouter Danes](https://twitter.com/wouterdanes) 发表，[叶可强](http://weibo.com/1224591704) 翻译。点击 [这里](http://www.wouterdanes.net/2014/04/16/some-docker-tips-and-tricks.html) 可查阅原文。

##### The article was contributed by [Wouter Danes](https://twitter.com/wouterdanes) , click [here](http://www.wouterdanes.net/2014/04/16/some-docker-tips-and-tricks.html) to read the original publication.


  
