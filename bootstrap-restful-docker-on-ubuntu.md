Bootstrap RESTful Docker on Ubuntu

标签（空格分隔）： Docker RESTful

---
> 注：该文的作者为 [Henryk Konsek][1]，原文地址为 [Bootstrap RESTful Docker on Ubuntu][2]

与 Docker 服务器最 “devops” 的交互方式是通过 RESTfUL API 暴露接口。然后使用你选择的 HTTP 客户端给 Docker Server 发送命令。

这里说明了为了能在 Ubuntu 14.04 上通过暴露 REST 来建立 Docker 服务器你需要做什么。

## 安装 Docker

这里有使用 nutshell 的 [Ubuntu 的官方安装文档][3] - 在 Ubuntu 上安装 Docker 你仅仅需要在你的 shell 键入以下命令：

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

上面提到的设置是 Docker 所期望的使用  [Fabric8 Docker 集成][4] 的默认设置。如果你遵循这个说明，你可以非常安全的使用 Fabric8 Docker 容器。

> 注：这个 Fabric8 Docker 是使用 Fabric8 来创建容器的文档，有时间会翻译下这文档


  [1]: http://www.blogger.com/profile/09392743290349794069
  [2]: http://henryk-konsek.blogspot.tw/2014/06/bootstrap-restful-docker-on-ubuntu.html?utm_source=Docker%20News&utm_campaign=60170b6fbc-Docker_0_5_0_7_18_2013&utm_medium=email&utm_term=0_c0995b6e8f-60170b6fbc-235715137
  [3]: https://docs.docker.com/installation/ubuntulinux/
  [4]: http://fabric8.io/gitbook/docker.html
