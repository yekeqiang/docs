##在CoreOS和Fig构建基于Docker的复杂应用


使用Docker来部署系统，CoreOS的功能比较强大但是初期的学习曲线比较高。Fig相对而言比较简单，但是很难在多台服务器上做扩展。这篇博客将向您展示如何解决使用Fig构建复杂的多个容器的应用并且把这些应用部署到基于CoreOS的生产环境之间的技术问题。

### 完整的构建基于Fig的多容器应用

在上个星期的博文中，我们谈到了在Fig中构建有4个容器的应用。开门见山，下面这个fig.xml就是用来构建4个容器的应用的。

```
serf:
  image: ctlc/serf
  ports:
    - 7373
    - 7946
lb:
  image: ctlc/haproxy
  ports:
    - 80:80
  links:
    - serf
  environment:
    HAPROXY_PASSWORD: qa1N76pWAri9
web:
  image: ctlc/wordpress
  ports:
    - 80
  environment:
    DB_PASSWORD: qa1N76pWAri9
  links:
    - serf
    - db
  volumes:
    - /local/path/to/wordpress:/app
db:
  image: orchardup/mysql
  ports:
    - 3306
  volumes:
    - /mysql:/var/lib/mysql
  environment:
    MYSQL_DATABASE: wordpress
    MYSQL_ROOT_PASSWORD: qa1N76pWAri9
    
```

如果要在你的环境中部署这个具有4个容器的系统，只需要简单运行 fig up -d (和重启系统的命令一样)。

```
$ fig up -d
Recreating ctlcblog_serf_1...
Recreating ctlcblog_db_1...
Recreating ctlcblog_web_1...
Creating ctlcblog_lb_1...

$ fig ps
     Name              Command         State                Ports               
-------------------------------------------------------------------------------
ctlcblog_serf_1   /run.sh              Up      49252->7373/tcp, 49253->7946/tcp 
ctlcblog_db_1     /usr/local/bin/run   Up      49254->3306/tcp                  
ctlcblog_web_1    /run.sh              Up      49255->80/tcp                    
ctlcblog_lb_1     /run.sh              Up      80->80/tcp                       

$ fig scale web=2
Starting ctlcblog_web_2...

$ fig ps
     Name              Command         State                Ports               
-------------------------------------------------------------------------------
ctlcblog_serf_1   /run.sh              Up      49252->7373/tcp, 49253->7946/tcp 
ctlcblog_db_1     /usr/local/bin/run   Up      49254->3306/tcp                  
ctlcblog_web_2    /run.sh              Up      49256->80/tcp                    
ctlcblog_web_1    /run.sh              Up      49255->80/tcp                    
ctlcblog_lb_1     /run.sh              Up      80->80/tcp          
```
你可以看到，Fig的使用是非常简单和方便的，但是有一个问题就是我们如何把它扩展到多个服务器上?

### 你如何在CoreOS中基于Fig的配置来创建容器？
CoreOS不是这个世界上最简单或者说最容易上手的系统。理解并正确编写一个systemd的配置文件是非常耗时而且让人困扰的。
这个也就是为什么我写个一个fig2coreos的gem ([github.com/centurylinklabs/fig2coreos](github.com/centurylinklabs/fig2coreos)),它可以把fig.yml自动转换成CoreOS格式的systemd配置文件。

