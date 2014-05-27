#Docker and MongoDB Sharded Cluster

[原文地址](http://sebastianvoss.com/docker-mongodb-sharded-cluster.html)

How Docker helps to create a complex MongoDB setup

This article describes how to create a Mongo DB Sharded Cluster using Docker. We will go through the creation of Dockerfiles and instanciation of the differnt moving parts of the cluster (replica sets, config servers and routers).

##Create the Dockerfiles

First we need to create two Dockerfiles - one for mongod and another one for mongos. Let’s start with mongod.

如何使用 docker 创建复杂的 mongodb 集群

本文描述了如何使用 docker 创建 mongodb shard 集群。我们将直接展示 mongodb 集群不同组件(副本集 replica sets 、配置服务器 config servers 以及资源路由 routes )的对应 dockerfile 文件以及实例化过程。

##创建Dockerfile文件

首先我们需要创建两个 dockerfile ，一个是 mongod 而另一个是 mongos 。先来看 mongod ：

```
FROM ubuntu:latest

# Add 10gen official apt source to the sources list
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/10gen.list

# Install MongoDB
RUN apt-get update
RUN apt-get install mongodb-10gen

# Create the MongoDB data directory
RUN mkdir -p /data/db

EXPOSE 27017
ENTRYPOINT ["usr/bin/mongod"]
```

The Dockerfile for mongos is even simpler as it is based on the previous one.

而为 mongos 准备的 dockerfile 就相当简单了，因为它基于前面做好的镜像。


```
FROM dev24/mongodb:latest

EXPOSE 27017
ENTRYPOINT ["usr/bin/mongos"]
```

Make sure you have stored each Dockerfile in its own directory.

请确保你已经将 dockerfile 存储在对应的位置了，如下：

```
docker_mongodb_cluster
|
|-- mongod
|   |-- Dockerfile
|
|-- mongos
    |-- Dockerfile
```

Switch to folder docker_mongodb_cluster and execute these commands to create our Docker images.

切换到 docker_mongodb_cluster 路径下并执行以下这些命令创建 docker 镜像：

```
sudo docker build \
  -t dev24/mongodb mongod

sudo docker build \
  -t dev24/mongodb mongos
```

##Create the Replica Sets

For our shard we need some replica sets. We will create two of them consisting of three nodes each. These commands will instantiate three mongod containers and assign them to replica set rs1. The parameters --noprealloc and --smallfiles are optional to avoid MongoDB is consuming lots of memory (useful while testing).

##创建副本集Replica Sets

集群中，每个分片都需要副本集以保证数据可靠性。在这里我们需要为每个分片创建两个副本以构成三个节点的副本集。这些命令将会实例化三个 mongod 容器并且将他们组成副本集 rs1 。参数 --noprealloc 和 --smallfiles 是可选项，用来避免 mongodb 消耗掉过多内存(在测试时十分有效)。

```
sudo docker run \
  -P -name rs1_srv1 \
  -d dev24/mongodb \
  --replSet rs1 \
  --noprealloc --smallfiles

sudo docker run \
  -P -name rs1_srv2 \
  -d dev24/mongodb \
  --replSet rs1 \
  --noprealloc --smallfiles

sudo docker run \
  -P -name rs1_srv3 \
  -d dev24/mongodb \
  --replSet rs1 \
  --noprealloc --smallfiles
```

For the second replica set we issue the same commands just using a different replica set name rs2.

将副本集的名称替换为 rs2 之后，我们就能够用同样的命令创建第二个副本集了。

```
sudo docker run \
  -P -name rs2_srv1 \
  -d dev24/mongodb \
  --replSet rs2 \
  --noprealloc --smallfiles

sudo docker run \
  -P -name rs2_srv2 \
  -d dev24/mongodb \
  --replSet rs2 \
  --noprealloc --smallfiles

sudo docker run \
  -P -name rs2_srv3 \
  -d dev24/mongodb \
  --replSet rs2 \
  --noprealloc --smallfiles
```

##Initialize the Replica Sets

First take a note of the IP address and port bindings of all Docker containers.

##初始化副本集

首先请记一下每个 docker 容器的 ip 地址以及绑定端口。

```
sudo docker inspect rs1_srv1
sudo docker inspect rs1_srv2
...
```

###Initialize Replica Set 1

Connect to MongoDB running in container rs1_srv1 (you need the local port bound for 27017/tcp figured out in the previous step).

###初始化副本集1

连接到运行于容器 rs1_srv1 的 mongodb (你需要将本地端口绑定到 27017/tcp ，如有疑问请查看前述步骤)。

```
mongo --port <port>

# MongoDB shell

rs.initiate()
rs.add("<IP_of_rs1_srv2>:27017")
rs.add("<IP_of_rs1_srv3>:27017")
rs.status()
```

Docker will assign an auto-generated hostname for containers and by default MongoDB will use this hostname while initializing the replica set. As we need to access the nodes of the replica set from outside the container we will use IP adresses instead.

Use these MongoDB shell commands to change the hostname to the IP address.

docker 将会自动为一个容器生成主机名，并且默认情况下 mongodb 将会在初始化副本集的时候使用这个主机名。所以我们需要使用 ip 地址来从容器外部登录到副本集的节点。

用以下的 mongodb shell 命令修改对应的主机名。

```
# MongoDB shell

cfg = rs.conf()
cfg.members[0].host = "<IP_of_rs1_srv1>:27017"
rs.reconfig(cfg)
rs.status()
```

###Initialize Replica Set 2

The exact same procedure is required for the second replica set. Connect to MongoDB running in container rs2_srv1.

###初始化副本集2

初始化副本集2也需要相同的步骤。请连上运行于 rs2_srv1 容器的 mongodb 。

```
mongo --port <port>

# MongoDB shell

rs.initiate()
rs.add("<IP_of_rs2_srv2>:27017")
rs.add("<IP_of_rs2_srv3>:27017")
rs.status()
```

Use these MongoDB shell commands to change the hostname to the IP address.

使用 mongodb shell 命令修改对应的主机名。

```
# MongoDB shell

cfg = rs.conf()
cfg.members[0].host = "<IP_of_rs2_srv1>:27017"
rs.reconfig(cfg)
rs.status()
```

##Create some Config Servers

Now we create three MongoDB config servers to manage our shard. For development one config server would be sufficient but this shows how easy it is to start more.

##创建Config Server

当副本集配置好之后，我们将创建三个 mongodb config 服务来管理我们的分片。对于开发来说，一个 config 服务已经足够了，不过这里我们将演示如何创建更多。

```
sudo docker run \
  -P -name cfg1 \
  -d dev24/mongodb \
  --noprealloc --smallfiles \
  --configsvr \
  --dbpath /data/db \
  --port 27017

sudo docker run \
  -P -name cfg2 \
  -d dev24/mongodb \
  --noprealloc --smallfiles \
  --configsvr \
  --dbpath /data/db \
  --port 27017

sudo docker run \
  -P -name cfg3 \
  -d dev24/mongodb \
  --noprealloc --smallfiles \
  --configsvr \
  --dbpath /data/db \
  --port 27017
```

Use docker inspect to figure out the IP adresses of each config server. We will need this to start the MongoDB router in the next step.

使用 docker inspect 命令查看每个 config 服务器的 ip 地址。我们将在下一个配置 mongodb router 的步骤中使用到它们。

##Create a Router

A MongoDB router is the point of contact for all the clients of the shard. The --configdb parameter takes a comma separated list of IP addresses and ports of the config servers.

Note that we are using the mongos image created in the first part.

##创建Router

mongodb router 是连接所有 shard 客户端的节点。--configdb 参数是一个用逗号分隔的 ip:port 列表，对应于 config 服务器。

注意到在这里我们用到了第一部分创建的 mongos 镜像。

```
sudo docker run \
  -P -name mongos1 \
  -d dev24/mongos \
  --port 27017 \
  --configdb \
    <IP_of_container_cfg1>:27017, \
    <IP_of_container_cfg2>:27017, \
    <IP_of_container_cfg3>:27017
```

##Initialize the Shard

Finally we need to initialize the shard. Connect to the MongoDB router (you need the local port bound for 27017/tcp) and execute these shell commands:

##初始化shard

最后，我们需要初始化 shard 。连接到 mongodb router (需要绑定本地端口 27017/tcp )并执行 shell 命令：

```
mongo --port <port>

# MongoDB shell

sh.addShard("rs1/<IP_of_rs1_srv1>:27017")
sh.addShard("rs2/<IP_of_rs2_srv1>:27017")
sh.status()
```

##Conclusion

It is really impressive how easy it is to create a fairly complex MongoDB setup using Docker. Nevertheless there are a couple of manual steps involved which ideally should be automated. Additionally wiring of the different moving parts using IP adresses should be replaced with a DNS based approach.

##小结

利用 docker 构建复杂 mongodb 的过程的确十分简单，让人印象深刻。虽然有很多需要设置的步骤，但是这很容易就能写自动化工具来完成。还有一点，就是对于不同部分使用 IP 地址的方式应该被使用 DNS 这样的方案来代替。

