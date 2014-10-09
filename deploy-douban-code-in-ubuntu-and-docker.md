#在 Ubuntu 和 Docker 下部署豆瓣 CODE

####作者：[巨震](http://weibo.com/1288360177)

***
##在 Ubuntu 下部署豆瓣 CODE


###豆瓣 CODE 需要的依赖

 - python 2.7 或更高

    Ubuntu 自带

 - pip1.4.1 或更高

```
$ apt-get install python-pip
```

 - 豆瓣打过 patch 的 libmemcached

有两种方式来安装，一种是下载原版 libmemcached 和豆瓣的 patch ，手动把 patch 打上去。另一种是下载豆瓣打好的 libmemcached 包。第一种方式，豆瓣给出的 patch 指定的路径存在问题（可能是 libmemcached 有更新，路径有所改变）。所以直接使用打好 patch 的包。

首先，我们来安装必要的库和g++编译器：

```
$ sudo apt-get install build-essential g++
```

然后下载、解压libmemcached包，编译安装：
    
```
$ wget  https://github.com/xtao/douban-patched/raw/master/libmemcached-douban-1.0.18.tar.gz
$ tar zxf libmemcached-douban-1.0.18.tar.gz
$ cd libmemcached-1.0.18
$ ./configure && make && sudo make install
$ cd ..
$ rm -rf python-libmemcached
$ rm -rf libmemcached*
$ sudo ldconfig
```

以上参考：http://douban-code.github.io/pages/python-libmemcached.html 。请注意，最后一步 load config 的操作是必须的。 make 的时候，可以指定调用 CPU 内核的个数，一般比 CPU 实际内核数多1，例如，你的CPU是4核的，就指定： `make -j5`

###下载执行豆瓣 CODE 的部署脚本

豆瓣 CODE 的 git 主页上有相关的说明，但是提供的脚本在执行过程中还存在一些问题，下面以 Ubuntu 为例，把脚本的工作整理一下：

脚本文件可以在 https://github.com/douban/code/tree/master/scripts 拿到，每个系统都有对应的脚本，目前支持archlinux，centos，fedora，gentoo，opensuse，ubuntu。
    
common.sh 脚本提供了一些通用的函数（如安装mysql、libmemcached ，如果已经安装好，则脚本会跳过这些软件的安装）。
    
OS_NAME.sh 根据每个系统的不同，调用了对应的安装命令和系统默认路径。基本过程是：

    
安装基本的开发环境：
    
```
$ sudo apt-get install build-essential g++ git python-pip python-virtualenv python-dev memcached -yq
```
    
安装 mysql 和相关的库：
    
```
$ sudo apt-get install mysql-client mysql-server libmysqlclient-dev -yq
```

设置 memcached 端口为 11311 并重启 memcached：
    
```
$ sudo sed -i "s/11211/11311/g" /etc/memcached.conf
$ sudo /etc/init.d/memcached restart
```
    
安装 libmemcached（即前面安装豆瓣打包过的 libmemcached ，这里不再赘述）
    
安装 douban/CODE ：
    
```
#clone CODE项目，并进入其目录：
$ git clone https://github.com/douban/code.git
$ cd code
    
#mysql创建valentine数据库（如果设置了密码，需要加上-p参数，然后输入密码）：
$ mysql -uroot -e 'drop database if exists valentine;' #先删除valentine数据库
$ mysql -uroot -e 'create database valentine;'	        #重新创建valentine数据库
$ mysql -uroot -D valentine < vilya/databases/schema.sql  #从CODE项目的sql语句导入表设计（注意当前在code目录下）
```	
    
用 pip 安装 virtualenv :
    
```
$ sudo pip install virtualenv
```

创建并激活 Python 虚拟环境：

```
# 激活后，命令行的前面会加上（venv）
$ virtualenv venv
$ . venv/bin/activate
```
    
pip 安装 cython 和 setuptools ：
    
```
(venv)$ pip install cython
(venv)$ pip install -U setuptools
```