```

$ sudo gem install fig2coreos
$ fig2coreos wordpress-app fig.yml coreos-dir
[SUCCESS] Try this: cd /Users/cardmagic/Sites/centurylinklabs.com/coreos-dir && vagrant up
$ cd coreos-dir && vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'coreos'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] -- 80 => 8080 (adapter 1)
[default] Running 'pre-boot' VM customizations...
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] Setting hostname...
[default] Configuring and enabling network interfaces...
[default] Exporting NFS shared folders...
Preparing to edit /etc/exports. Administrator privileges will be required...
Password:
[default] Mounting NFS shared folders...
[default] Running provisioner: shell...
[default] Running: /var/folders/9j/gkydy1sn1nsd73yd2n3l68400000gn/T/vagrant-shell20140224-18153-19i4zr8
$ cd coreos-dir && vagrant ssh
Last login: Mon Feb 24 23:57:38 UTC 2014 from 10.0.2.2 on ssh
   ______                ____  _____
  / ____/___  ________  / __ \/ ___/
 / /   / __ \/ ___/ _ \/ / / /\__ \
/ /___/ /_/ / /  /  __/ /_/ /___/ /
\____/\____/_/   \___/\____//____/
core@coreos-wordpress-app ~ $ 

```
由于某些原因(我也不知道为什么)，CoreOS发布的默认版本，当你用vagrant up启动的时候，其实它不是最新的coreos版本，这个就意味着它没有安装[Fleet](https://github.com/coreos/fleet)，并且上面的Docker(0.7.2)也是旧的.一旦vagrant coreos box已经启动并且下载完了最新的coreos的版本。你就可以通过在coreos的虚拟机中运行sudo reboot或者在vagrant的当前工作目录下运行vagrant reload --provision 来执行更新的操作。

### fig2coreos到底做了什么呢？
fig2coreos解析你的fig.yml并且生成一系列的systemd配置文件。你可以在它生成的目录中查看这些文件

```

$ ls coreos-dir/media/state/units/
db-discovery.1.service
lb-discovery.1.service
serf-discovery.1.service
web-discovery.1.service
db.1.service
lb.1.service
serf.1.service
web.1.service

$ cat coreos-dir/media/state/units/db.1.service
[Unit]
Description=Run db_1
After=docker.service
Requires=docker.service

[Service]
Restart=always
RestartSec=10s
ExecStartPre=/usr/bin/docker ps -a -q | xargs docker rm
ExecStart=/usr/bin/docker run -rm -name db_1 -v /mysql:/var/lib/mysql  -e "MYSQL_DATABASE=wordpress" -e "MYSQL_ROOT_PASSWORD=nqhT4hT0RT6k" -p 3306 orchardup/mysql
ExecStartPost=/usr/bin/docker ps -a -q | xargs docker rm
ExecStop=/usr/bin/docker kill db_1
ExecStopPost=/usr/bin/docker ps -a -q | xargs docker rm

[Install]
WantedBy=local.target

```
看上去很复杂的，是你fig.yml中关于CoreOS版本信息的那一部分：

```
db:
  image: orchardup/mysql
  ports:
    - 3306
  volumes:
    - /mysql:/var/lib/mysql
  environment:
    MYSQL_DATABASE: wordpress
    MYSQL_ROOT_PASSWORD: qa1N76pWAri9
    
```

除了MySQL的启动脚本之外，也有一个通过systemd实现的etcd自动注册脚本，它在db-discovery.1.service 文件中。

```
$ cat coreos-dir/media/state/units/db-discovery.1.service
[Unit]
Description=Announce db_1
BindsTo=db.1.service

[Service]
ExecStart=/bin/sh -c "while true; do etcdctl set /services/db/db_1 '{ \"host\": \"%H\", \"port\": 3306, \"version\": \"52c7248a14\" }' --ttl 60;sleep 45;done"
ExecStop=/usr/bin/etcdctl rm /services/db/db_1

[X-Fleet]
X-ConditionMachineOf=db.1.service
```
这个把服务注册到etcd的键值存储中。在这个Fig应用的例子中，我们使用Serf来实现Docker容器的自省，但是etcd是标准的CoreOS容器用来替换Serf的服务。
你既需要Serf又需要etcd么？不，两者最大的区别在于守护进程的运行位置。etcd守护进程是运行在Docker容器外部的，它就像systemd一样的一个系统来保证key/value的更新。Serf的守护进程是运行在Docker容器内部的，所以当容器死亡或者下线的时候，同等的容器会被自动移除。
在系统管理和配置管理中有两个不同的哲学，但是结果殊途同归。Serf和etcd都可以让Docker的应用察觉到它们所对应的容器，它们所扮演的角色以及互相的联系

### 那么如何把Docker扩展到不同的主机上？
理论上来说，你可以在你每一台服务器上使用Fig,让每一台机器运行一个独立的Docker守护进程。然而链接容器变得很复杂，而且网络问题也变得很严重，即使是用Serf也是一样。
在这一系列的博文中，我们差不多可以向您展示如何部署一个多主机的Docker服务了。但是这篇博文任然不会给你明确的答案。订阅我们每周的更新，我们最终将向您展示一切。
与此同时，你也知道我没有在骗你，看这个[youtube上2分钟的关于CoreOS Fleet的视频](http://www.youtube.com/watch?v=u91DnN-yaJ8)来了解一个真实的关于部署多主机CoreOS服务的例子。你可以在这个简单的视频中发现CoreOS和Fleet的强大。但是现在还没法把所有的组件组合在一起来重现视频中的例子。

### 小结
