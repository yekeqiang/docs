# 构建一个高可用及自动发现的Docker基础架构

---

##### 作者：刘天斯

---

Docker的生态日趋成熟，开源社区也不断孵化出优秀的周边项目，覆盖网络、监控、维护、部署、开发等方面。帮助开发、运维人员快速构建、运营Docker服务环境，其中也不乏有大公司的影子，如Google、IMB、Redhat，甚至微软也宣称后续将提供Docker在Windows平台的支持。Docker的发展前景一片大好。但在企业当中，如何选择适合自己的Docker构建方案？可选的方案有kubernetes与CoreOS（都已整合各类组件），另外一种方案为Haproxy+etcd+confd，采用松散式的组织结构，但各个组件之间的通讯是非常严密的，且扩展性更强，定制也更加灵活。下面详细介绍如何使用 Haproxy+etcd+confd 构建一个高可用及自动发现的Docker基础架构。

## 一、架构优势

约定由 Haproxy + etcd + confd + Docker 构建的基础服务平台简称 “HECD”  架构，整合了多种开源组件，看似松散的结构，事实上已经是一个有机的整体，它们互相联系、互相作用，是 Docker 生态圈中最理想的组合之一，具有以下优势：

- 自动、实时发现及无感知服务刷新；
- 支持任意多台Docker主宿机；
- 支持多种APP接入且打散至不分主宿机；
- 采用Etcd存储信息，集群支持可靠性高；
- 采用Confd配置引擎，支持各类接入层，如Nginx；
- 支持负载均衡、故障迁移；
- 具备资源弹性，伸缩自如（通过生成、销毁容器实现）；

## 二、架构说明

在HECD架构中，首先管理员操作 Docker Client ，除了提交容器（ Container ）启动与停止指令外，还通过 REST-API 方式向 Etcd（K/V） 存储组件注册容器信息，包括容器名称、主宿机IP、映射端口等。Confd配置组件会定时查询Etcd组件获取最新的容器信息，根据定义好的配置模板生成 Haproxy 配置文件 Haproxy.cfg ，并且自动 reload haproxy 服务。用户在访问业务服务时，完全没有感知后端APP的上线、下线、切换及迁移，达到了自动发现、高可用的目的。详细架构图见图1-1。

![alt](http://resource.docker.cn/platform-architect.jpg)

图1-1 平台架构图

为了方便大家理解各组件间的关系，通过图1-2进行架构流程梳理，首先管理员通过Shell或api操作容器，下一步将容器信息注册到Etcd组件，Confd组件会定时查询Etcd，获取已经注册到Etcd中容器信息，最后通过Confd的模板引擎生成Haproxy配置，整个流程结束。

![alt](http://resource.docker.cn/architect-procedure.jpg)

图1-2架构流程图


了解架构流程后，我们逐一对流程中各组件进行详细介绍。

### 1. Etcd介绍

Etcd是一个高可用的 Key/Value 存储系统，主要用于分享配置和服务发现。

- 简单：支持 curl 方式的用户 API (HTTP+JSON)
- 安全：可选 SSL 客户端证书认证
- 快速：单实例可达每秒 1000 次写操作
- 可靠：使用 Raft 实现分布式

### 2. Confd介绍

Confd是一个轻量级的配置管理工具。通过查询Etcd，结合配置模板引擎，保持本地配置最新，同时具备定期探测机制，配置变更自动reload。

### 3. Haproxy介绍

HAProxy是提供高可用性、负载均衡以及基于TCP和HTTP应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案。（来源百科）

## 三、架构部署

平台环境基于Centos6.5+Docker1.2构建，其中Etcd的版本为etcd version 0.5.0-alpha，Confd版本为confd 0.6.2，Haproxy版本为HA-Proxy version 1.4.24。下面对平台的运行环境、安装部署、组件说明等进行详细说明，环境设备角色表如下：

![alt](http://resource.docker.cn/env-equipments.jpg)

### 1. 组件安装

#### 1.1 Docker安装

SSH终端登录192.168.1.22服务器，执行以下命令：

```
# yum -y install docker-io  
# service docker start  
# chkconfig docker on
```

#### 1.2 Haproxy、confd安装

SSH终端登录192.168.1.20服务器，执行以下命令：

```
1、haproxy  
# yum –y install haproxy  
  
2、confd  
# wget https://github.com/kelseyhightower/confd/releases/download/v0.6.3/confd-0.6.3-linux-amd64  
# mv confd /usr/local/bin/confd  
# chmod +x /usr/local/bin/confd  
# /usr/local/bin/confd -version  
confd 0.6.2  
```

#### 1.3 Etcd安装

SSH终端登录192.168.1.21服务器，执行以下命令：

```
# yum -y install golang  
# mkdir -p /home/install && cd /home/install  
# git clone https://github.com/coreos/etcd  
# cd etcd  
# ./build  
# cp bin/etcd /bin/etcd  
# /bin/etcd -version  
etcd version 0.5.0-alpha  
```

### 2. 组件配置

#### 2.1 Etcd配置

由于etcd是一个轻量级的K/V存储平台，启动时指定相关参数即可，无需配置。

```
# /bin/etcd -peer-addr 192.168.1.21:7001 -addr 192.168.1.21:4001 -data-dir /data/etcd -peer-bind-addr 0.0.0.0:7001 -bind-addr 0.0.0.0:4001 &  
```

由于etcd具备多机支持，参数“-peer-addr”指定与其它节点通讯的地址；参数“-addr”指定服务监听地址；参数“-data-dir”为指定数据存储目录。

由于etcd是通过REST-API方式进行交互，常见操作如下：

1) 设置(set) key操作

```
# curl -L http://192.168.1.21:4001/v2/keys/mykey -XPUT -d value="this is awesome"  
{"action":"set","node":{"key":"/mykey","value":"this is awesome","modifiedIndex":28,"createdIndex":28}}  
```

2) 获取(get) key信息

```
# curl -L http://192.168.1.21:4001/v2/keys/mykey  
{"action":"get","node":{"key":"/mykey","value":"this is awesome","modifiedIndex":28,"createdIndex":28}}  
```

3) 删除key信息