（如果系统是 archlinux ）安装 MySQL-python 的补丁:
    
```
（venv)$ pip install "distribute==0.6.29" 
```
    
安装 CODE 项目中，requirements.txt 中指定的包：
    
```
（venv)$ pip install -r requirements.txt
```
    
对 IP 、端口进行一些配置：
    
```    
#把模板复制到vilya/local_config.py，CODE将从vilya/local_config.py文件中读取配置:
（venv)$ cp vilya/local_config.py.tmpl vilya/local_config.py
#打开配置文件，进行设置
（venv)$ vim vilya/local_config.py
```
    
打开 vilya/local_config.py 后，可以看到里面的参数，包括 domain 、端口、 MySQL 的配置等等。如果 MySQL的root 用户需要密码访问，请在 42 行加上密码：
    
```
"master": "localhost:3306:valentine:root:YOUR_MYSQL_ROOT_PASSWORD"
```
    
最后，启动 CODE 项目（注意：把下面的 127.0.0.1 换成真实的 IP ）：
    
```
（venv)$ gunicorn -w 2 -b 127.0.0.1:8000 app:app
```
    
OK ，到这里 CODE 就部署完成了，打开浏览器，输入：
    
```
http://YOUR_IP:8000
```
    
就可以看到 CODE 的界面了：

