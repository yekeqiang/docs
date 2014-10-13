# How to Deploy Douban CODE in Ubuntu and Docker

##### Author：[Zhen JU](http://weibo.com/1288360177)


[CODE](http://douban-code.github.io/) , standing for <b>C</b>ommunity <b>O</b>riginal <b>D</b>eveloper <b>E</b>ldamaris, initially was an internal project to build a cooperation platform based on Git version control system in [Douban](http://douban.com). Up to now, CODE has hosted almost all internal code in Douban. Feb 14, 2014 Douban Inc. offically announced to open-source the project. You can visit [Github](http://douban-code.github.io/) to get more information.


## Deploy CODE in Ubuntu

### Dependancies CODE requires

- python 2.7 or higher (this is included in Ubuntu)

- pip 1.4.1 or higher
```
	$ apt-get install python-pip
```
- libememcached patched by Douban

There are two approaches to install. You can download the original libmemcached and Douban patch, then path the files manually. Or you just download the libmemcached package Douban already patched. As to the first approach, there is an error on the patch path(probably it's caused by libmemcached's updates). I recommend to use the patched package directly.




Firstly, let's install build-essential and g++ compiler:

```
$ sudo apt-get install build-essential g++
```


Then download and unzip libmemcached package, make and make install:
    
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



> You can find specific instructions [here](http://douban-code.github.io/pages/python-libmemcached.html).

> Notice: You must complete the last step of "load config" operation to make libmemcached take effect. You can specify the number of CUP cores to do the "make" with -j, gengerally it should be one more than the actual core number. For example, your CPU is 4-core, then make it `make -j5` .



### Download CODE deployment scripts



Although CODE provides scripts, there are some problems during execution. I reorganized the workflow based on my use. I use Ubuntu for the demo.



You can find the scripts [here](https://github.com/douban/code/tree/master/scripts). Each system has a corresponding script, currently CODE supports archlinux, centos, fedora, gentoo, openuse and ubuntu.
    

    
common.sh script provides some general fuctions such as installing mysql and libmemcached, if you have them installed, you can just skip it.



OS_NAME.sh call the package installation command(such as 'apt-get' in Ubuntu) to:


Install basic develop environment:
    
```
$ sudo apt-get install build-essential g++ git python-pip python-virtualenv python-dev memcached -yq
```


Install mysql and the related dev libaries:
    
```
$ sudo apt-get install mysql-client mysql-server libmysqlclient-dev -yq
```


Set memecached port to 11311 and restart memcachaed:
    
```
$ sudo sed -i "s/11211/11311/g" /etc/memcached.conf
$ sudo /etc/init.d/memcached restart
```
    
Install libememcached(just as described above).
    
Install douban/CODE:
    
```
#clone CODE项目，并进入其目录：
#clone CODE repo and enter the directory:
$ git clone https://github.com/douban/code.git
$ cd code
    
#mysql创建valentine数据库（如果设置了密码，需要加上-p参数，然后输入密码）：
#Create valentine database in MySQL (if you have setup a password while installing MySQL, please add the -p parameter, then input your password):
$ mysql -uroot -e 'drop database if exists valentine;' 
$ mysql -uroot -e 'create database valentine;'

#从CODE项目的sql语句导入表设计（注意当前在code目录下）
# Import the table from CODE SQL file(Notice: we are now in the CODE directory)
$ mysql -uroot -D valentine < vilya/databases/schema.sql  
```	
    
Install virtualenv with pip:
    
```
$ sudo pip install virtualenv
```

Create and activate Python virtual environment:

```
# 激活后，命令行的前面会加上(venv)
# After activation, (venv) will be added to $PS1
$ virtualenv venv
$ . venv/bin/activate
```

Install cython and setuptools with pip:
    

```
(venv)$ pip install cython
(venv)$ pip install -U setuptools
```

If you are using archlinx, install MySQL-python patch:
    
```(venv)$ pip install "distribute==0.6.29" ```
    
Install the packages specified in requirements.txt in CODE:
    
```(venv)$ pip install -r requirements.txt```
    
Config the IP and port

    
```    
#把模板复制到vilya/local_config.py，CODE将从vilya/local_config.py文件中读取配置:
＃Copy the template config file to viliya/local_config.py, where CODE will read the configs from:
（venv)$ cp vilya/local_config.py.tmpl vilya/local_config.py
#打开配置文件，进行设置
#Open the config file and modify the configs
（venv)$ vim vilya/local_config.py
```


After openning vilya/local_config.py, you can see there are paremeters including domain, port, MySQL config, etc. If MySQL root user require password, add it on line 42:
    
```
"master": "localhost:3306:valentine:root:YOUR_MYSQL_ROOT_PASSWORD"
```
    


At last, launch CODE project (Notice: please replace 127.0.0.1 with a real IP):
    
```
（venv)$ gunicorn -w 2 -b 127.0.0.1:8000 app:app
```
    


Ok, now we have CODE deployed! Open the browser and type in the url:
    
```
http://YOUR_IP:8000
```


Then here we go!

![alt](http://resource.docker.cn/douban-code-interface.jpeg)    



Register an account, create a git repository, go look around and Have FUN!
    


When using pip to install python libs, it might be quite slow to visit the official mirror in domestic China, then you can use douban's mirror (it's fast and solid with a full collection of all packages). Open `~/.pip/pip/conf` file (it probably dose not exist, you can edit and save it), then input:
    
```    
[global]                                  
index-url = http://pypi.douban.com/simple/
```
    


Well now, download python library from douban mirror. Speedy!
     


## Deploy CODE in Docker



It's quite easy to deploy CODE in Docker. Following Docker's design philosophy, we seperate data and logic, create three different mirrors for MySQL, memcached and CODE, and communicate with each other with Docker linkage. It just looks like that you deploy them to three machines, but faster and easier. 



Now we look up the Dockerfiles, firstly MySQL:

```
FROM   stackbrew/ubuntu:saucy

RUN    apt-get install -y --force-yes software-properties-common
# 由于Docker的ubuntu镜像apt源不完善，我们要加入一个
# Because Docker ubuntu mirror' apt source is not adquate, we have to add the official source to the list file
RUN    add-apt-repository -y "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) universe"
RUN    apt-get --yes --force-yes update
RUN    apt-get --yes --force-yes upgrade
# 通过ENV指令，设定环境变量。将MySQL的root密码保存到变量$MYSQL_PASSWORD中
# Set up environment variable via ENV command. Save MySQL root password to $MYSQL_PASSWORD

ENV    MYSQL_PASSWORD docker

# 安装MySQL过程中，要输入2次root密码
# Input the root password twice during the installation of MySQL
RUN    echo "mysql-server mysql-server/root_password password $MYSQL_PASSWORD" | debconf-set-selections
RUN    echo "mysql-server mysql-server/root_password_again password $MYSQL_PASSWORD" | debconf-set-selections

# 安装mysql-server和git
# Install mysql-server and git
RUN    apt-get -y --force-yes install mysql-server git
RUN    apt-get autoclean
RUN    sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/my.cnf

# clone豆瓣CODE项目，因为我们要使用schema.sql来创建CODE所需要的valentine数据库
# Clone CODE repo, we will use schema.sql to create 'valentine' database
RUN    sudo git clone https://github.com/douban/code.git

# 将数据库的创建写到import.sh脚本中，并添加执行权限。由于每次RUN都会commit一个新的镜像，AUFS的状态发生变化，因此3条导入操作必须放在一起执行。
# Put the 'creation of database' into import.sh script, and add execute permission. Since every RUN will commit a new mirror, and lead to a status change of AUFS, therefore the three import operations must be executed in the same run.
RUN echo "mysql -uroot -p$MYSQL_PASSWORD -e 'drop database if exists valentine;'" >> import.sh
RUN echo "mysql -uroot -p$MYSQL_PASSWORD -e 'create database valentine;'" >> import.sh
RUN echo "mysql -uroot -p$MYSQL_PASSWORD -D valentine < /code/vilya/databases/schema.sql" >> import.sh
RUN chmod +x import.sh

# 为root用户设定权限，然后执行导入数据库操作
# Set up permissions for root user, and import database
RUN    /usr/sbin/mysqld & \
 sleep 10s &&\
 echo "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '$MYSQL_PASSWORD' WITH GRANT OPTION; FLUSH PRIVILEGES" | mysql -p$MYSQL_PASSWORD &&\
 ./import.sh

# EXPOSE命令指定对外暴露的接口，这一条非常重要，通过EXPOSE命令，可以把MySQL镜像中数据库的地址和端口开放给CODE镜像，然后通过Docker的link功能来获取地址和端口
EXPOSE 3306
# EXPOSE command specifies external interfaces. This is very important. By EXPOSE command, database's address and port in MySQL is exposed to CODE mirror. Then we can use Docker link to obtain address and port.

# CMD命令指定当镜像运行时，启动mysqld_safe
CMD ["/usr/bin/mysqld_safe", "--skip-syslog", "--log-error=/dev/null"]
# Start mysqld_safe when when run the mirror.
```



That's all of the Dockerfile of MySQL mirror, only 20 lines. Now let's check memcached Dokcerfile:

```
FROM ubuntu
# 添加apt源
# Add apt source
RUN sudo echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

# 编译安装豆瓣patch的memcached需要build-essential库和g++编译器，Docker的ubuntu镜像自身没有安装wget，我们这还要安装一下
# Install build-essential library, g++ compiler and wget(since the Ubuntu docker image doesn't have wget installed)
RUN sudo apt-get update
RUN sudo apt-get install -y build-essential g++ memcached wget

# 编译安装豆瓣patch的memcached
# Compile and install douban patched memcached
RUN wget --no-check-certificate https://github.com/xtao/douban-patched/raw/master/libmemcached-douban-1.0.18.tar.gz

RUN tar zxf libmemcached-douban-1.0.18.tar.gz
RUN cd libmemcached-1.0.18 && ./configure && make && sudo make install
RUN rm -rf libmemcached*
RUN sudo ldconfig


# 通过sed替换memcached配置，把memcached端口设定到11311
# Replace memcached config, set the port to 11311
RUN sudo sed -i "s/11211/11311/g" /etc/memcached.conf
RUN sudo /etc/init.d/memcached restart

# 对外暴露11311端口
# Expose port 11311
EXPOSE 11311

# 启动镜像时，启动memcached
# Start memcached when run the mirror
CMD memcached -u daemon
```



It's as concise as the Ubuntu deployment commands! Now it's the Dockerfile of CODE mirror:

```
FROM ubuntu

RUN sudo echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

# 安装必要软件。这里file是必须装的，因为Docker镜像默认没有安装file，会导致执行gunicorn的时候找不到Python的magic库
# Install the necessary softwares. The file package is necessary, lack of file package will lead to an error that Python magic library could not be found when execute gunicorn. 
RUN apt-get install -yq build-essential g++ wget libmysqlclient-dev git python-pip python-dev python-virtualenv file
RUN pip install virtualenv python-memcached

# 因为CODE需要调用豆瓣patch后的memcached，所以这里也要安装豆瓣patch的memcached
# Install douban patched memcached
RUN wget --no-check-certificate https://github.com/xtao/douban-patched/raw/master/libmemcached-douban-1.0.18.tar.gz

RUN tar zxf libmemcached-douban-1.0.18.tar.gz
RUN cd libmemcached-1.0.18 && ./configure && make && sudo make install
RUN rm -rf libmemcached*
RUN sudo ldconfig

# clone豆瓣CODE项目
# Clone douban CODE repo
RUN git clone https://github.com/douban/code.git

# 因为每次调用RUN的时候，都会产生一个新的layer，AUFS路径会发生改变，所以我们把安装Python库的工作都写到install.sh脚本中。为其添加执行权限并执行
# Since each RUN will produce a new layer, and the root path of AUFS will change. So we install python libs with install.sh script. Here we add the script to the image, add execution permission to it and run it
# ADD命令把./intall.sh拷贝到Docker容器的/目录下
# ADD command copy ./install.sh to Docker container directory
ADD ./install.sh /
RUN chmod +x install.sh
RUN /install.sh

# 和install.sh相同，我们把启动CODE的工作也写到脚本中，并为其添加执行权限
# And the same for the start.sh script
ADD ./start.sh /
RUN chmod +x start.sh

# 当启动时，执行start脚本
# Execute start script when the image runs
CMD /start.sh
```


The install.sh script in the host machine:

```
# 进入code目录并安装必要的python库
# Enter CODE directory and install essential python library
cd code
pip install cython
pip install -U setuptools
pip install -r requirements.txt
```


And the start.sh script:
```
cd code
cp vilya/local_config.py.tmpl vilya/local_config.py
# 使用link时，在环境变量中可以拿到link的MySQL镜像和memcached镜像中数据库的地址和端口。开头的DB和MEM是指定运行MySQL镜像和Memcached镜像时指定的别名（alias）
# When using linkage in Docker, we can access the address and port of the MySQL and memcached image through the environment variables, which start with 'DB' and 'MEM'(They are the alias name we specified when running MySQL and memcached image).
sed -i "s/localhost/$DB_PORT_3306_TCP_ADDR/g" vilya/local_config.py
sed -i "s/root:/root:docker/g" vilya/local_config.py #docker是MySQL的密码，根据需要替换成自己的密码
sed -i "s/127.0.0.1:11311/$MEM_PORT_11311_TCP_ADDR:11311/g" vilya/local_config.py

# 我们还需要获取当前Docker镜像的ip，来替换原来的
# We specify the current ip when running gunicorn, notice that the eth0 might be different in your machine
CURRENT_IP=$(ifconfig eth0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}')

virtualenv venv
. venv/bin/activate

gunicorn -w 2 -b $CURRENT_IP:8000 app:app
```


The start.sh script starts virtual environment and run CODE.



Now we have three direcotries which are douban-mysql, douban-memcached and douban-code. We put the three mirror Dockerfiles into corresponding directory, put install.sh and start.sh to douban-code directory, then construct the docker images:

```
# 首先来构建MySQL：
# Construct MySQL image:
cd douban-mysql
# -t参数为构建的镜像打上名称为douban/mysql的标签，便于我们访问
# -t parameter add the tag 'douban/mysql' for the MySQL image
docker build -t douban/mysql .

# memcached的构建：
# Construct memcached image:
cd douban-memcached
docker build -t douban/memcached .

# memcached的构建，估计你都已经猜到了，还是照猫画虎：
# continue constructing memcached image:
cd douban-code
docker build -t douban/code .
```



Be patient, it taks some time while compiling memcached. After all the three images are successfully constructed, let's have a check:

```
docker images
```
输出结果：

Output result:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
douban/memcached    latest              442f1e0a8ff2        12 hours ago        677.1 MB
douban/mysql        latest              0874559a4b68        12 hours ago        508.4 MB
douban/code         latest              6cd79d42bf19        14 hours ago        418.2 MB
helloworld          latest              861397c62c8a        2 weeks ago         204.4 MB
ubuntu              13.10               9f676bd305a4        7 weeks ago         178 MB
......
```


Now we have three mirrors, and it's time to run CODE. 

Start MySQL and memcached:

```
# -d参数让镜像以daemon模式（或者叫detached模式）来执行。因为mysql和memcached都是daemon模式运行的，所以不带-d参数，会一直挂起。
# -d parameter lets images to run in daemon mode. If -d parameter is missed, mysql and memecached will be hung up.
# 镜像运行时候，我们用-name参数为容器指定一个便于访问的名称，这里就用mysql和memcached
# Use -name parameter to specify container name. In this case we use mysql and memecached
docker run -d -name mysql douban/mysql
docker run -d -name memcached douban/memcached
```

：

Finally it's CODE mirror:

```
# 魔法就在-link参数上。
# The key point is the -link parameter.

# link参数的格式是：name:alias。以mysql为例：mysql是我们为MySQL镜像指定的名称，db是别名，这样，在CODE镜像访问MySQL时，环境变量中就有DB开头的MySQL路径和端口。请参考CODE镜像的start.sh脚本，其中对MySQL路径的替换，就是用$DB_PORT_3306_TCP_ADDR变量。
# The format of -link parameter is "name:alias". For exmaple the "mysql:db", mysql is the container name we specified in the previous step, db is the alias, so when CODE image communicates with MySQL image, the address and port will appear in CODE image's environment variables, staring with 'DB_'.
# -p参数对Host和Docker容器进行端口的重定向，也就是说，把Host的8812端口定向到CODE镜像的8000端口，这样，外部就可以通过8812端口访问CODE啦。
# -p parameter redirect Host port 8812 to Docker container port 8000, so we can access the CODE image by HOST port 8812.
docker run -link mysql:db -link memcached:mem -p 8812:8000 douban/code
```


We don't specify -d parameter here because we need to check the result of CODE run. When CODE image runs, the start.sh script will create virtual environment and execute gunicor. When we see the output, open browser and input: 

```
http://HOST_IP:8812
```



Now, it's BACK! Enjoy!

![alt](http://resource.docker.cn/douban-code-interface-firefox.jpeg)