```
# curl -L http://192.168.1.21:4001/v2/keys/mykey -XDELETE         {"action":"delete","node":{"key":"/mykey","modifiedIndex":29,"createdIndex":28},"prevNode":{"key":"/mykey","value":"this is awesome","modifiedIndex":28,"createdIndex":28}}  
```

更多操作API见https://github.com/coreos/etcd/blob/master/Documentation/api.md。

#### 2.2 Confd+Haproxy配置

由于Haproxy的配置文件是由Confd组件生成，要求Confd务必要与haproxy安装在同一台主机上，Confd的配置有两种，一种为Confd资源配置文件，默认路径为“/etc/confd/conf.d”目录，另一种为配置模板文件，默认路径为“/etc/confd/templates”。具体配置如下：

创建配置文件目录

`# mkdir -p /etc/confd/{conf.d,templates}`

（1）配置资源文件

详细见以下配置文件，其中“src”为指定模板文件名称（默认到路径/etc/confd/templates中查找）；“dest”指定生成的Haproxy配置文件路径；“keys”指定关联Etcd中key的URI列表；“reload_cmd”指定服务重载的命令，本例中配置成haproxy的reload命令。

【/etc/confd/conf.d/ haproxy.toml】

```
[template]  
src = "haproxy.cfg.tmpl"  
dest = "/etc/haproxy/haproxy.cfg"  
keys = [  
  "/app/servers",  
]  
reload_cmd = "/etc/init.d/haproxy reload"  
```

（2）配置模板文件

Confd模板引擎采用了Go语言的文本模板，更多见http://golang.org/pkg/text/template/，具备简单的逻辑语法，包括循环体、处理函数等，本示例的模板文件如下，通过range循环输出Key及Value信息。

【/etc/confd/templates/haproxy.cfg.tmpl】

```
global  
        log 127.0.0.1 local3  
        maxconn 5000  
        uid 99  
        gid 99  
        daemon  
  
defaults  
        log 127.0.0.1 local3  
        mode http  
        option dontlognull  
        retries 3  
        option redispatch  
        maxconn 2000  
        contimeout  5000  
        clitimeout  50000  
        srvtimeout  50000  
  
listen frontend 0.0.0.0:80  
        mode http  
        balance roundrobin  
        maxconn 2000  
        option forwardfor  
        {{range gets "/app/servers/*"}}  
        server {{base .Key}} {{.Value}} check inter 5000 fall 1 rise 2  
        {{end}}  
  
        stats enable  
        stats uri /admin-status  
        stats auth admin:123456  
        stats admin if TRUE  
```

（3）模板引擎说明

本小节详细说明Confd模板引擎基础语法与示例，下面为示例用到的KEY信息。

