# Docker 远程 python API 操作容器一例

---

作者：刘天斯

---

[Docker-py](https://github.com/docker/docker-py) 作为官方推出的客户端 API ，功能可以满足我们大部分操作需求，API涉及镜像(images)及容器(CONTAINER)的功能操作，利用docker-py可以轻松开发出Docker的管理平台，以便维护大规模的Docker集群，本文介绍如何通过DockerFile创建一个WEB服务的镜像，再通过远程API对容器进行管理。

## 一、环境准备
1. 环境说明
	- 192.168.1.20 #Docker python API主机
	- 192.168.1.22 #Docker服务主机
2. Docker环境部署(略)
3. 修改自启动服务文件，支持远程TCP接口与本地SOCK连接；


	`# vi /etc/init.d/docker`

```
$exec -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -d &>> $logfile &  
```

	`#service docker restart`


## 二、创建镜像

1. 获取最新的centos镜像

	`# docker pull centos:latest`

2. 编写Dockerfile（支持apache+ssh服务）

	`# mkdir /home/Dockerfile/webserver`

	`# cd /home/Dockerfile/webserver`

	`# vi Dockerfile`

```
# This is a base comment  
FROM centos:latest  
MAINTAINER yorko Liu <liutiansi@gmail.com>  
  
#yum install Package  
RUN yum -y install net-tools  
RUN yum -y install iputils  iproute  man  vim-minimal  openssh-server  openssh-clients  
RUN yum -y install httpd  
RUN yum -y install python-setuptools  
RUN easy_install supervisor  
  
#set sshd  
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key  
RUN ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key  
RUN ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ""  
RUN sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd  
RUN mkdir -p /root/.ssh && chown root.root /root && chmod 700 /root/.ssh  
RUN echo 'root:Ksjhg34TDju' | chpasswd  
  
#set supervisor  
RUN mkdir -p /var/log/supervisor  
ADD supervisord.conf /etc/supervisord.conf  
  
#set port  
EXPOSE 22  
EXPOSE 80  
  
#set ENV  
ENV LANG en_US.UTF-8  
ENV LC_ALL en_US.UTF-8  
  
#run supervisor  
CMD ["/usr/bin/supervisord -c /etc/supervisord.conf"]  
```

通过supervisord来维护Docker容器中服务进程，编写 supervisord.conf

	`# vi supervisord.conf`

```
[supervisord]  
nodaemon=true  
  
[program:sshd]  
command=/usr/sbin/sshd -D  
  
[program:httpd]  
command=/usr/sbin/httpd -DFOREGROUND  
```

创建镜像，运行：

	`# docker build -t yorko/webserver:v1 .`

> 注：最后有一个`.`，别遗漏。

镜像生成完毕后运行docker images查看，见下图：

![alt](http://resource.docker.cn/docker-python-api-1.png)

## 三、编写操作 API

登录 192.168.1.20 服务器


	`# mkdir /home/test/docker-py`

	`# cd /home/test/docker-py`

1. 安装 docker-py

	`# wget https://github.com/docker/docker-py/archive/master.zip`

	`# unzip master`

	`# cd docker-py-master/`

	`# python setup.py install`

如正常导入模块（import docker）说明安装成功。

2. 创建容器 docker_create.py

```
import docker  
  
c = docker.Client(base_url='tcp://192.168.1.22:2375',version='1.14',timeout=10)  
c.create_container(image="yorko/webserver:v1",stdin_open=True,tty=True,command="/usr/bin/supervisord -c /etc/supervisord.conf",volumes=['/data'],ports=[80,22],name="webserver11")  
#通过create_container方法创建容器，指定"yorko/webserver:v1"镜像名称，使用supervisord接管进程服务，挂载主宿机/data作为数据卷，容器监听80与22端口，容器的名称为webserver11  
print str(r)  
```

3. 运行容器`docker_start.py

```
import docker  
  
c = docker.Client(base_url='tcp://192.168.1.22:2375',version='1.14',timeout=10)  
r=c.start(container='webserver11', binds={'/data':{'bind': '/data','ro': False}}, port_bindings={80:80,22:2022}, lxc_conf=None,  
        publish_all_ports=True, links=None, privileged=False,  
        dns=None, dns_search=None, volumes_from=None, network_mode=None,  
        restart_policy=None, cap_add=None, cap_drop=None)  
#通过start方法启动容器，指定数据卷的挂载关系及权限，以及端口与主宿机的映射关系等  
print str(r)  
```

4. 运行

	`# python docker_create.py`

	`# python docker_start.py`

更多 API 参考 [https://github.com/docker/docker-py](https://github.com/docker/docker-py)

5. 在 Docker 主机观察结果，见下图：

![alt](http://resource.docker.cn/docker-python-api-2.png)

## 三、校验服务

1. 校验 SSH 服务


![alt](http://resource.docker.cn/docker-python-api-3.png)

2. 校验 WEB 服务

![alt](http://resource.docker.cn/docker-python-api-4.png)

3. 检查数据卷

![alt](http://resource.docker.cn/docker-python-api-5.png)

---

本文原载于 [刘天斯](http://weibo.com/yorkoliu) 个人博客，原文地址：[Docker远程python API操作容器一例](http://blog.liuts.com/post/241/)