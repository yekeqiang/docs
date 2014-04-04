##USING NGINX, CONFD, AND DOCKER FOR ZERO-DOWNTIME WEB UPDATES
##使用 Nginx，Confd 以及 Docker 实现零停机 Web 更新

***

Update March 5, 2014:  Added drone.io continuous delivery to this process, for git push to production deployment.

***

I’ve been really enjoying working with Docker recently, but one of my pain points has been how to update a website running in a docker container with no downtime.  There are apps like [https://github.com/dotcloud/hipache](https://github.com/dotcloud/hipache) which uses a custom node.js proxy that gets configuration from a redis instance.  I don’t really want to maintain something so “heavy” for just a small website, so I’ve been searching for a better solution for a while.  I even considered writing one of my own, and started a few times, only to stop — thinking that this was a wheel that didn’t need to be reinvented.

最近我一直非常喜欢和 Docker 一起工作，但是让我痛苦的是如何在不停机的情况下更新运行在 Docker 容器中的 web 应用。有这样一个 [https://github.com/dotcloud/hipache](https://github.com/dotcloud/hipache) 应用，它使用了一个从 Redis 实例中获取配置的自定义的 Node.js 代理。我真的不想为一个小小的网站维护如此“重量级”的东西，所以我花了一段时间去查找更好的方案。我甚至考虑为自己实现一个类似的东西，在开始了一段时间后，考虑到那仅仅是一个“轮子”，没有必要彻底重写，所以停止了。

***

Recently I came across [confd](https://github.com/kelseyhightower/confd) from Kelsey Hightower, which is a really slick tool that watches [etcd](https://github.com/coreos/etcd) for changes, then emits configuration file(s) based on the contents of the keys you’re watching in etcd.  It occurred to me that this could be a way to generate a proxy configuration file for one of the web proxies like haproxy or nginx that already knows how to do reverse proxy tasks well.  Just recently, Kelsey merged in support for iteration, which sealed the deal.  Now you can iterate over a list of keys and generate a configuration file to dynamically configure your proxy server.


最近我从 kelsey Hightower 了解到 Confd，它真是一个观察 etcd 变化，然后发出基于从 etcd 观察到 keys 的内容的配置文件的好工具。对与我来说，这可能是一种为网络代理（如haproxy或nginx可以很好的处理反向代理任务）产生代理配置文件的方式。就在最近，Kelsey为迭代添加了支持，它可以帮你搞定一切。现在，你可以遍历一个链表的keys，并且产生一个配置文件去动态的配置你的代理服务器。

***

I set aside a few hours this morning to try it out, and I’m very pleasantly surprised by how well it all works.  There are a few prerequisites:

今天早上我花了几小时去尝试，非常高兴它可以正常工作了。这里有几个先决条件：

***

1. You need a server running docker — I got mine from the inexpensive but very reliable DigitalOcean.com – just create a new droplet with the “Application” named Docker, and you’re set!
2. You need etcd running on that host.
3. You need confd – the latest release doesn’t include iteration support so I built it locally and installed it.

1. 你需要一个运行 Docker 的服务器。
2. 你需要一个运行 etcd 的主机。
3. 你需要 Confd －最新的版本并不包含迭代支持，所以我在本地构建和安装了 Confd。

***

To install etcd, I ran the following command on my docker enabled server:

在运行Docker的服务器上运行如下命令安装 etcd：

***

```
sudo docker run -d -p 4001:4001 -p 7001:7001 coreos/etcd
```

Now you have an instance of etcd running locally.  Optionally, download the etcd release locally and copy the file `etcdctl` to /usr/local/bin or somewhere else in your path, it’ll be useful for watching as you set all this up.  etcdctl allows you to read, set, and remove keys from the command line.

现在你就有了一个在本地的 etcd 的实例。或者，把 etcd 下载到本地，将 `etcdctl` 拷贝到 `/usr/local/bin` 或 `PATH` 中的其它位置，当你设置好这一切将对观察 etcd 非常有用。`etcdctl`允 许你通过命令行去读取，设置以及删除键值。

***

Next, I installed Go so I could compile and run confd.  I’ll leave this as an exercise for you, but it’s easy if you follow the installation instructions at [golang.org](http://golang.org).  After cloning and compiling confd, I moved it to /usr/local/bin so it’d be available everywhere.

With confd available, I created the configuration and template directories required to run confd.

接下来，我安装了`Go`以至于可以编译及运行Confd。我将此留做练习，根据[golang.org]的安装说明,对与你那将不是难事。在克隆和编译 Confd 后，我将它拷贝到`/usr/local/bin`目录下以至于随处可用。然后，通过如下命令创建了运行Confd所必需的配置和模版目录：

```
sudo mkdir -p /etc/confd/conf.d
sudo mkdir -p /etc/confd/templates
```

***

Next I made a template for nginx to use as the reverse proxy configuration.  Here’s what it looks like:

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

I saved that file as `/etc/confd/templates/gophercon-nginx.tmpl`

我将文件保存为`/etc/confd/templates/gophercon-nginx.tmpl`。

Next, I created the configuration file to tell confd which keys to watch.   Here’s that file:

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

Note that it references the configuration file we created above.  Save this file as `/etc/confd/conf.d/gophercon-nginx.toml`  The particularly cool part about this configuration is that it has commands to check the configuration and reload nginx if the configuration passes.

注意：它引用了我们上一步创建的配置文件。将这个文件保存为`/etc/confd/conf.d/gophercon-nginx.toml`。这个配置文件特别酷的部分是它有命令去检查配置，如果配置没有问题，那么会重载 nginx。

Lastly, I wrote a bash script to wire the whole thing together.  The general workflow looks like this:

最后，我写了一个 bash 脚本去将所有的东西绑定到一起。工作流大致如下所示：

1. Get the ID’s of running gophercon containers
2. Start new gophercon containers
3. Register new gophercon containers with etcd
4. run confd to reconfigure nginx
5. stop old containers
6. run confd to remove old containers from nginx configurations

1. 获取正在运行 gophercon 的容器的 IDs
2. 启动新的 gophercon 容器
3. 注册新的 gophercon 容器到 etcd
4. 运行 Confd 去重新配置 nginx
5. 停止运行旧容器
6. 运行 Confd，从 nginx 配置中移除旧容器的配置

Here’s what that script looks like:

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

Note that it references the configuration file we created above.  Save this file as `/etc/confd/conf.d/gophercon-nginx.toml`  The particularly cool part about this configuration is that it has commands to check the configuration and reload nginx if the configuration passes.

现在可以在不停机的的情况下更新 GopherCon 网站了，我在本地 create，tag 以及 push 容器，然后在服务器中运行`gophercon.sh 3`去启动3个新容器以及删除旧容器。使用了Docker，nginx以及一些有创意的工具实现了零停机部署。最好的一点是我不用写自定义的反向代理。谁想去维护那样的东西呢？

NB: if you’re going to run more than one of these, you can copy all of these scripts, and change container/key names to suit the new web app.  You can’t reuse the `upstream` name in nginx, so give each of your upstreams a unique name.

注意：如果你想运行多个，你可以拷贝这些脚本，修改容器/键名以适应新的 Web 应用程序。在 nginx 中你不能复用`upstream`的名字，所以给你的每个'upstream'一个唯一的名字。

Also, since we’re all friends here, this was the first bash script I wrote on my own.  Hard to believe that I’ve been programming for 25 years and haven’t had to write a bash script.  Feel free to gently suggest things I could have done differently so I can learn!

同样的，在这里我们都是朋友，这是我自己写的第一个 bash 脚本。很难相信我已经编程25年了，但是从来没有写过 bash 脚本。

Update:  Configuration files are on [Github](https://github.com/bketelsen/zerodowntime)
更新：所有的配置文件都在GitHub上。

Update: Updated bash script to pull new image before deploying.  D’oh!  Also note, it’s important to get the list of currently running containers before pulling the new image, because once a new image is marked “latest” for a tag, the old images won’t show up as “gophercon” in `docker ps`, they’ll just list an image ID instead.

更新：因为更新了 bash 脚本，所以在部署之前请 pull 最新的镜像。噢！请注意，在 pull 新镜像之前，获取当前运行中的容器列表是非常重要的，因为一旦新镜像被标注为“latest”标签，通过执行`docker ps`命令，老镜像将不再显示为“gophercon”，它们只会列出镜像ID。

Update:  I added this project to drone.io and used their awesome ssh deployment to copy the project to the server on a successful build, build the container locally, then trigger the gophercon.sh script.  Here’s the ssh-deployment script from drone.io:

更新：我添加列这个项目到drone.io，使用它们的ssh部署。以下是一个来自 drone.io 的 ssh-deployment 脚本：

```
docker build -t gophercon .
/root/gophercon.sh 3
```

Now all commits to the master branch will trigger a drone.io build.  When it passes, it will be copied via ssh/rsync to a temporary directory on the GopherCon server, then a docker image is built.  The last step triggers the deployment script I created earlier.  Continuous Delivery, automated docker deployments, and automated nginx reverse proxy.  Not bad!

现在，所有提交到 master 分支的代码都会触发一个 drone.io 的构建。当没有错误发生，它将通过 ssh/rsync 被拷贝到 Gophercon 服务器的一个临时文件夹，然后 Docker 镜像将被构建。最后一步将触发我前面创建的部署脚本。持续发布，自动化Docker部署，以及自动化的nginx反向代理。

[应用]:https://github.com/dotcloud/hipache
[golang.org]:http://golang.org