```
# curl -XPUT http://192.168.1.21:4001/v2/keys/app/servers/backstabbing_rosalind -d value="192.168.1.22:49156"  
# curl -XPUT http://192.168.1.21:4001/v2/keys/app/servers/cocky_morse -d value="192.168.1.22:49158"  
# curl -XPUT http://192.168.1.21:4001/v2/keys/app/servers/goofy_goldstine -d value="192.168.1.22:49160"  
# curl -XPUT http://192.168.1.21:4001/v2/keys/app/servers/prickly_blackwell -d value="192.168.1.22:49162"  
```

- base

作为path.Base函数的别名，获取路径最后一段。


{{ with get "/app/servers/prickly_blackwell"}}
    server {{base .Key}} {{.Value}} check
{{end}}

```
prickly_blackwell 192.168.1.22:49162  
```

- get

返回一对匹配的KV，找不到则返回错误。

{{with get "/app/servers/prickly_blackwell"}}
    key: {{.Key}}
    value: {{.Value}}
{{end}}

```
/app/servers/prickly_blackwell 192.168.1.22:49162  
```

- gets

返回所有匹配的KV，找不到则返回错误。

{{range gets "/app/servers/*"}}
    {{.Key}} {{.Value}}
     {{end}}

```
/app/servers/backstabbing_rosalind 192.168.1.22:49156  
/app/servers/cocky_morse 192.168.1.22:49158  
/app/servers/goofy_goldstine 192.168.1.22:49160  
app/servers/prickly_blackwell 192.168.1.22:49162  
```

- getv

返回一个匹配key的字符串型Value，找不到则返回错误。

{{getv "/app/servers/cocky_morse"}}

```
192.168.1.22:49158
```  

- getvs 

返回所有匹配key的字符串型Value，找不到则返回错误。

{{range getvs "/app/servers/*"}}
  value: {{.}}
 {{end}}

```
value: 192.168.1.22:49156  
value: 192.168.1.22:49158  
value: 192.168.1.22:49160  
value: 192.168.1.22:49162  
```

- split

对输入的字符串做split处理，即将字符串按指定分隔符拆分成数组。
        {{ $url := split (getv "/app/servers/cocky_morse") ":" }}
          host: {{index $url 0}}
          port: {{index $url 1}}

```
host: 192.168.1.22  
port: 49158  
```

- ls

返回所有的字符串型子key，找不到则返回错误。

{{range ls "/app/servers/"}}
   subkey: {{.}}
{{end}}

```
subkey: backstabbing_rosalind  
subkey: cocky_morse  
subkey: goofy_goldstine  
subkey: prickly_blackwell  
```

- lsdir

返回所有的字符串型子目录，找不到则返回一个空列表。

{{range lsdir "/app/"}}
   subdir: {{.}}
{{end}}

```subdir: servers```  

（4）启动confd及haproxy服务

下面为启动Confd服务命令行，参数“interval”为指定探测etcd的频率，单位为秒，参数“-node”为指定etcd监听服务主地址，以便获取容器信息。

```
# /usr/local/bin/confd -verbose -interval 10 -node '192.168.1.21:4001' -confdir /etc/confd > /var/log/confd.log &  
# /etc/init.d/haproxy start  
```

### 3. 容器配置

前面HECD架构说明内容，有讲到容器的操作会即时注册到etcd组件中，是通过curl命令进行REST-API方式提交的，下面详细介绍通过SHELL及Python-api两种方式的实现方法，支持容器启动、停止的联动。

#### 3.1 SHELL实现方法

实现的原理是通过获取“Docker run ***”命令输出的Container ID，通过“docker inspect Container ID”得到详细的容器信息，分析出容器服务映射的外部端口及容器名称，将以“/app/servers/容器名称”作为Key，“主宿机: 映射端口”作为Value注册到etcd中。其中Key信息前缀(/app/servers)与“/etc/confd/conf.d/haproxy.toml”中的keys参数是保持一致的。
【docker.sh】

