#Bootstrap RESTful Docker on Ubuntu


#####作者： [Henryk Konsek](https://twitter.com/hekonsek)

#####译者：[叶可强](http://weibo.com/yeziyu)

***
与 Docker 服务器最 “devOps” 的交互方式是通过 RESTful API 暴露接口，然后使用你选择的 HTTP 客户端给 Docker Server 发送命令。

这里说明了为了能在 Ubuntu 14.04 上通过暴露 REST 来建立 Docker 服务器你需要做什么。

## 安装 Docker

这里有使用 nutshell 的 [Ubuntu 的官方安装文档](https://docs.docker.com/installation/ubuntulinux/) - 在 Ubuntu 上安装 Docker 。你仅需要在 shell 键入以下命令：

```
curl -s https://get.docker.io/ubuntu/ | sudo sh
```

## 通过 HTTP 暴露 Docker 接口

Docker 默认是通过 Unix sockets 暴露的，你可以通过额外的选项 ```-H``` 来改变它：
```
sudo sh -c 'echo "DOCKER_OPTS=\"-H unix:///var/run/docker.sock -H tcp://127.0.0.1:2375\"" > /etc/default/docker'
sudo service docker restart
```

为了验证 Docker 已经正确的通过 HTTP 暴露接口，执行以下命令：

```
curl http://127.0.0.1:2375/version
{"ApiVersion":"1.12","Arch":"amd64","GitCommit":"990021a","GoVersion":"go1.2.1","KernelVersion":"3.13.0-29-generic","Os":"linux","Version":"1.0.1"}

```

## 非 ROOT 权限运行 Docker
如果你想不通过 sudo 执行 Docker 命令，把当前用户添加进 Docker 系统账户组：
```
sudo vim /etc/group
...
    docker:x:999:hekonsek
```

## 准备 Fabric8 

上面提到的设置是 Docker 所期望的使用  [Fabric8 Docker 集成](http://fabric8.io/gitbook/docker.html) 的默认设置。如果你遵循这个说明，你可以非常安全的使用 Fabric8 Docker 容器。

> 译者注：这个 Fabric8 Docker 是使用 Fabric8 来创建容器的文档，译者有时间会翻译这文档。

***

#####这篇文章由 [Henryk Konsek](https://twitter.com/hekonsek) 撰写，[叶可强](http://weibo.com/yeziyu) 翻译。点击 [这里](http://henryk-konsek.blogspot.tw/2014/06/bootstrap-restful-docker-on-ubuntu.html) 阅读原文。

#####The article was contributed by [Henryk Konsek](https://twitter.com/hekonsek), click [here](http://henryk-konsek.blogspot.tw/2014/06/bootstrap-restful-docker-on-ubuntu.html) to read the original publication.
