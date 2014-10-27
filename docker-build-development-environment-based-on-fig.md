# 深入浅出Docker（五）：基于Fig搭建开发环境

---

##### 作者：肖德时

---

【编者按】Docker是PaaS供应商dotCloud开源的一个基于LXC 的高级容器引擎，源代码托管在 GitHub 上, 基于Go语言开发并遵从Apache 2.0协议开源。Docker提供了一种在安全、可重复的环境中自动部署软件的方式，它的出现拉开了基于云计算平台发布产品方式的变革序幕。为了更好的促进Docker在国内的发展以及传播，我们决定开设《[深入浅出Docker](http://www.infoq.com/cn/dockers)》专栏，邀请Docker相关的布道师、开发人员、技术专家来讲述Docker的各方面内容，让读者对Docker有更深入的了解，并且能够积极投入到新技术的讨论和实践中。另外，欢迎加入InfoQ Docker技术交流群交流Docker的最佳实践，QQ群号：365601355。

## 1. 概述

在搭建开发环境时，我们都希望搭建过程能够简单，并且一劳永逸，其他的同事可以复用已经搭建好的开发环境以节省开发时间。而在搭建开发环境时，我们经常会被复杂的配置以及重复的下载安装所困扰。在Docker技术未出现之前，我们可以使用Pupet、Chef、Ansible等配置管理工具把复杂的配置管理起来，这样的管理配置技术仍然是目前比较流行的方式之一。配置管理工具使用的都是自己的DSL语法定义，考虑到环境的复杂性，配置一套通用的开发环境需要针对各个系统定制，对于大部分开发环境这种维护成本仍然是很高的。Docker技术出现之后，系统的依赖问题得到了彻底的解决，我们可以通过镜像的方式简化环境的安装。结合Docker的开发部署工具Fig，我们可以使用fig.yml文件来定义所有的环境，一次定义，多处使用，简单而且高效。

![alt](http://resource.docker.cn/fig-png)

### 1.1 打包的方式

Docker本省并不能创建一个真实的虚拟机，它只是基于Linux Kernel把额外的系统文件做了封装，并利用Linux Kernel相关的技术如Cgroup、Namespace隔离用户应用。应用Docker技术，团队之间通过共享Image或者Dockefile来复用开发环境。为了简化写Dockerfile的方式 ，Fig提供更加精简的DSL定义文件fig.yml，可以让新成员快速搭建开发环境并将精力投入到开发过程中去，而不是研究如何正确安装并配置诸如PostgreSQL之类的数据库。目前，软件开发需要的环境Image，大部分都可以在 [Docker Hub](https://hub.docker.com/) 中搜索到，需要使用时直接下载就可以使用。

### 1.2 应用组合的方式

使用Docker之后，我们不再需要在本地机器安装所有的软件包。我们可以根据项目需要在Docker Hub上搜索相关软件的Image，然后使用Fig pull从Docker Hub上直接下载并由Fig调用Docker的Link命令把Image关联起来，这样所有应用都可以直接调用指定端口的服务，比如Mysql的3306端口提供数据库服务。开发者并不需要对Docker有太多的了解，只需要掌握常用的几条Fig命令就可以随时调试自己的环境。

### 1.3 环境共享的方式

Fig直接定义好了Image，我们不需要过多的关心容器或者镜像。在分享环境时，只需要把对应的Dockerfile和fig.yml文件分享给同事，他们就可以在自己的机器上运行并搭建出需要的环境，且不用再担心环境依赖带来的意外调试烦恼。团队成员在git clone项目代码后，就可以如下图一样使用一条命令启动自己的开发环境：

![alt](http://resource.docker.cn/fig-image.gif)

## 2. Fig安装指南

首先，我们需要安装Docker Engine，官方针对主流的系统提供了对应的[安装手册](https://docs.docker.com/installation/)。这里，我的开发机系统是一台MacBook Pro，所以我需要安装[boot2docker](http://www.fig.sh/install.html)来支持Docker。

接下来，我们就可以安装对应系统版本的Fig运行文件。比如在Mac OS或者在64位的Linux上安装Fig：

```
$ curl -L https://github.com/docker/fig/releases/download/1.0.0/fig-`uname 
-s`-`uname -m` > /usr/local/bin/fig; chmod +x /usr/local/bin/fig
```

当以上的安装方式不能成功的话，我们可以选择使用Python Package的安装方式：

```
$ sudo pip install -U fig
```

最后，不管使用什么Linux系统，安装完Fig之后通过运行以下命令来确保环境的一致性。

```
$ fig --version
```

如果一切顺利的话，恭喜你，Fig安装成功了。下面请随我一起学习如何使用Fig配置开发环境。

## 3. Rails开发环境配置详解

我们来配置一套最常用的Rails+PostgreSQL项目。

第一步，确保Fig已经正确的安装到主机系统，如果还没有，请参考上一章节的安装指南。

第二步，在项目根目录下存放一个名为 **Dockerfile** 文件：

```
FROM ruby
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
RUN bundle install
ADD . /myapp
```

第三步，创建一个 **Gemfile** 文件用来定义Rails软件包。 内容如下所示：

```
source 'https://rubygems.org'
gem 'rails', '4.0.2'
```

第四步，创建一个fig.yml文件，并使用以下配置文件来做最后的环境初始化。

```
db:
  image: postgres
  ports:
    - "5432"
web:
  build: .
  command: bundle exec rackup -p 3000
  volumes:
    - .:/myapp
  ports:
    - "3000:3000"
  links:
    - db
```

第五步，当前你目录下空空如也，使用如下命令可以生成一套Rails项目骨架：

```
$ fig run web rails new . --force --database=postgresql --skip-bundle
```

当你跑完以上命令，你就会得到一个崭新的Rails项目。

```
$ ls
Dockerfile Rakefile config fig.yml public vendor
Gemfile app config.ru lib test
README.rdoc bin db log tmp
```

编辑一下Gemfile文件，去掉therubyracer包的注释, 让Rails依赖的Javascript的运行时环境。

```
gem 'therubyracer', platforms: :ruby
```

所有的文件编辑都做完之后，运行命令创建开发环境image。

```
$ fig build
```

运行完build命令后，我们就有了可以立即使用的Image。这两个Image的名字分别是web和db。为了让db能连上web，我们通常还需要修改database.yml来支持数据库连接。

```
development: &default adapter: postgresql encoding: unicode database: postgres pool: 5 username: postgres password: host: db test: <<: *default database: myapp_test
```

好了，让我们运行一下：

```
$ fig up
```

命令行将显示如下日志：

```
Recreating figtest_db_1...
Creating figtest_web_1...
Attaching to figtest_db_1, figtest_web_1
db_1  | LOG:  database system was shut down at 2014-10-01 23:53:11 UTC
db_1  | LOG:  autovacuum launcher started
db_1  | LOG:  database system is ready to accept connections
web_1 | [2014-10-01 23:53:16] INFO  WEBrick 1.3.1
web_1 | [2014-10-01 23:53:16] INFO  ruby 2.1.2 (2014-05-08) [x86_64-linux]
web_1 | [2014-10-01 23:53:16] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
db_1  | FATAL:  database "myapp_development" does not exist
db_1  | FATAL:  database "myapp_development" does not exist
web_1 | 192.168.59.3 - - [01/Oct/2014 23:53:40] "GET / HTTP/1.1" 500 13476 0.5112
web_1 | 192.168.59.3 - - [01/Oct/2014 23:53:40] "GET %2Ffavicon.ico HTTP/1.1" 200 - 0.0067
```

通过以上日志可以知道开发数据库还没有创建。这时我们可以开启另外一个terminal，创建开发数据库：

```
$ fig run web rake db:create
```

当以上所有步骤都成功运行后，就需要来验证开发环境的正确性了。通过访问 `http://localhost:3000/` 我们应该可以看到熟悉的Rails欢迎页面：

![alt](http://resource.docker.cn/rails-png)

## 4. Django开发环境配置详解

接下来让我们使用Fig来配置一套运行Django/PostgreSQL的应用程序吧。

首先我们新建一个项目目录，并在目录里创建3个文件。第一个文件是Docker镜像的定义文件: **Dockerfile**，用来描述安装在Docker容器里软件依赖关系。文件如下：

```
FROM python:2.7
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

以上文件描述这个Image将安装requirements.txt指定的python依赖包。

第二个文件是**requirements.txt**，它是python依赖包定义描述文件，内容如下：

```
Django
psycopg2
```

最后，Fig需要把所有的环境都连接起来运行。这个文件名为 fig.yml 。它描述项目需要的服务组件、指定镜像的版本、如何连接服务、什么卷可以被载入容器内部、什么端口可以暴露出来等。内容形如：

```
db:
  image: postgres
web:
  build: .
  command: python manage.py runserver 0.0.0.0:8000
  volumes:
    - .:/code
  ports:
    - "8000:8000"
  links:
    - db
```

到此处，我们就可以使用 fig run 来创建一个Django项目了：

```
$ fig run web django-admin.py startproject figexample .
```

当你运行完之后，可以在当前目录下看到创建的新项目文件：

```
$ ls
Dockerfile fig.yml figexample manage.py requirements.txt
```

接下来的事情就是创建数据库链接，修改 figexample/settings.py 的DATABASES = ...部分。

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

这个配置信息是用来连接postgres镜像服务。

然后，运行命令 fig up来启动容器。

```
Recreating myapp_db_1...
Recreating myapp_web_1...
Attaching to myapp_db_1, myapp_web_1
myapp_db_1 |
myapp_db_1 | PostgreSQL stand-alone backend 9.1.11
myapp_db_1 | 2014-01-27 12:17:03 UTC LOG:  database system is ready to accept connections
myapp_db_1 | 2014-01-27 12:17:03 UTC LOG:  autovacuum launcher started
myapp_web_1 | Validating models...
myapp_web_1 |
myapp_web_1 | 0 errors found
myapp_web_1 | January 27, 2014 - 12:12:40
myapp_web_1 | Django version 1.6.1, using settings 'figexample.settings'
myapp_web_1 | Starting development server at http://0.0.0.0:8000/
myapp_web_1 | Quit the server with CONTROL-C.
```

这个时候，你可以开启浏览器，然后输入 [localhost:8000](http://localhost:8000/) 就可以访问这个Django应用。

注意，当你跑起来应用之后，就可以初始化数据库了。这里，请一定要保证fig up是在运行中，并另外开启一个命令行窗口执行一下命令：

```
$ fig run web python manage.py syncdb
```

如果你反复使用了fig up之后，可以体会到它一次性会把web和db两个镜像一起启动，如果你CONTROL-C之后，数据库也会停止服务。你甚至可以fig run web /bin/bash的方式直接进入到容器里看看。

## 5. Wordpress开发环境配置详解

Fig一样可以应付PHP应用的开发需求。

首先，创建一个Dockerfile来支持生成开发用Image。

```
$ curl https://wordpress.org/latest.tar.gz | tar -xvzf -
```

以上这条命令创建名为 wordpress 的目录。进入到此目录，创建镜像文件 Dockerfile，内容如下：

```
FROM stackbrew/ubuntu:13.10
RUN apt-get update && apt-get install php5 php5-mysql -y
ADD . /code
```

下一步创建 fig.yml，它将定义出web服务和mysql db服务。

```
web:
  build: .
  command: php -S 0.0.0.0:8000 -t /code
  ports:
    - "8000:8000"
  links:
    - db
  volumes:
    - .:/code
db:
  image: mysql
  environment:
    MYSQL_DATABASE: wordpress
    MYSQL_ROOT_PASSWORD: wordpress
```

wordpress有一个支持文件需要修改，它是wp-config.php:

```
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'wordpress');
define('DB_HOST', getenv("DB_1_PORT_3306_TCP_ADDR") . ":" . getenv("DB_1_PORT_3306_TCP_PORT"));
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');

define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');

$table_prefix  = 'wp_';
define('WPLANG', '');
define('WP_DEBUG', false);

if ( !defined('ABSPATH') )
    define('ABSPATH', dirname(__FILE__) . '/');

require_once(ABSPATH . 'wp-settings.php');
```

以上三个文件都修改后，就可以在此目录下运行 fig up 来启动 wordpress 。可以使用浏览器访问 [localhost:8000](http://localhost:8000/) 浏览 Wordpress 首页。

![alt](http://resource.docker.cn/wordpress.png)

## 6. 结论

通过Fig来构建基于Docker的开发环境可以让我们的开发事半功倍。其实如果读者按照我的步骤一步一步搭建开发环境，仍然会有很多挑战。

首先，因为国内带宽的限制，从官方下载Image是一个长时间等待的过程。即使Docker公司已经使用了CDN服务，但在国内网络使用仍然很慢。为了能快速下载，我还常备一条VPN线路作为数据通道来下载Image。目前这是最理想的方式，可以节省很多开发时间。

第二，fig up并不能保证Image能一次成功。但不需要灰心。你可以fig run web /bin/bash直接到容器里面调试。我遇到过多次费解的问题都是直接在里面跑命令解决的。当你退出时，fig会自动更新到相应的Image里。当你再次fig up，它会使用新Image去创建这个运行容器。

第三，fig up会创建两个以上的容器实例，在退出容器后再次启动fig up，并不会重用之前退出的容器实例，而是新建容器实例。像fig up这类常用命令运行多次之后会导致过期的容器文件仍然存储在你的开发机器上并占用硬盘空间，并且Fig还没有提供对应的命令自动清理它们。目前可以解决的办法是直接使用Docker rm/rmi命令手工清除。

以上总结出的实战经验，仍然不能掩盖Fig特有的亮点：运用Fig可以定义统一运行步骤，让部署环境可以完全的隔离在一个独立的容器环境里，并且这个隔离环境可以在开发、测试、生产多个步骤中保持一致。当前Fig项目还很年轻，需要大家多参与项目的讨论，提出自己的问题才能让Fig更好使用，更多信息可以到这里[查阅](https://github.com/docker/fig)。

## 7. 作者简介

肖德时, Red Hat Engineering Service/HSS 内部工具组Team Leader。Nodejs开源项目nodejs-cantas Lead Developer。擅长企业内部工具的设计以及实现。开源课程Rails Starter的发起人。rubygem: lazy_high_charts的Maintainer。Twitter账号：xds2000，邮箱：xiaods@gmail.com

## 8.下期预告

Docker引擎可以在多台主机上启动成千上百的实例，那么如何管理这些容器实例显然会成为一个挑战。所以下期我将给大家介绍一款非常出色的容器集群管理框架Kubernetes，敬请期待！

感谢[郭蕾](http://www.infoq.com/cn/author/%E9%83%AD%E8%95%BE)对本文的审校和策划。

---

本文原载于 InfoQ 中文站，得到作者与 InfoQ 中文站授权后，我们将其转载。原文地址：[深入浅出Docker（五）：基于Fig搭建开发环境](http://www.infoq.com/cn/articles/docker-build-development-environment-based-on-fig)