```
#!/bin/bash  
  
if [ -z $1 ]; then  
        echo "Usage: c run <image name>:<version>"  
        echo "       c stop <container name>"  
        exit 1  
fi  
  
if [ -z $ETCD_HOST ]; then  
  ETCD_HOST="192.168.1.21:4001"  
fi  
  
if [ -z $ETCD_PREFIX ]; then  
  ETCD_PREFIX="app/servers"  
fi  
  
if [ -z $CPORT ]; then  
  CPORT="80"  
fi  
  
if [ -z $FORREST_IP ]; then  
  FORREST_IP=`ifconfig eth0| grep "inet addr" | head -1 | cut -d : -f2 | awk '{print $1}'`  
fi  
  
function launch_container {  
        echo "Launching $1 on $FORREST_IP ..."  
  
        CONTAINER_ID=`docker run -d --dns 172.17.42.1 -P -v /data:/data -v /etc/httpd/conf:/etc/httpd/conf -v /etc/httpd/conf.d:/etc/httpd/conf.d -v /etc/localtime:/etc/localtime:ro $1 /bin/sh -c "/usr/bin/supervisord -c /etc/supervisord.conf"`  
        PORT=`docker inspect $CONTAINER_ID|grep "\"Ports\"" -A 50|grep "\"$CPORT/tcp\"" -A 3| grep HostPort|cut -d '"' -f4|head -1`  
        NAME=`docker inspect $CONTAINER_ID | grep Name | cut -d '"' -f4 | sed "s/\///g"|sed -n 2p`  
  
        echo "Announcing to $ETCD_HOST..."  
        curl -XPUT "http://$ETCD_HOST/v2/keys/$ETCD_PREFIX/$NAME" -d value="$FORREST_IP:$PORT"  
  
        echo "$1 running on Port $PORT with name $NAME"  
}  
  
function stop_container {  
        echo "Stopping $1..."  
        CONTAINER_ID=`docker ps -a| grep $1 | awk '{print $1}'`  
        echo "Found container $CONTAINER_ID"  
        docker stop $CONTAINER_ID  
  echo http://$ETCD_HOST/v2/keys/$ETCD_PREFIX/$1  
        curl -XDELETE http://$ETCD_HOST/v2/keys/$ETCD_PREFIX/$1 &> /dev/null  
        echo "Stopped."  
}  
  
  
if [ $1 = "run" ]; then  
  launch_container $2  
else  
  stop_container $2  
fi  
```

**docker.sh使用方法**:

- 启动一个容器

  `# ./docker.sh run yorko/webserver:v3(镜像)`
  
- 停止一个容器

  `# ./docker.sh stop berserk_hopper(容器名)`

#### 3.2 Docker-py API实现方法

通过Python语言调用Docker-py的API实现容器的远程操作（创建、运行、停止），并结合python-etcd模块对etcd进行操作(set、delete)，达到与SHELL方式一样的效果，很明显，Docker-py方式更加容易扩展，可以无缝与现有运营平台对接。

为兼顾到远程API支持，需对docker启动文件“exec”处进行修改，详细见如下：

`# vi /etc/init.d/docker`

```
$exec -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -d &>> $logfile &  
```

- 启动容器的程序如下：

【docker_run.py】

```
#!/usr/local/Python/bin/python  
import docker  
import etcd  
import sys  
  
Etcd_ip="192.168.1.21"  
Server_ip="192.168.1.22"  
App_port="80"  
App_protocol="tcp"  
Image="yorko/webserver:v3"  
  
Port=""  
Name=""  
  
idict={}  
rinfo={}  
try:  
    c = docker.Client(base_url='tcp://'+Server_ip+':2375',version='1.14',timeout=15)  
except Exception,e:  
    print "Connection docker server error:"+str(e)  
    sys.exit()  
  
try:  
    rinfo=c.create_container(image=Image,stdin_open=True,tty=True,command="/usr/bin/supervisord -c /etc/supervisord.conf",volumes=['/data','/etc/httpd/conf','/etc/httpd/conf.d  
','/etc/localtime'],ports=[80,22],name=None)  
    containerId=rinfo['Id']  
except Exception,e:  
    print "Create docker container error:"+str(e)  
    sys.exit()  
  
try:  
    c.start(container=containerId, binds={'/data':{'bind': '/data','ro': False},'/etc/httpd/conf':{'bind': '/etc/httpd/conf','ro': True},'/etc/httpd/conf.d':{'bind': '/etc/htt  
pd/conf.d','ro': True},'/etc/localtime':{'bind': '/etc/localtime','ro': True}}, port_bindings={80:None,22:None}, lxc_conf=None,publish_all_ports=True, links=None, privileged=F  
alse,dns='172.17.42.1', dns_search=None, volumes_from=None, network_mode=None,restart_policy=None, cap_add=None, cap_drop=None)  
except Exception,e:  
    print "Start docker container error:"+str(e)  
    sys.exit()  
  
try:  
    idict=c.inspect_container(containerId)  
    Name=idict["Name"][1:]  
    skey=App_port+'/'+App_protocol  
    for _key in idict["NetworkSettings"]["Ports"].keys():  
        if _key==skey:  
            Port=idict["NetworkSettings"]["Ports"][skey][0]["HostPort"]  
except Exception,e:  
    print "Get docker container inspect error:"+str(e)  
    sys.exit()  
  
if Name!="" and Port!="":  
    try:  
        client = etcd.Client(host=Etcd_ip, port=4001)  
        client.write('/app/servers/'+Name, Server_ip+":"+str(Port))  
        print Name+" container run success!"  
    except Exception,e:  
        print "set etcd key error:"+str(e)  
else:  
    print "Get container name or port error."  
```

