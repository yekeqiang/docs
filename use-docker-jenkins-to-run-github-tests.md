# 使用Docker和Jekins运行Github测试用例

***
作者：[Greg Bergé](http://www.therightcode.net/use-docker-jenkins-to-run-github-tests/) 

译者：[我不围观](http://weibo.com/ooutman)
***

## 为啥不用Travis呢？

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/travis-mascot.png)

现如今，采用持续集成(continous integration)平台是必须的。如果你用过[Github](https://github.com/)，你很可能了解[Travis](http://www.therightcode.net/use-docker-jenkins-to-run-github-tests/travis-ci.org)。Travis是最好的持续集成平台之一，免费开源，易于设置，在小的开源模块中，绝对是我的最爱。

如果你在开发大的项目，很可能需要操作你的测试环境，保证和产品环境的一致性。这种情况下，Travis可能不是最好的工具，因为它没有本地存储，而且不免费。

## Jenkins的缺陷

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-mascot.png)

Jenkins可能是运行持续集成测试最灵活和可配置的工具，你能用它干任何事。

Jenkins最大的问题是缺乏隔离性，所有的测试都在一个环境中运行。比如，如果你在测试中需要一个数据库，需要安装到Jenkins服务器中。问题就来了，这在所有的任务中是共享的，可能存在冲突，你就需要在每次测试完成后恢复数据库。

## Docker是救星

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/docker-logo.png)

Jenkins不能给每次测试都原生提供一个新的专用环境。为什么不能在虚拟机中运行每次测试呢？

虚拟机非常重型，而且很难维护，真的。

但是，我们已经在2014年了，有[Docker](http://docker.io/)啊。Docker就是这种场景下最好的工具，由于采用了LXC，[比虚拟机要要轻量级](http://stackoverflow.com/questions/16047306/how-is-docker-io-different-from-a-normal-virtual-machine)，而且[稳定](http://blog.docker.com/2014/06/its-here-docker-1-0/)。采用Docker，就可能在每个专有的环境下运行单独的测试用例。能够避免冲突的问题，而且保证环境之间的一致性。

## Jenkins + GitHub组合拳

首先就是要设置Jenkins和Github的关联关系，可以安装[Jenkins的Github插件](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin)搞定。

### SSH密钥文件

采用SSH最简单的方法是给“jenkins”用户[生成一个密钥文件](https://help.github.com/articles/generating-ssh-keys)并添加到账号中。你要是在一个组织中工作，你可以创建一个机器人账号，拥有你的Github项目管理权限并添加了生成的SSH密钥文件。

### 全局配置

首先你必须配置Github的API令牌，这样Jenkins才能管理项目的钩子。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-github-web-hook.png)

### 任务设置

首先你必须在“Github project”中填入你的Github项目地址。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-github-project.png)

然后你需要设置git仓库，只需要填入仓库地址，和其他项一样就可以了。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-git-repository.png)

触发构建最佳的方式是使用Github的Web钩子，Jenkins必须设置成public，“Build when a change is pushed to Github”选项是选中的。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-github-trigger-build.png)

最后，需要让Jenkins在Github提交时设置构建状态。在Job设置中，必须增加一个“Action after build”，名字是“Set build status on GitHub commit”。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-set-build-status-on-github-commit.png)

这个设置允许你直接在Github中看到pull-request状态。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/github-pr-status.png)

## 设置Docker

