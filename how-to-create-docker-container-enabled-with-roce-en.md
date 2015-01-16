# HowTo Create Docker Container enabled with RoCE

---

##### Author: Mellanox

---

This post describes how to create and run a docker containers which supports RDMA, using CentOS 7.0 as the OS of the host and the container.
 
## References

- [https://www.docker.com/whatisdocker/](https://www.docker.com/whatisdocker/)
- [https://docs.docker.com/](https://docs.docker.com/)
- [Docker infiniband](http://www.slideshare.net/syoyo/docker-infiniband)
- [Docker InfiniBand - Qiita](http://qiita.com/syoyo/items/bea48de8d7c6d8c73435)
 
## Introduction

As of time being, performing RDMA from containers requires some amount of manual configuration.
Note: similar work have been done for InfiniBand devices, available at [Docker infiniband](http://www.slideshare.net/syoyo/docker-infiniband) and [Docker InfiniBand - Qiita](http://qiita.com/syoyo/items/bea48de8d7c6d8c73435)
 
## Setup

The setup assumes two hosts connected via a switch or any L2 domain. In this example, the subnet of 192.168.101.xxx is used.
 
## Host Configuration

Configure both hosts to be enabled with RoCE and the drivers shipped with CentOS 7.
Refer to this post [HowTo setup RoCE connection using Inbox driver (RHEL, Ubuntu)](http://community.mellanox.com/docs/DOC-1465)
 
Once there is RoCE connectivity between the hosts. we are ready to launch a container which will support RDMA operations.
For this, we will need either to make the container privileged, or install a git revision of docker, that supports exposing to the container specific devices.

To get , compile docker and lunch it on CentOS 7, follow the next steps:

### 1. Get the docker sources

```
# git clone https://github.com/docker/docker.git
```
 
### 2. Install the required dependencies for building docker

```
# yum install -y golang golang-github-godbus-dbus-devel golang-github-coreos-go-systemd-devel golang-github-gorilla-mux-devel golang-github-gorilla-context-devel  golang-googlecode-net-devel golang-googlecode-sqlite-devel golang-github-syndtr-gocapability-devel golang-github-kr-pty-devel
# yum install -y sqlite-devel device-mapper-devel device-mapper-event-devel glibc-static btrfs-progs-devel device-mapper-devel systemd-devel
```
 
### 3. Compile

```
AUTO_GOPATH=1 ./hack/make.sh dynbinary
```
 
### 4. Stop system docker service:

```
# service docker stop
```
 
### 5. Launch docker service:

```
# cd bundles/1.2.0-dev/dynbinary
# ./docker -d --selinux-enabled &
```
 
Using the docker you just compiled, we can launch a centos container, that is allowed to access the infiniband devices uverbs0 and the RDMA-CM. Due to RDMA-CM limitations, the container must use the host network name space. The following command line launches such container:

```
# ./docker run --net=host --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm  \
  -t -i centos /bin/bash
```
 
Now, we need to install the user space RDMA software in the container:

```
bash-4.2 # yum install -y libibverbs-utils libibverbs-devel libibverbs-devel-static libmlx4 \
libmlx5 ibutils libibcm libibcommon libibmad libibumad
bash-4.2 # yum install -y rdma  librdmacm-utils librdmacm-devel librdmacm libibumad-devel perftest
```
And finally, we can test RDMA traffic from the container, using the same commands as above (ib_send_bw).

---

Original source: [HowTo Create Docker Container enabled with RoCE](http://community.mellanox.com/docs/DOC-1506)