- 停止容器的程序如下：

【docker_stop.py】

```
#!/usr/local/Python/bin/python  
import docker  
import etcd  
import sys  
  
Etcd_ip="192.168.1.21"  
Server_ip="192.168.1.22"  
containerName="grave_franklin" #指定需要停止容器的名称  
  
try:  
    c = docker.Client(base_url='tcp://'+Server_ip+':2375',version='1.14',timeout=10)  
    c.stop('furious_heisenberg')  
except Exception,e:  
    print str(e)  
    sys.exit()  
  
try:  
    client = etcd.Client(host=Etcd_ip, port=4001)  
    client.delete('/app/servers/'+containerName)  
    print containerName+" container stop success!"  
except Exception,e:  
print str(e)  
```

> ##### 注意：由于容器是无状态的，尽量让其以松散的形式存在，映射端口选项要求使用“-P”参数，即使用随机端口的模式，减少人手干预。

## 四、业务上线

HECD架构已部署完毕，接下来就是让其为我们服务，案例中使用的镜像“yorko/webserver:v3”为已经构建好的LAMP平台。类似的镜像也可以在docker-pub中下载到，开始跑起，运行 dockery.sh 创建两个容器：

```
# ./docker.sh run yorko/webserver:v3  
Launching yorko/webserver:v3 on 192.168.1.22 ...  
Announcing to 192.168.1.21:4001...  
{"action":"set","node":{"key":"/app/servers/berserk_hopper","value":"192.168.1.22:49170","modifiedIndex":33,"createdIndex":33}}  
yorko/webserver:v3 running on Port 49170 with name berserk_hopper  
  
# ./docker.sh run yorko/webserver:v3  
Launching yorko/webserver:v3 on 192.168.1.22 ...  
Announcing to 192.168.1.21:4001...  
{"action":"set","node":{"key":"/app/servers/lonely_meitner","value":"192.168.1.22:49172","modifiedIndex":34,"createdIndex":34}}  
yorko/webserver:v3 running on Port 49172 with name lonely_meitner  
```

访问Haproxy监控地址：http://192.168.1.20/admin-status，刚创建的容器已经添加到haproxy中，见图1-3。

![alt](http://resource.docker.cn/haproxy-momitor.jpg)

图1-3 Haproxy监控后台截图


### 观察Haproxy的配置文件(更新部分)：

```
# vi /etc/haproxy/haproxy.cfg  
… …  
listen frontend 0.0.0.0:80  
        mode http  
        balance roundrobin  
        maxconn 2000  
        option forwardfor  
        server berserk_hopper 192.168.1.22:49170 check inter 5000 fall 1 rise 2  
        server lonely_meitner 192.168.1.22:49172 check inter 5000 fall 1 rise 2  
… …  
```

### 访问php测试文件http://192.168.1.20/info.php

![alt](http://resource.docker.cn/php-testing.jpg)

图1-4 php测试文件截图


从图1-4可以看出，获取的服务器端IP为容器本身的IP地址（172.17.0.11），在System环境变量处输出容器名为“598cf10a50a2”的信息。

---

参考：
http://ox86.tumblr.com/post/90554410668/easy-scaling-with-docker-haproxy-and-confd
https://github.com/AVGP/forrest/blob/master/forrest.sh

---

本文原载于作者刘天斯的[博客](http://blog.liuts.com/index.php)，我们获得其授权后转载。如果您有任何疑问，可以通过[邮件](mailto:liutiansi@gmail.com) 、[腾讯微博](http://t.qq.com/yorkoliu) 、[新浪微博](http://weibo.com/u/1775431677) 与其交流。您也可以阅读这篇文章的 [原始版本](http://blog.liuts.com/post/242/)。