![alt](http://resource.docker.cn/douban-code-interface.jpeg)    

进入注册一个帐号，创建一个 git 库玩玩，everything dependes on you ^_<
    
另外，用 pip 安装 python 包的时候，可能会遇到访问国外镜像速度太慢的问题，可以换用豆瓣的镜像（这里要赞一下，豆瓣的镜像访问速度非常快，而且各种包都很全，相比下，清华大学的镜像很多包都是没有的）。具体方法是打开  `~/.pip/pip.conf` 文件（可能不存在，编辑后保存就可以）， 输入如下内容：
    
```    
[global]                                  
index-url = http://pypi.douban.com/simple/
```
    
这样，就可以从豆瓣的镜像下载 python 的库了，速度杠杠滴啊，哈哈。
     
*** 
##在 Docker 中部署豆瓣 CODE

有了在 Ubuntu 下部署 CODE 经验，在 Docker 中部署 CODE 就变得非常容易了。根据 Docker 的设计哲学，我们把数据和逻辑分开，为 MySQL 、 memcached 和 CODE 创建3个不同的镜像，然后彼此进行通信，就如同将三者部署到三台服务器上，而且部署速度更快、更容易。

下面我们来看一下三个镜像的 Dockerfile ，先从 MySQL 开始：

```
FROM   stackbrew/ubuntu:saucy

RUN    apt-get install -y --force-yes software-properties-common
# 由于Docker的ubuntu镜像apt源不完善，我们要加入一个
RUN    add-apt-repository -y "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) universe"
RUN    apt-get --yes --force-yes update
RUN    apt-get --yes --force-yes upgrade
# 通过ENV指令，设定环境变量。将MySQL的root密码保存到变量$MYSQL_PASSWORD中
ENV    MYSQL_PASSWORD docker

# 安装MySQL过程中，要输入2次root密码
RUN    echo "mysql-server mysql-server/root_password password $MYSQL_PASSWORD" | debconf-set-selections
RUN    echo "mysql-server mysql-server/root_password_again password $MYSQL_PASSWORD" | debconf-set-selections

# 安装mysql-server和git
RUN    apt-get -y --force-yes install mysql-server git
RUN    apt-get autoclean
RUN    sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/my.cnf

# clone豆瓣CODE项目，因为我们要使用schema.sql来创建CODE所需要的valentine数据库
RUN    sudo git clone https://github.com/douban/code.git

# 将数据库的创建写到import.sh脚本中，并添加执行权限。由于每次RUN都会commit一个新的镜像，AUFS的状态发生变化，因此3条导入操作必须放在一起执行。
RUN echo "mysql -uroot -p$MYSQL_PASSWORD -e 'drop database if exists valentine;'" >> import.sh
RUN echo "mysql -uroot -p$MYSQL_PASSWORD -e 'create database valentine;'" >> import.sh
RUN echo "mysql -uroot -p$MYSQL_PASSWORD -D valentine < /code/vilya/databases/schema.sql" >> import.sh
RUN chmod +x import.sh

# 为root用户设定权限，然后执行导入数据库操作
RUN    /usr/sbin/mysqld & \
 sleep 10s &&\
 echo "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '$MYSQL_PASSWORD' WITH GRANT OPTION; FLUSH PRIVILEGES" | mysql -p$MYSQL_PASSWORD &&\
 ./import.sh

# EXPOSE命令指定对外暴露的接口，这一条非常重要，通过EXPOSE命令，可以把MySQL镜像中数据库的地址和端口开放给CODE镜像，然后通过Docker的link功能来获取地址和端口
EXPOSE 3306

# CMD命令指定当镜像运行时，启动mysqld_safe
CMD ["/usr/bin/mysqld_safe", "--skip-syslog", "--log-error=/dev/null"]
```

以上就是 MySQL 镜像的 Dockerfile ，只有短短的 20 行左右。下面我们再看 memcached 的 Dockerfile ：

```
FROM ubuntu
# 添加apt源
RUN sudo echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

# 编译安装豆瓣patch的memcached需要build-essential库和g++编译器，Docker的ubuntu镜像自身没有安装wget，我们这还要安装一下
RUN sudo apt-get update
RUN sudo apt-get install -y build-essential g++ memcached wget

# 编译安装豆瓣patch的memcached
RUN wget --no-check-certificate https://github.com/xtao/douban-patched/raw/master/libmemcached-douban-1.0.18.tar.gz

RUN tar zxf libmemcached-douban-1.0.18.tar.gz
RUN cd libmemcached-1.0.18 && ./configure && make && sudo make install
RUN rm -rf libmemcached*
RUN sudo ldconfig


# 通过sed替换memcached配置，把memcached端口设定到11311
RUN sudo sed -i "s/11211/11311/g" /etc/memcached.conf
RUN sudo /etc/init.d/memcached restart

# 对外暴露11311端口
EXPOSE 11311

# 启动镜像时，启动memcached
CMD memcached -u daemon
```

基本上就是 Ubuntu 下部署的命令，非常简单。最后我们再来看 CODE 镜像的 Dockerfile ：

```
FROM ubuntu

RUN sudo echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

# 安装必要软件。这里file是必须装的，因为Docker镜像默认没有安装file，会导致执行gunicorn的时候找不到Python的magic库
RUN apt-get install -yq build-essential g++ wget libmysqlclient-dev git python-pip python-dev python-virtualenv file
RUN pip install virtualenv python-memcached

# 因为CODE需要调用豆瓣patch后的memcached，所以这里也要安装豆瓣patch的memcached
RUN wget --no-check-certificate https://github.com/xtao/douban-patched/raw/master/libmemcached-douban-1.0.18.tar.gz

RUN tar zxf libmemcached-douban-1.0.18.tar.gz
RUN cd libmemcached-1.0.18 && ./configure && make && sudo make install
RUN rm -rf libmemcached*
RUN sudo ldconfig

# clone豆瓣CODE项目
RUN git clone https://github.com/douban/code.git

# 因为每次调用RUN的时候，都会产生一个新的layer，AUFS路径会发生改变，所以我们把安装Python库的工作都写到install.sh脚本中。为其添加执行权限并执行
# ADD命令把./intall.sh拷贝到Docker容器的/目录下
ADD ./install.sh /
RUN chmod +x install.sh
RUN /install.sh

# 和install.sh相同，我们把启动CODE的工作也写到脚本中，并为其添加执行权限
ADD ./start.sh /
RUN chmod +x start.sh

# 当启动时，执行start脚本
CMD /start.sh
```

要从 Host 复制到镜像的 install.sh 脚本：

```
# 进入code目录并安装必要的python库
cd code
pip install cython
pip install -U setuptools
pip install -r requirements.txt
```

以及需要复制的 start.sh 脚本：

```
cd code
cp vilya/local_config.py.tmpl vilya/local_config.py
# 使用link时，在环境变量中可以拿到link的MySQL镜像和memcached镜像中数据库的地址和端口。开头的DB和MEM是指定运行MySQL镜像和Memcached镜像时指定的别名（alias）
sed -i "s/localhost/$DB_PORT_3306_TCP_ADDR/g" vilya/local_config.py
sed -i "s/root:/root:docker/g" vilya/local_config.py #docker是MySQL的密码，根据需要替换成自己的密码
sed -i "s/127.0.0.1:11311/$MEM_PORT_11311_TCP_ADDR:11311/g" vilya/local_config.py

# 我们还需要获取当前Docker镜像的ip，来替换原来的127.0.0.1
CURRENT_IP=$(ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}')

virtualenv venv
. venv/bin/activate

gunicorn -w 2 -b $CURRENT_IP:8000 app:app
```

start.sh 脚本主要开启虚拟环境，并启动 CODE 。

OK ，万事俱备，只欠东风，我们准备好三个目录，分别是 douban-mysql 、 douban-memcached 和 douban-code 。把三个镜像的 Dockerfile 放在对应的目录中， install.sh 和 start.sh 放到 douban-code 目录中，然后进行镜像的构建：

```
# 首先来构建MySQL：
cd douban-mysql
# -t参数为构建的镜像打上名称为douban/mysql的标签，便于我们访问
docker build -t douban/mysql .

# memcached的构建：
cd douban-memcached
docker build -t douban/memcached .

# memcached的构建，估计你都已经猜到了，还是照猫画虎：
cd douban-code
docker build -t douban/code .
```

编译 memcached 的过程比较长，大家要耐心等待哦。当三个镜像都构建好时，我们来查一下镜像是否存在：

```
docker images
```

输出结果：

```
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
douban/memcached    latest              442f1e0a8ff2        12 hours ago        677.1 MB
douban/mysql        latest              0874559a4b68        12 hours ago        508.4 MB
douban/code         latest              6cd79d42bf19        14 hours ago        418.2 MB
helloworld          latest              861397c62c8a        2 weeks ago         204.4 MB
ubuntu              13.10               9f676bd305a4        7 weeks ago         178 MB
......
```

好了，我们已经拥有这三个镜像了，下面我们就把 CODE 跑起来。首先，启动 MySQL 和 memcached ：

```
# -d参数让镜像以daemon模式（或者叫detached模式）来执行。因为mysql和memcached都是daemon模式运行的，所以不带-d参数，会一直挂起。
# 镜像运行时候，我们用-name参数为容器指定一个便于访问的名称，这里就用mysql和memcached
docker run -d -name mysql douban/mysql
docker run -d -name memcached douban/memcached
```

最后，是 CODE 镜像：

```
# 魔法就在-link参数上。

# link参数的格式是：name:alias，以mysql为例。mysql是我们为MySQL镜像指定的名称，db是别名，这样，在CODE镜像访问MySQL时，环境变量中就有DB开头的MySQL路径和端口。请参考CODE镜像的start.sh脚本，其中对MySQL路径的替换，就是用$DB_PORT_3306_TCP_ADDR变量。
# -p参数对Host和Docker容器进行端口的重定向，也就是说，把Host的8812端口定向到CODE镜像的8000端口，这样，外部就可以通过8812端口访问CODE啦。
docker run -link mysql:db -link memcached:mem -p 8812:8000 douban/code
```

我们这里没有指定 -d 参数，因为我们需要查看 CODE 运行的结果。 CODE 镜像运行时，先创建虚拟环境，然后执行 gunicorn ，当我们看到输出结果时候，就可以打开浏览器，输入：

```
http://HOST_IP:8812
```

怎么样，熟悉的 CODE 界面是不是又回来了呢？

![alt](http://resource.docker.cn//douban-code-interface-firefox.jpeg)
