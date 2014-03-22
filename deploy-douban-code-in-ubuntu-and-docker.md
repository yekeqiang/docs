# Ubuntu下部署豆瓣CODE
---

###豆瓣CODE需要的依赖

 - python 2.7或更高

    Ubuntu自带

 - pip1.4.1或更高

        $ apt-get install python-pip

 - 豆瓣打过patch的libmemcached

    有两种方式来安装，一种是下载原版libmemcached和豆瓣的patch，手动把patch打上去。另一种是下载豆瓣打好的libmemcached包。第一种方式，豆瓣给出的patch指定的路径存在问题（可能是libmemcached有更新，路径有所改变）。所以直接使用打好patch的包。
    首先，我们来安装必要的库和g++编译器：
    
        $ sudo apt-get install build-essential g++
    
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
    以上参考：http://douban-code.github.io/pages/python-libmemcached.html
    请注意，最后一步load config的操作是必须的。make的时候，可以指定调用CPU内核的个数，一般比CPU实际内核数多1，例如，你的CPU是4核的，就指定： `make -j5`




###下载执行豆瓣CODE的部署脚本

- ` `

    豆瓣CODE的git主页上有相关的说明，但是提供的脚本在执行过程中还存在一些问题，下面以Ubuntu为例，把脚本的工作整理一下：
    
    脚本文件可以在 https://github.com/douban/code/tree/master/scripts 拿到，每个系统都有对应的脚本，目前支持archlinux，centos，fedora，gentoo，opensuse，ubuntu。
    
    common.sh脚本提供了一些通用的函数（如安装mysql、libmemcached，如果已经安装好，则脚本会跳过这些软件的安装）
    
    OS_NAME.sh根据每个系统的不同，调用了对应的安装命令和系统默认路径。基本过程是：

    
    安装基本的开发环境：
    
    ```
    $ sudo apt-get install build-essential g++ git python-pip python-virtualenv python-dev memcached -yq
    ```
    
    安装mysql和相关的库：
    
    ```
    $ sudo apt-get install mysql-client mysql-server libmysqlclient-dev -yq
    ```
    
    设置memcached端口为11311并重启memcached：
    
    ```
    $ sudo sed -i "s/11211/11311/g" /etc/memcached.conf
    $ sudo /etc/init.d/memcached restart
    ```
    
    安装libmemcached（即前面安装豆瓣打包过的libmemcached，这里不再赘述）
    
    安装douban/CODE：
    
    ```
    #clone CODE项目，并进入其目录：
    $ git clone https://github.com/douban/code.git
    $ cd code
    
    #mysql创建valentine数据库（如果设置了密码，需要加上-p参数，然后输入密码）：
    $ mysql -uroot -e 'drop database if exists valentine;'    #先删除valentine数据库
    $ mysql -uroot -e 'create database valentine;'	        #重新创建valentine数据库
    $ mysql -uroot -D valentine < vilya/databases/schema.sql  #从CODE项目的sql语句导入表设计（注意当前在code目录下）
    ```	
    
    用pip安装virtualenv:
    
    ```
    $ sudo pip install virtualenv
    ```
    
    激活virtualenv： 
    ```
    # 激活后，命令行的前面会加上（venv）
    $ . venv/bin/activate
    ```
    
    pip安装cython和setuptools：
    
    ```
    (venv)$ pip install cython
    (venv)$ pip install -U setuptools
    ```
    
    （如果系统是archlinux）安MySQL-python的补丁:
    
    ```
    （venv)$ pip install "distribute==0.6.29" 
    ```
    
    安装CODE项目中，requirements.txt中指定的包：
    
    ```
    （venv)$ pip install -r requirements.txt
    ```
    
    		
    对ip、端口进行一些配置：
    
    ```    
    #把模板复制到vilya/local_config.py，CODE将从vilya/local_config.py文件中读取配置:
    （venv)$ cp vilya/local_config.py.tmpl vilya/local_config.py
    #打开配置文件，进行设置
    （venv)$ vim vilya/local_config.py
    ```
    
    打开vilya/local_config.py后，可以看到里面的参数，包括domain、端口、MySQL的配置等等。如果MySQL的root用户需要密码访问，请在42行加上密码：
    
    ```
        "master": "localhost:3306:valentine:root:YOUR_MYSQL_ROOT_PASSWORD"
    ```
    
    最后，启动CODE项目（注意把下面的127.0.0.1换成真实的IP）：
    
    ```
    （venv)$ gunicorn -w 2 -b 127.0.0.1:8000 app:app
    ```
    
    OK，到这里CODE就部署完成了，打开浏览器，输入：
    
    ```
    http://YOUR_IP:8000
    ```
    
    就可以看到CODE的界面了：

    ![enter image description here][1]    

    进入注册一个帐号，创建一个git库玩玩，everything dependes on you ^_<
    
    另外，用pip安装python包的时候，可能会遇到访问国外镜像速度太慢的问题，可以换用豆瓣的镜像（这里要赞一下，豆瓣的镜像访问速度非常快，而且各种包都很全，相比下，清华大学的镜像很多包都是没有的）。具体方法是打开 `~/.pip/pip.conf` 文件（可能不存在，编辑后保存就可以）， 输入如下内容：
    
    ```    
    [global]                                  
    index-url = http://pypi.douban.com/simple/
    ```
    
    这样，就可以从豆瓣的镜像下载python的库了，速度杠杠滴啊，哈哈。
    


  [1]: http://lanceju-com.qiniudn.com/lanceju%E8%B1%86%E7%93%A3CODE.jpg
  
  
作者：crystaldust
E-mail：juzhenatpku@gmail.com
