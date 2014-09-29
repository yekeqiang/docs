#使用 nginx 、 Confd 以及 Docker 实现零停机 Web 更新

#####作者：[Brian Ketelsen](https://twitter.com/bketelsen)
#####译者：[Xiaobo He](https://github.com/xiaobohe)

***
最近我很享受使用 Docker ，但是让我痛苦的是如何在不停机的情况下更新运行在 Docker 容器中的 web 应用。像这个 [应用](https://github.com/dotcloud/hipache) 使用了一个从 Redis 实例中获取配置的自定义的 Node.js 代理，但我真的不想为一个小小的网站维护如此“重量级”的东西，所以花了一段时间去查找更好的方案。我甚至考虑自己实现一个类似的东西，在开始了一段时间后，意识到没必要重写，所以就停止了。

最近我从 Kelsey Hightower 了解到 [Confd](https://github.com/kelseyhightower/confd) 。它能观察 [etcd](https://github.com/coreos/etcd) 变化，然后根据从 etcd 观察到 keys 的内容发出配置文件。对于我来说，这可能是一种为网络代理（如 haproxy 或 nginx 可以很好地处理反向代理任务）产生代理配置文件的方式。就在最近， Kelsey 为迭代添加了支持，它可以帮你搞定一切。现在，你可以遍历一个链表的 keys ，并且产生一个配置文件去动态的配置你的代理服务器。

今天早上我花了几小时去尝试，非常高兴它可以正常工作了。这里有几个先决条件：

1. 你需要一个运行 Docker 的服务器。
2. 你需要一个运行 etcd 的主机。
3. 你需要 Confd －最新的版本并不包含迭代支持，所以我在本地构建和安装了 Confd 。

在运行Docker的服务器上运行如下命令安装 etcd ：

```
sudo docker run -d -p 4001:4001 -p 7001:7001 coreos/etcd
```

现在你就有了一个在本地的 etcd 的实例。或者，把 etcd 下载到本地，将`etcdctl`拷贝到`/usr/local/bin`或`PATH`中的其它位置，当你设置好这一切将对观察etcd非常有用。`etcdctl`允许你通过命令行去读取，设置以及删除键值。

接下来，我安装了 `Go` 从而编译及运行 Confd 。根据 [golang.org](http://golang.org/) 的安装说明，这个过程并非难事。在克隆和编译 Confd 后，我将它拷贝到 `/usr/local/bin` 目录下以至于随处可用。然后，通过如下命令创建了运行 Confd 所必需的配置和模版目录：

```
sudo mkdir -p /etc/confd/conf.d
sudo mkdir -p /etc/confd/templates
```

然后，我为 nginx 制作了一个用来反向代理配置的模版。如下所示：

```

upstream myapp {
{{range $server := .gophercon_upstream}}
server {{$server.Value}};
{{end}}
}
server {
server_name http://www.gophercon.com gophercon.com;
location / {
proxy_pass http://myapp;
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}
```

我将文件保存为 `/etc/confd/templates/gophercon-nginx.tmpl` 。

接下来，我创建了一个配置文件去告诉 confd 哪些 keys 需要去观察。下面是该文件：

```
[template]
keys = [
"gophercon/upstream",
]
owner = “nginx”
mode = “0644″
src = “gophercon-nginx.tmpl”
dest = “/etc/nginx/sites-enabled/gophercon.conf”
check_cmd = “/usr/sbin/nginx -t -c /etc/nginx/nginx.conf”
reload_cmd = “/usr/sbin/service nginx reload”
```

注意：它引用了我们上一步创建的配置文件。将这个文件保存为 `/etc/confd/conf.d/gophercon-nginx.toml` 。这个配置文件特别酷的部分是它有命令去检查配置，如果配置没有问题，那么会重载 nginx 。

最后，我写了一个 bash 脚本去将所有的东西绑定到一起。工作流大致如下所示：

1. 获取正在运行 gophercon 的容器的 IDs
2. 启动新的 gophercon 容器
3. 注册新的 gophercon 容器到 etcd
4. 运行 Confd 去重新配置 nginx
5. 停止运行旧容器
6. 运行 Confd，从 nginx 配置中移除旧容器的配置

脚本如下所示：

```
#!/bin/bash
if [ -z "$1" ]
then
echo “usage : gophercon.sh 3 — start three new instances”
exit -1
fi
 
echo “Getting currently running gophercon containers”
OLDPORTS=( `docker ps | grep gophercon | awk ‘{print $1}’` )
 
echo “starting new containers”
for i in `seq 1 $1` ; do
echo “inside loop $1″
JOB=`docker run -d -p 9003 gophercon | cut -c1-12`
echo $JOB
PORT=`docker inspect $JOB | grep HostPort | cut -d ‘”‘ -f 4 | head -1`
curl http://127.0.0.1:4001/v2/keys/gophercon/upstream/$JOB -XPUT -d value=”127.0.0.1:$PORT”
done
 
echo “removing old containers”
for i in ${OLDPORTS[@]}
do
etcdctl rm /gophercon/upstream/$i
confd -onetime
docker kill $i
done
```

现在可以在不停机的的情况下更新 GopherCon 网站了，我在本地 create 、 tag 以及 push 容器，然后在服务器中运行 `gophercon.sh 3` 去启动 3 个新容器并删除旧容器。通过使用 Docker 、 nginx 及其它有创意的工具实现了零停机部署。最棒的就是我不用写自定义的反向代理。谁想去维护那样的东西呢？

注意：如果你想运行多个，你可以拷贝这些脚本，修改容器/键名以适应新的 Web 应用程序。在 nginx 中你不能复用 `upstream` 这一文件名，所以需要给你的每个“ upstream ”单独命名。

坦白讲，这是我自己写的第一个 bash 脚本。很难相信我已经编程25年了，但是从来没有写过 bash 脚本。 

更新：所有的配置文件都在 [GitHub](https://github.com/bketelsen/zerodowntime) 上。

更新：因为更新了 bash 脚本，所以在部署之前请 pull 最新的镜像。噢！请注意，在 pull 新镜像之前，获取当前运行中的容器列表是非常重要的，因为一旦新镜像被标注为 “latest” 标签，通过执行 `docker ps` 命令，老镜像将不再显示为 “gophercon” ，它们只会列出镜像 ID 。

更新：我已经将这个项目添加到 [drone.io](http://drone.io/) ，使用它们的 ssh 部署。以下是一个来自 drone.io 的 ssh-deployment 脚本：

```
docker build -t gophercon .
/root/gophercon.sh 3
```

现在，所有提交到 master 分支的代码都会触发一个 drone.io 的构建。当没有错误发生，它将通过 ssh/rsync 被拷贝到 Gophercon 服务器的一个临时文件夹，然后 Docker 镜像将被构建。最后一步将触发我前面创建的部署脚本，持续发布，自动化 Dockek 部署及 nginx 反向代理。

***
#####这篇文章由 [Brian Ketelsen](https://twitter.com/bketelsen) 发布，点击 [这里](http://brianketelsen.com/2014/02/25/using-nginx-confd-and-docker-for-zero-downtime-web-updates/) 可查阅原文。 [Xiaobo He](https://github.com/xiaobohe) 翻译了本文。

#####The article was contributed by [Brian Ketelsen](https://twitter.com/bketelsen), click [here](http://brianketelsen.com/2014/02/25/using-nginx-confd-and-docker-for-zero-downtime-web-updates/) to read the original publication.
