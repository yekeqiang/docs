##使用nginx，Confd以及Docker实现零停机Web更新

最近我一直非常喜欢和 Docker 一起工作，但是让我痛苦的是如何在不停机的情况下更新运行在 Docker 容器中的 web 应用。有这样一个[应用]，它使用了一个从 Redis 实例中获取配置的自定义的 Node.js 代理。我真的不想为一个小小的网站维护如此“重量级”的东西，所以我花了一段时间去查找更好的方案。我甚至考虑为自己实现一个类似的东西，在开始了一段时间后，考虑到那仅仅是一个“轮子”，没有必要彻底重写，所以停止了。

最近我从 kelsey Hightower 了解到 Confd，它真是一个观察 etcd 变化，然后发出基于从 etcd 观察到 keys 的内容的配置文件的好工具。对与我来说，这可能是一种为网络代理（如haproxy或nginx可以很好的处理反向代理任务）产生代理配置文件的方式。就在最近，Kelsey为迭代添加了支持，它可以帮你搞定一切。现在，你可以遍历一个链表的keys，并且产生一个配置文件去动态的配置你的代理服务器。

今天早上我花了几小时去尝试，非常高兴它可以正常工作了。这里有几个先决条件：

1. 你需要一个运行 Docker 的服务器。
2. 你需要一个运行 etcd 的主机。
3. 你需要 Confd －最新的版本并不包含迭代支持，所以我在本地构建和安装了 Confd。

在运行Docker的服务器上运行如下命令安装 etcd：

```
sudo docker run -d -p 4001:4001 -p 7001:7001 coreos/etcd
```

现在你就有了一个在本地的 etcd 的实例。或者，把 etcd 下载到本地，将`etcdctl`拷贝到`/usr/local/bin`或`PATH`中的其它位置，当你设置好这一切将对观察etcd非常有用。`etcdctl`允许你通过命令行去读取，设置以及删除键值。

接下来，我安装了`Go`以至于可以编译及运行Confd。我将此留做练习，根据[golang.org]的安装说明,对与你那将不是难事。在克隆和编译 Confd 后，我将它拷贝到`/usr/local/bin`目录下以至于随处可用。然后，通过如下命令创建了运行Confd所必需的配置和模版目录：

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

我将文件保存为`/etc/confd/templates/gophercon-nginx.tmpl`。

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

注意：它引用了我们上一步创建的配置文件。将这个文件保存为`/etc/confd/conf.d/gophercon-nginx.toml`。这个配置文件特别酷的部分是它有命令去检查配置，如果配置没有问题，那么会重载 nginx。

最后，我写了一个 bash 脚本去将所有的东西绑定到一起。工作流大致如下所示：

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

现在可以在不停机的的情况下更新 GopherCon 网站了，我在本地 create，tag 以及 push 容器，然后在服务器中运行`gophercon.sh 3`去启动3个新容器以及删除旧容器。使用了Docker，nginx以及一些有创意的工具实现了零停机部署。最好的一点是我不用写自定义的反向代理。谁想去维护那样的东西呢？

注意：如果你想运行多个，你可以拷贝这些脚本，修改容器/键名以适应新的 Web 应用程序。在 nginx 中你不能复用`upstream`的名字，所以给你的每个'upstream'一个唯一的名字。

同样的，在这里我们都是朋友，这是我自己写的第一个 bash 脚本。很难相信我已经编程25年了，但是从来没有写过 bash 脚本。 

更新：所有的配置文件都在GitHub上。

更新：因为更新了 bash 脚本，所以在部署之前请 pull 最新的镜像。噢！请注意，在 pull 新镜像之前，获取当前运行中的容器列表是非常重要的，因为一旦新镜像被标注为“latest”标签，通过执行`docker ps`命令，老镜像将不再显示为“gophercon”，它们只会列出镜像ID。

更新：我添加列这个项目到drone.io，使用它们的ssh部署。以下是一个来自 drone.io 的 ssh-deployment 脚本：

```
docker build -t gophercon .
/root/gophercon.sh 3
```

现在，所有提交到 master 分支的代码都会触发一个 drone.io 的构建。当没有错误发生，它将通过 ssh/rsync 被拷贝到 Gophercon 服务器的一个临时文件夹，然后 Docker 镜像将被构建。最后一步将触发我前面创建的部署脚本。持续发布，自动化Docker部署，以及自动化的nginx反向代理。

[应用]:https://github.com/dotcloud/hipache
[golang.org]:http://golang.org

