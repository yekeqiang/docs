#Mac OS X 下 Docker 安装教程

---

**注意事项**

1. 这些指令使用的版本是 docker version 0.8 版本，当然随时可能发生变化。

2. docker 一直在稳健的发展，不建议在生产环境中使用。请参阅 docker 官方博客 [*Getting to Docker 1.0*](http://blog.docker.io/2013/08/getting-to-docker-1-0/)

3. docker 支持 Mac OS X 10.6 及以上版本

---

##如何在 Mac OS X 上安装Docker

###VirtualBox

在 OS X 运行 docker 需要安装 virtualbox ，首先，先从 Virtualbox 页面获取安装包（ OS X 主机版本 x86/amd64 ）。

下载完成后，打开磁盘镜像，设置和运行文件（ VirtualBox.pkg ）来安装 virtualbox ，不要简单的复制没有运行的安装包。

###boot2docker

[boot2docker](https://github.com/boot2docker/boot2docker) 提供了一个简单的脚本来管理正在运行 docker 进程的虚拟主机。它还负责为操作系统镜像的安装工作。

如果你还没安装 boot2docker ，请打开一个新的终端窗口

运行下边的命令来获得 boot2docker 。

`# 进去安装目录
cd ~/bin`

`# 获取安装文件
curl https://raw.github.com/steeve/boot2docker/master/boot2docker > boot2docker`

`# 设置可执行
chmod +x boot2docker`

###Docker OS X Client
docker 进程使用 docker 客户端访问。

运行下边的命令，来获取docker并且设置它。

`# 获取文件
curl -o docker http://get.docker.io/builds/Darwin/x86_64/docker-latest`

`# 设置可执行
chmod +x docker`

`# 设置docker进程的环境变量
export DOCKER_HOST=tcp://`

`# 复制可执行文件
sudo cp docker /usr/local/bin/`

然后让我们看看如何使用它。

##如何在 Mac OS X 上使用 Docker
###docker 进程（通过 boot2docker ）

进行~/bin目录，运行下边的命令：

`# 初始化虚拟主机
./boot2docker init`

`# 运行虚拟主机 (the docker daemon)
./boot2docker up`

`# 看所有可用的命令:
./boot2docker`

`# Usage ./boot2docker {init|start|up|pause|stop|restart|status|info|delete|ssh|download}`

###docker 客户端

一旦虚拟主机运行 docker 进程，你可以像使用其它的一些应用一样来使用 docker 客户端。

`docker version`	
`# Client version: 0.7.6`	
`# Go version (client): go1.2`	
`# Git commit (client): bc3b2ec`	
`# Server version: 0.7.5`	
`# Git commit (server): c348c04`	
`# Go version (server): go1.2`

###用 SSH 来连接虚拟主机
如果你感觉需要连接虚拟主机，你可以简单的运行下边的命令：

`./boot2docker ssh`

`# User: docker`	
`# Pwd:  tcuser`

现在你可以使用 hello World 的例子了~

###学习更多

**boot2docker**

在 GitHub 查看 [boot2docker](https://github.com/steeve/boot2docker) 页面。

**如果 SSH 提示需要秘钥**

`ssh-keygen -R '[localhost]:2022'`

---
#####这篇文章由 [widuu](http://weibo.com/widuu) 翻译并发表于 [微度网络](http://www.widuu.com/)，Docker 中文社区获得作者许可转载。您也可以点击 [这里](http://docs.docker.io/en/latest/installation/mac/) 阅读英文原版教程。