首先是在Jenkins服务器上[安装Docker](https://docs.docker.com/installation/ubuntulinux/)。

你可以把用户jenkins加入到docker组中，避免Docker运行在sudo模式。

```
sudo gpasswd -a $USER docker  
sudo service docker restart  
```

设置完后，你就可以不用sudo就可以运行```docker ps```命令了。如果还是不行，可以试试退出会话，建议是重启Jenkins。

## 运行测试

我们尝试在一个新的docker容器写入运行测试的命令，假设你的Github项目测试需要一个Postgres数据库。

### Postgres容器

首先我们需要一个运行postgres的容器。我们可以用官方的Postgres镜像，名字就是“postgres”，如果你的服务器上没有这个镜像，你不需要手动pull，Docker会自动的从[Docker仓库](https://registry.hub.docker.com/)下载。

```
docker run -d postgres  
```

我们有一个运行Postgres的容器了，这个容器是运行在分离模式的(-d)，完全隔离的（没有端口开放）。

### 测试容器

然后我们可以启动项目的测试了。

```
APP_DIR="/usr/src/app"  
docker run -v $WORKSPACE:$APP_DIR -w $APP_DIR node:0.10.28 bash -c 'npm install && npm test'  
```

这行有一点点复杂，我简单解释一下：

+ ```-v $WORKSPACE:$APP_DIR```: 我们挂载当前目录（利用jenkins的环境变量$WORKSPACE）到容器的“/usr/src/app”目录中。
+ ```-w $APP_DIR```: 我们定义当前工作目录为之前挂载的目录。
+ ```node:0.10:28```: 官方的节点镜像v0.10.28
+ ```bash -c 'npm install && npm test'```: 运行bash脚本“npm install && npm test”。我们需要用"bash -c"，这样就可以使用&&了。

### 把容器连接起来

现在，测试环境是不能和数据库通信的，我们还需要设置数据库和容器的关联。

```
DB_NAME="$BUILD_TAG-db"  
docker run -d --name DB_NAME postgres  
APP_DIR="/usr/src/app"  
docker run -v $WORKSPACE:$APP_DIR -w $APP_DIR --link DB_NAME:database node:0.10.28 bash -c 'npm install && npm test'  
```

我们给数据库容器起了一个名字，然后连接到测试容器上。在测试容器中，数据库的名字是“database”，所以在测试中要使用数据库，数据库的主机名就必须是“database”。要了解更多的Docker容器连接，可以看看[这个例子](https://docs.docker.com/userguide/dockerlinks/)。

好了，这两个容器已经连接上了，可以进行测试了。你可能还有一些其他问题。

### Postgres还没有启动

可能docker启动太快，还要等待数据库就绪。我是在run命令后加了```sleep 1```搞定的，虽然不够优雅，，正常工作是没问题。

```
DB_NAME="$BUILD_TAG-db"  
docker run -d --name DB_NAME postgres  
sleep 1  
APP_DIR="/usr/src/app"  
docker run -v $WORKSPACE:$APP_DIR -w $APP_DIR --link DB_NAME:database node:0.10.28 bash -c 'npm install && npm test'  
```

### 我不能安装私有的node_modules

要安装私有的node_modules和其他需要SSH密钥的模块，需要在容器中放入SSH密钥。操作比较简单，只需要挂载目录 (```-v ~/.ssh:/root/.ssh```），然后运行chmod命令 (```chmod 700 /root/.ssh/id_rsa```) ，保证设置了密钥文件正确的权限。

```
DB_NAME="$BUILD_TAG-db"  
docker run -d --name DB_NAME postgres  
sleep 1  
APP_DIR="/usr/src/app"  
docker run -v ~/.ssh:/root/.ssh -v $WORKSPACE:$APP_DIR -w $APP_DIR --link DB_NAME:database node:0.10.28 bash -c 'chmod 700 /root/.ssh/id_rsa && npm install && npm test'  
```

### 我不能清空工作目录

运行docker容器的默认用户是“root”，所以挂载目录后写入Docker容器的文件所属者都是“root”。这就是不能用“jenkins”用户清空工作目录的原因。

有很多的解决方案，最简单的就是在```npm install```之后运行```chmod -R 777 node_modules```，最终的脚本如下：

```
DB_NAME="$BUILD_TAG-db"  
docker run -d --name DB_NAME postgres  
sleep 1  
APP_DIR="/usr/src/app"  
docker run -v ~/.ssh:/root/.ssh -v $WORKSPACE:$APP_DIR -w $APP_DIR --link DB_NAME:database node:0.10.28 bash -c 'chmod 700 /root/.ssh/id_rsa && npm install && chmod -R 777 node_modules && npm test' 
```

第二种方法复杂一点，创建一个有“jenkins”用户的镜像节点，这样就可以用“jenkins”用户 (```--user jenkins```)运行命令了。你还是可以直接把SSH密钥文件放到VM中，最终的脚本如下：

```
DB_NAME="$BUILD_TAG-db"  
docker run -d --name DB_NAME postgres  
sleep 1  
APP_DIR="/usr/src/app"  
docker run --user jenkins -v $WORKSPACE:$APP_DIR -w $APP_DIR --link DB_NAME:database custom-node:0.10.28 bash -c 'npm install && npm test'  
```

## 移除容器

容器在任务完后仍然是启动状态的。用Jenkins的[Process Tree Killer](https://wiki.jenkins-ci.org/display/JENKINS/ProcessTreeKiller)，就可以在任务结束后停止所有的进程。但是在容器中，就搞不定了，因为它们是运行在分离模式的。

要在测试运行失败的情况下也能保证容器被移除，需要用到[Post Build Task插件](http://wiki.hudson-ci.org/display/HUDSON/Post+build+task)。

最后要做的就是在构建结束后移除容器。

![](https://raw.githubusercontent.com/outmana/outmana.github.io/source/source/_posts/use-docker-jenkins-to-run-github-tests/jenkins-post-build-task.png)


***
这篇文章由 [Greg Bergé](http://blog.arungupta.me/2014/07/getting-started-with-docker/)  撰写，[我不围观](http://weibo.com/ooutman) 翻译。点击 [这里](http://www.therightcode.net/use-docker-jenkins-to-run-github-tests/) 阅读原文。
***
