# 如何在"特殊"的网络环境下编译 Docker

---

##### 作者：马全一

---

> 本文主要解决如何在“**特殊**”的网络环境下编译 Docker 程序，至于造成网络环境特殊的原因及解决办法请读者自行搜索。

---

笔者在 [2014 Container 技术大会](http://con2.csdn.net) 讲述了如何构建 Docker 开发环境，本文是对演讲第一部分 **如何编译 Docker** 的总结，希望为国内从事 Docker 研究的开发者提供一个解决编译的办法。其中要感谢 **[无闻](http://www.weibo.com/Obahua)** 同学开发的  **[gopm](http://gopm.io)** 项目，为解决“特殊”网络环境下载 Golang 的 Library 提供了方案，同时他也是本文编译方法3的主要贡献者。

由于 Docker 编译需要依赖于 Docker Daemon ，所以只能在 64 位的 Linux 环境下先安装 Docker 程序，再从 Github 上克隆 Docker 的代码进行编译。

在 Docker 的目录下执行 `make` 命令将默认执行 `Makefile` 中 `make binary` 指令进行编译。

```
default: binary

all: build
	$(DOCKER_RUN_DOCKER) hack/make.sh

binary: build
	$(DOCKER_RUN_DOCKER) hack/make.sh binary

cross: build
	$(DOCKER_RUN_DOCKER) hack/make.sh binary cross
```

从以上的 `Makefile` 可以看出，执行 `make`、`make binary`、`make all` 或 `make cross` 都可以得到可运行的 Docker 程序。


> ##### 在 Mac OS 环境下使用 brew 的命令安装 Docker ，只能得到一个 docker client 的二进制程序，如果以 daemon 的方式运行，会得到 ‘This is a client-only binary - running the Docker daemon is not supported.’ 的错误提示信息。

### 方法 1.

使用 VirtualBox 或者 VMWare Workstation 安装一个 Linux 的虚拟机。宿主机使用 VPN 等方案使网络“正常”访问各种“服务”，虚拟机网卡使用 NAT 模式。在 Linux 虚拟机内使用 make 进行编译 Docker 不会有任何网络问题。只是编译速度受限于 VPN 等网络解决方案，有可能等待时间很长。

### 方法 2.

Docker 每次发布新版本，都会在 [docker-dev](https://docker.cn/docker/docker-dev) 的镜像仓库发布一个新的标签，这个镜像仓库包含了编译 Docker 镜像所依赖的所有环境，只需替换 Docker 代码目录下的 `Dockerfile` 即可实现编译 Docker 。

```
FROM docker.cn/docker/docker-dev:v1.2.0
VOLUME /var/lib/docker
WORKDIR /go/src/github.com/docker/docker
ENV DOCKER_BUILDTAGS apparmor selinux
ENTRYPOINT [“hack/dind”]
COPY . /go/src/github.com/docker/docker
```

`Dockerfile` 中只保留必要的步骤就可以实现编译了。

### 方法 3.

对 Docker 代码中的 Docker 进行彻底的改造，用国内的各种镜像替换其中不能在“正常”网络条件下访问的镜像，使得代码能够快速编译通过。

```
FROM docker.cn/docker/ubuntu:14.04.1
MAINTAINER Meaglith Ma <genedna@gmail.com> (@genedna)

RUN echo "deb http://mirrors.aliyun.com/ubuntu trusty main universe" > /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty universe" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty universe" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-updates universe" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates universe" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted" >> /etc/apt/sources.list && echo "deb http://mirrors.aliyun.com/ubuntu/ trusty-security universe" >> /etc/apt/sources.list && echo "deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security universe" >> /etc/apt/sources.list

RUN	apt-get update && apt-get install -y \
	aufs-tools \
	automake \
	btrfs-tools \
	build-essential \
	curl \
	dpkg-sig \
	git \
	iptables \
	libapparmor-dev \
	libcap-dev \
	libsqlite3-dev \
	lxc=1.0* \
	mercurial \
	parallel \
	reprepro \
	ruby1.9.1 \
	ruby1.9.1-dev \
	s3cmd=1.1.0* \
	unzip \
	--no-install-recommends

RUN	git clone --no-checkout https://coding.net/genedna/lvm2.git /usr/local/lvm2 && cd /usr/local/lvm2 && git checkout -q v2_02_103

RUN	cd /usr/local/lvm2 && ./configure --enable-static_link && make device-mapper && make install_device-mapper

RUN	curl -sSL http://docker-cn.qiniudn.com/go1.3.1.src.tar.gz | tar -v -C /usr/local -xz
ENV	PATH	/usr/local/go/bin:$PATH
ENV	GOPATH	/go:/go/src/github.com/docker/docker/vendor
ENV PATH /go/bin:$PATH
RUN	cd /usr/local/go/src && ./make.bash --no-clean 2>&1

ENV	DOCKER_CROSSPLATFORMS	\
	linux/386 linux/arm \
	darwin/amd64 darwin/386 \
	freebsd/amd64 freebsd/386 freebsd/arm
ENV	GOARM	5
RUN	cd /usr/local/go/src && bash -xc 'for platform in $DOCKER_CROSSPLATFORMS; do GOOS=${platform%/*} GOARCH=${platform##*/} ./make.bash --no-clean 2>&1; done'

RUN	mkdir -p /go/src/github.com/gpmgo \
	&& cd /go/src/github.com/gpmgo \
	&& curl -o gopm.zip http://gopm.io/api/v1/download?pkgname=github.com/gpmgo/gopm\&revision=dev --location \
	&& unzip gopm.zip \
	&& mv $(ls | grep "gopm-") gopm \
	&& rm gopm.zip \
	&& cd gopm \
	&& go install

RUN	gopm bin -v code.google.com/p/go.tools/cmd/cover

RUN gem sources --remove https://rubygems.org/ \
  && gem sources -a https://ruby.taobao.org/ \
  && gem install --no-rdoc --no-ri fpm --version 1.0.2

RUN	gopm bin -v -d /go/bin github.com/cpuguy83/go-md2man@tag:v1

RUN	git clone -b buildroot-2014.02 https://github.com/jpetazzo/docker-busybox.git /docker-busybox

RUN	/bin/echo -e '[default]\naccess_key=$AWS_ACCESS_KEY\nsecret_key=$AWS_SECRET_KEY' > /.s3cfg

RUN	git config --global user.email 'docker-dummy@example.com'

RUN groupadd -r docker
RUN useradd --create-home --gid docker unprivilegeduser

VOLUME	/var/lib/docker
WORKDIR	/go/src/github.com/docker/docker
ENV	DOCKER_BUILDTAGS	apparmor selinux

ENTRYPOINT	["hack/dind"]

COPY	.	/go/src/github.com/docker/docker
```

以上的命令把 `Ubuntu` 镜像中的源替换为国内速度较快的阿里源；把 lvm2 镜像到国内的 Git 托管服务 [coding.net](https://coding.net) ；从 [七牛云存储](http://www.qiniu.com) 保存的 Golang 源码进行获取和编译；使用 [gopm](http://gopm.io) 下载编译所需要的 Library ；最后把其中 gem 源切换到淘宝。至此，可以在“特殊”的网络条件下快速编译 Docker 。

> ##### 曾经尝试使用 [golang](https://docker.cn/docker/golang) 的镜像仓库进行编译，但它是一个以 Debian 为基础的镜像仓库，所以在尝试安装第三方依软件的时候很多无法找到。

---

浏览/下载 [**如何编译 Docker**](http://resource.docker.cn/develop-docker-container-conference.pptx) 

---

**作者**

马全一，docker.cn 创始人和 Docker 中文社区发起人。资深系统架构师、SAP FI/CO 顾问和 ITSM & BMC Remedy 实施顾问，现专注于 Docker 的研发，维护开源 Docker  镜像仓库项目 [Docker  Bucket ](https://github.com/dockercn/docker-bucket)。新浪微博：[马全一](https://weibo.com/genedna)、Twitter：[meaglith](https://twitter.com/genedna)、Email：meaglith AT docker.cn
