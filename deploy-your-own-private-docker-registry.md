#部署自己的私有 Docker Registry

![alt](http://resource.docker.cn/ship-with-containers.jpg)
######图片来自：[Glyn Lowe Photoworks](http://www.flickr.com/photos/glynlowe/)

#####作者：[Matthew Fisher](http://www.activestate.com/blog/authors/matthewf)

#####译者：[巨震](https://github.com/crystaldust)

---

这篇博客讨论了如何部署一个带 SSL 加密、HTTP 验证并有防火墙防护的私有 [Docker Registry](https://github.com/dotcloud/docker-registry) 。[Docker Registry](https://github.com/dotcloud/docker-registry) 是一个存储和分享 [Docker](http://docker.io) 镜像的服务。本文中我们使用的操作系统是 [Ubuntu](http://www.ubuntu.com)，任何支持 [Upstart](http://upstart.ubuntu.com/) 的系统都可以。我们用 [Nginx](http://nginx.org) 作为 [Docker Registry](https://github.com/dotcloud/docker-registry) 的前端代理服务器，同时也用 [Nginx](http://nginx.org) 完成 SSL 加密和基本的 HTTP 验证。我们用 [Gunicorn](http://gunicorn.org) 运行 [Docker Registry](https://github.com/dotcloud/docker-registry) 并用 [Upstart](http://upstart.ubuntu.com/) 管理 [Gunicorn](http://gunicorn.org)。我们还用 [Redis](http://redis.io) 实现一个 LRU(Least Recently Used，近期最少使用算法) 缓存机制来减少 [Docker Registry](https://github.com/dotcloud/docker-registry) 和硬盘之间的数据存取。


##为什么需要Docker Registry?

当在自己的环境中创建 [Docker](http://docker.io) 镜像的时候，无论是装 [Redis](http://redis.io/)，[Hipache](https://github.com/dotcloud/hipache)，还是 IRC 协议的 [logbot](https://github.com/dannvix/Logbot) ，你都希望可以把镜像存到一个安全的地方。也许你项目中的 [Docker](http://docker.io) 镜像需要安装 [Jenkins](http://buildbot.net/)，或者每次 commit 都跑一遍 [Buildbot](http://buildbot.net/)，又或者给镜像打上 bag 和 tag （相关阅读：[docker commit](http://docs.docker.io/en/latest/reference/commandline/cli/#commit)，[docker tag](http://docs.docker.io/en/latest/reference/commandline/cli/#tag)），再发送到 [Docker Registry](https://github.com/dotcloud/docker-registry)。可是如果镜像中的代码是私有的，你不想把镜像放到公共的 [Docker Registry](http://index.docker.io) 上呢？[Docker](http://docker.io) 公司已经想到了这一点，并因此建立了 [docker-registry](https://github.com/dotcloud/docker-registry) 项目。[docker-registry](https://github.com/dotcloud/docker-registry) 允许你把自己的镜像 [push](http://docs.docker.io/en/latest/reference/commandline/cli/#push) 到自己的 registry 中，酷！


如果你想感受一下[docker registry](https://github.com/dotcloud/docker-registry)，可以用公共的 [registry](http://index.docker.io) 来试试：

    $ docker pull samalba/docker-registry
    $ docker run -d -p 5000:5000 samalba/docker-registry
    # 我们先pull下来一个简单的镜像（或者自己做一个也可以）
    $ docker pull busybox
    $ docker tag busybox localhost:5000/busybox
    $ docker push localhost:5000/busybox



对于 registry 入门，这个例子很有用，但是例子中仅用了一个简单的 HTTP 服务。任何知道服务器地址的人都可以随意 push 镜像，这不是个好方案。下面我们来建立自己的私有 registry 以供内部使用。


##准备自己的部署方案

我们要创建一个 Ubuntu 服务器来部署 registry，在此之前，我们先考虑几件事情...

###用什么作为后台存储？

我们用什么来做后台存储呢？请看下面几种存储方案：

 - local：用本地存储
 - s3：存到 Amazon S3 的 bucket
 - swift：存到 OpenStack 的 Swift 容器
 - glance：使用 OpenStack 的 Glance 项目
 - elliptics：使用 Elliptics 的键值存储方案

这些方案的 python 脚本在 [这里](https://github.com/dotcloud/docker-registry/tree/master/lib/storage)，大家可以参考。

注：Openstack Swift 这个方案是我自己写的，如果发现 bug，欢迎在 docker registry 的 gituhb 主页提出。

译者注：国内的开发者 [桂阳](http://weibo.com/u/1656755095) 贡献了存储在阿里云的 [方案](https://github.com/guiyang/docker-registry/blob/aliyun-oss/lib/storage/aliyun_oss.py)。

###托管服务器还是用自己搭建服务器？

我们要把 docker registry 服务部署到哪里呢？用自己的 OpenStack 集群？Amazon 的网络服务？还是 Rackspace？或者自己购买服务器？答案是：用什么都行！

使用云服务，我们可以使用可扩展的存储空间，便于我们管理自己的备份，非常方便。

###用什么操作系统？

docker registry 是用 python 写的，所以把它导入到各种操作系统中真是太简单了。你可以轻轻松松的写一个 [systemd配置文件](https://wiki.archlinux.org/index.php/systemd#Writing_custom_.service_files) ，或者把它做成 [Widnows服务](http://en.wikipedia.org/wiki/Windows_service) 。本例中，我们在 Ubuntu 上安装 docker registry，因此，我们用 [upstart](http://upstart.ubuntu.com/)来管理 Gunicorn 进程。

接下来的部署工作都在本地硬盘里进行。我们自己的办公室里有一个内部的 Openstack 集群（我爱 Openstack！），因此我们用它来作服务器托管，域名用 “docker-internal.example.com”，服务器系统则用 [Ubuntu Cloud image](http://cloud-images.ubuntu.com/releases/12.04.3/release/)，版本为12.04.3。

万事俱备，开工！

###启动服务器

首先，启动服务器。因为我是用的内部 Openstack，我用 [nova客户端](https://github.com/openstack/python-novaclient) 来启动就可以了。如果你按照本例来操作，请在 .bashrc 文件中设置下面列出的验证信息：


	$ cat ~/.bashrc
	[...]
	export OS_AUTH_URL=http://******/v2.0
	export OS_TENANT_ID=******
	export OS_TENANT_NAME="******"
	export OS_USERNAME=******
	export OS_PASSWORD="******"
	[...]


设置完成后，请用下面的命令测试一下：

	$ sudo pip install python-novaclient
	$ nova list


启动服务器之前，我们先上传 Ubuntu cloud image 和自己的 SSH Key 文件：

	$ nova keypair-add --pub-key ~/.ssh/id_rsa.pub bacongobbler
	$ sudo pip install python-glanceclient
	$ glance image-create --name ubuntu-12.04.3-server-cloudimg-amd64 --disk-format qcow2 --container-format bare --location http://cloud-images.ubuntu.com/releases/12.04.3/release/ubuntu-12.04-server-cloudimg-amd64-disk1.img


再创建一个安全组，来允许外部对 80 和 443 端口的访问：

	$ nova secgroup-create web-server "security group for standard web servers"
	$ nova secgroup-add-rule web-server tcp 80 80 0.0.0.0/0
	$ nova secgroup-add-rule web-server tcp 443 443 0.0.0.0/0


现在，我们来创建一个 512G 的分区，用于存储我们的 docker 镜像：

	$ nova volume-create 512 --display-name docker-internal


最后，启动服务器吧！

	$ nova boot docker-internal --image ubuntu-12.04.3-server-cloudimg-amd64 --flavor m1.medium --security-groups web-server --key-name bacongobbler
	$ # do some grepping for the volume ID
	$ VOLUME_ID=$(nova volume-list | grep docker-internal | awk '{print $2}')
	$ nova volume-attach docker-internal $VOLUME_ID /dev/vdb
	$ nova floating-ip-list
	+----------------+--------------------------------------+---------------+------+
	| Ip             | Instance Id                          | Fixed Ip      | Pool |
	+----------------+--------------------------------------+---------------+------+
	| 192.168.68.222 | 79caf450-7b23-46bd-839a-abec7408a2c0 | 192.168.32.26 | nova |
	| 192.168.68.224 | a10cb949-09b6-4533-9733-860a5f8fdff4 | 192.168.32.19 | nova |
	| 192.168.68.225 | None                                 | None          | nova |
	| 192.168.68.236 | None                                 | None          | nova |
	| 192.168.68.237 | dc835a69-2894-4278-aebe-4f9ca6363724 | 192.168.32.12 | nova |
	| 192.168.68.238 | 4a8835b6-a318-44b5-897d-2320977cfe01 | 192.168.32.20 | nova |
	| 192.168.68.239 | afde96f2-9bac-441a-a0c7-589ace2ac6b9 | 192.168.32.15 | nova |
	| 192.168.68.246 | 00ceedf4-8d85-4ea5-8f42-78a1ab521a62 | 192.168.32.13 | nova |
	| 192.168.68.250 | c1ef2314-6067-464d-85ec-de2a26a80f3e | 192.168.32.4  | nova |
	| 10.3.4.1       | 192dadcc-e786-4366-8091-2e9a364a65cf | 192.168.32.17 | nova |
	+----------------+--------------------------------------+---------------+------+
	$ nova add-floating-ip docker-internal 192.168.68.236


等一小会儿，并把子域名 docker-internal 绑定到当前的 IP，然后用 SSH 登陆：

	$ ssh ubuntu@docker-internal.example.com


哦耶！搞定！

##部署和配置 registry


我们已经有自己的服务器了，下面我们来装几个必要软件吧。

    # 安装软件之前，我们先更新一下软件源列表，然后重启
    ubuntu@docker-internal:~$ sudo apt-get update
    ubuntu@docker-internal:~$ sudo apt-get upgrade
    ubuntu@docker-internal:~$ sudo reboot now

    #登陆到docker registry
    $ ssh ubuntu@docker-internal.example.com
    
    # 切换到root用户
    ubuntu@docker-internal:~$ sudo su
    
    # 安装nginx-extras包，我们要用其中的chunkin模块
    root@docker-internal:~# apt-get install git nginx-extras
    
    # 安装apache2-utils包，这样我们就可以用htpasswd命令来设置密码
    root@docker-internal:~# apt-get install apache2-utils
    
    # 安装一些必要的依赖
    root@docker-internal:~# apt-get install build-essential libevent-dev libssl-dev liblzma-dev python-dev python-pip
    
    # 安装redis来实现我们的LRU缓存策略
    root@docker-internal:~# apt-get install redis-server
    root@docker-internal:~# apt-get clean


必要的软件都装好了，下面我们就来安装 docker registry：

    root@docker-internal:~# git clone https://github.com/dotcloud/docker-registry.git /opt/docker-registry
    root@docker-internal:~# cd /opt/docker-registry
    
    # 切换到最新的 registry 版本 
    root@docker-internal:~# git checkout 0.6.3
    
    # 创建日志路径
    root@docker-internal:~# mkdir -p /var/log/docker-registry
    
    # 安装 pip 包
    root@docker-internal:~# pip install -r requirements.txt
    root@docker-internal:~# cp config/config_sample.yml


如果一切顺利，我们现在应该可以用下面的命令来测试一下 docker registry 了：

    root@docker-internal:~# ./wsgi.py
    2014-01-13 23:38:38,470 INFO:  * Running on http://0.0.0.0:5000/
    2014-01-13 23:38:38,470 INFO:  * Restarting with reloader


如果你看到的结果和上面一样，那么，恭喜你，成功了！接下来我们需要设置一些选项，记得我们之前分配给 docker registry 的分区吗？现在我们就来挂在这个分区：

    root@docker-internal:~# mkdir -p /data/registry
    root@docker-internal:~# mkfs.ext4 /dev/vdb
    root@docker-internal:~# mount /dev/vdb /data/registry


现在，我们来编辑 docker registry 的配置文件，我们用 http://uuidgenerator.net 在线生成密钥：

    root@docker-internal:~# cat << EOF > /opt/docker-registry/config/config.yml
    # The 'common' part is automatically included (and possibly overriden by
    # all other flavors)
    common:
        # Set a random string here
        secret_key: REPLACEME
        standalone: true
     # This is the default configuration when no flavor is specified
    dev:
        storage: local
        storage_path: /tmp/registry
        loglevel: debug
    # To specify another flavor, set the environment variable SETTINGS_FLAVOR
    # $ export SETTINGS_FLAVOR=prod
    prod:
        storage: local
        storage_path: /data/registry
        loglevel: info
        # Enabling LRU cache for small files. This speeds up read/write on
        # small files when using a remote storage backend (like S3).
        cache:
            host: localhost
            port: 6379
        cache_lru:
            host: localhost
            port: 6379
    EOF


然后为 docker registry 设置一个 upstart 作业：

    root@docker-internal:~# cat << EOF > /etc/init/docker-registry.conf
    description "Docker Registry"
    version "0.6.3"
    author "Docker, Inc."
    
    start on runlevel [2345]
    stop on runlevel [016]
    
    respawn
    respawn limit 10 5
    
    # set environment variables
    env REGISTRY_HOME=/opt/docker-registry
    env SETTINGS_FLAVOR=prod
    
    script
    cd $REGISTRY_HOME
    exec gunicorn -k gevent --max-requests 100 --graceful-timeout 3600 -t 3600 -b 0.0.0.0:5000 -w 8 --access-logfile /var/log/docker-registry/access.log --error-logfile /var/log/docker-registry/server.log wsgi:application
    end script
    EOF


用下面的命令启动 registry 的作业：

    root@docker-internal:~# start docker-registry
    docker-registry start/running, process 10872


用下面的命令检查 registry 的作业是否运行：

    root@docker-internal:~# cat /var/log/docker-registry/server.log
    2014-01-14 00:33:44 [15051] [INFO] Starting gunicorn 18.0
    2014-01-14 00:33:44 [15051] [INFO] Listening at: http://0.0.0.0:5000 (15051)
    2014-01-14 00:33:44 [15051] [INFO] Using worker: gevent
    2014-01-14 00:33:44 [15056] [INFO] Booting worker with pid: 15056
    2014-01-14 00:33:44 [15057] [INFO] Booting worker with pid: 15057
    2014-01-14 00:33:44 [15062] [INFO] Booting worker with pid: 15062
    2014-01-14 00:33:45 [15067] [INFO] Booting worker with pid: 15067
    2014-01-14 00:33:45 [15068] [INFO] Booting worker with pid: 15068
    2014-01-14 00:33:45 [15069] [INFO] Booting worker with pid: 15069
    2014-01-14 00:33:45 [15070] [INFO] Booting worker with pid: 15070
    2014-01-14 00:33:45 [15071] [INFO] Booting worker with pid: 15071


接下来，我们设置 nginx：

    root@docker-internal:~# rm /etc/nginx/sites-enabled/default
    root@docker-internal:~# cat << EOF > /etc/nginx/sites-enabled/docker-registry
` nginx的配置：`

    upstream docker-registry {
      server localhost:5000;
    }
    
    server {
      listen 443;
      server_name docker-internal.example.com;
    
      ssl on;
      ssl_certificate /etc/ssl/certs/docker-registry.crt;
      ssl_certificate_key /etc/ssl/private/docker-registry.key;
    
      proxy_set_header Host             $http_host;   # required for docker client's sake
      proxy_set_header X-Real-IP        $remote_addr; # pass on real client's IP
      proxy_set_header Authorization    ""; # see https://github.com/dotcloud/docker-registry/issues/170
    
      client_max_body_size 0; # disable any limits to avoid HTTP 413 for large image uploads
       
      # required to avoid HTTP 411: see Issue #1486 (https://github.com/dotcloud/docker/issues/1486)
      chunkin on;
      error_page 411 = @my_411_error;
      location @my_411_error {
        chunkin_resume;
      }
    
      location / {
        auth_basic              "Restricted";
        auth_basic_user_file    docker-registry.htpasswd;
    
        proxy_pass http://docker-registry;
        proxy_set_header Host $host;
        proxy_read_timeout 900;
      }
    
      location /_ping {
        auth_basic off;
        proxy_pass http://docker-registry;
      }
    
      location /v1/_ping {
        auth_basic off;
        proxy_pass http://docker-registry;
      }
    }
    EOF
` `    
    
    root@docker-internal:~# service nginx restart


别忘了在 htpasswd 文件里设置账号密码：

    root@docker-internal:~# htpasswd -bc /etc/nginx/docker-registry.htpasswd USERNAME PASSWORD


我们还要在服务器上安装一个 SSL 密钥。本例中，假设我们已经有认证机构颁发的 SSL证 书了，SSL 授权给'docker-internal.example.com'或者'*.example.com'，用下面的命令来安装 SSL 密钥：

    root@docker-internal:~# mv server.key /etc/ssl/private/docker-registry.key
    root@docker-internal:~# mv server.crt /etc/ssl/certs/docker-registry.crt


如果你不打算花钱去搞一个认证机构授权的SSL密钥，或者你只是练习着部署 docker registry，那么你也可以按照 [Akadia的教程](http://www.akadia.com/services/ssh_test_certificate.html) 装一个自己授权的 SSL key，如下：

    root@docker-internal:~# openssl genrsa -des3 -out server.key 1024
    root@docker-internal:~# openssl req -new -key server.key -out server.csr
    root@docker-internal:~# cp server.key server.key.org
    root@docker-internal:~# openssl rsa -in server.key.org -out server.key
    root@docker-internal:~# openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt


请注意，现在官方的 docker 还不能用自授权的证书，要等到 [`#2687`](https://github.com/dotcloud/docker/pull/2687) 的 pull request 合并到官方 master 分支后才能使用。或者，你也可以试着修改 docker 的源代码来让它支持自授权证书。

##测试


最后，我们来测试一下自己的 docker registry：

    root@docker-internal:~# exit
    ubuntu@docker-internal:~$ exit
    $ curl -u bacongobbler:******* https://docker-internal.example.com
    "docker-registry server (prod)"
    $ docker login https://docker-internal.example.com
    Login against server at https://docker-internal.example.com/v1/
    Username (): bacongobbler
    Login Succeeded
    $ docker pull busybox
    Pulling repository busybox
    e9aa60c60128: Download complete 
    $ docker tag busybox docker-internal.example.com/busybox
    $ docker push docker-internal.example.com/busybox
    The push refers to a repository [docker-internal.example.com/busybox] (len: 1)
    Sending image list
    Pushing repository docker-internal.example.com/busybox (1 tags)
    Pushing tags for rev [e9aa60c60128] on {https://docker-internal.example.com/v1/repositories/busybox/tags/latest}
    e9aa60c60128: Image already pushed, skipping


完成！现在我们在 Openstack 上部署了一个 docker registry，随时可用！


##下一步


部署好 docker registry 后，我们还可以进一步让它跑的更好，比如：

- 当docker registry崩溃的时候，发出邮件通知
- 用 [logstash](http://logstash.net/) 或其他日志软件来管理日志
- 把docker registry部署到CentOS或者RHEL
- 做一些测试，看看docker registry的性能如何

这只是我想到的一些，如果你有什么好点子，请在下面留言！

在 ActiveState，我们在 [Stackato v3](http://www.activestate.com/stackato) 项目中大量的使用了 Docker。Phil 在博客中全面的介绍了 Stackat v3，如果你还没来得及看，一定要去拜读他的 [大作](http://www.activestate.com/blog/2013/11/technical-look-stackato-v30-beta) ，特别是其中 [Stackato在哪些地方适合用Docker](http://www.activestate.com/blog/2013/11/technical-look-stackato-v30-beta#docker) 这一节。



---
####这篇文章由 [Matthew Fisher](http://www.activestate.com/blog/authors/matthewf) 发表，点击[此处](http://www.activestate.com/blog/2014/01/deploying-your-own-private-docker-registry)可查阅原文。

####The article was contributed by [Matthew Fisher](http://www.activestate.com/blog/authors/matthewf), click [here](http://www.activestate.com/blog/2014/01/deploying-your-own-private-docker-registry) to read the original publication.


