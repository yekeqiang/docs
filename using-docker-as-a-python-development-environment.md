# 使用Docker做Python开发环境

***
作者：[Adrian Mouat](http://continuousdelivery.uglyduckling.nl/uncategorized/using-docker-as-a-python-development-environment/) 

译者：[我不围观](http://weibo.com/ooutman)
***

**(让我们忘了virtualenv吧)**

本文将介绍一种使用Docker开发Python程序可行的方法（主要是Web应用程序）。尽管我聚焦的是*[Flask](http://flask.pocoo.org/)*微框架来开发Python，其实是想展示怎么用Docker来更好的开发和分享应用程序（可以是任何语言或者框架），通过封装依赖关系，显著的减少开发环境和产品环境之间的差异。

大多数的Python程序员开发Python程序的时候会用*[virtualenv](https://virtualenv.pypa.io/en/latest/virtualenv.html)*，它提供了一致简单易用的机制来保证和其他应用、操作系统冲突的特定依赖有效（最明显的就是Python的版本，还有特定版本的库）。我个人向来不喜欢virtualenv，主要原因有：

	* 我经常会忘记启动virtualenv，出现一些莫名其妙的错误，我就抓瞎了；切换项目的时候也会忘记修改virtualenv
	* virtualenv并不是提供真正的隔离，只是在Python这个层面（系统库和非Python的依赖还是会导致问题）
	* 我特别不想在产品中运行virtualenv，这就会导致开发和产品环境的不一致
	* 我经常感觉有点"极客"，依赖修改脚本和配置新的路径

*（可以看看[pythonrants上的博客](https://pythonrants.wordpress.com/2013/12/06/why-i-hate-virtualenv-and-pip/)，了解为啥你应该不想用virtualenv)*

好了，那Docker是怎么改进的呢？Docker提供了轻量级的VM（容器的另外一个说法），可以更高层面的隔离和减少开发与产品环境之间差异。（如果你对Docker不了解，想要学习一下，可以看看我在Edinburgh techmeetup上的访谈[Docker简介](https://vimeo.com/96474917)）

我们构建了一个小的虚拟化Web应用，我和Mark Coleman用的是这个方法（[详细文档见链接](http://continuousdelivery.uglyduckling.nl/continuous-delivery/rapid-prototyping-a-python-web-application-with-vagrant-and-docker-part-1-development/)）。这个需要构建基础镜像，包括安装Python 2.7、Flask依赖和PostgreSQL。我现在用这个镜像来一步一步的开发一个hello world的Web应用。假定你是在Linux上开发，安装了git和docker，这个过程和MacOS是非常接近的。开始克隆和构建基础镜像吧：

```
$ git clone https://github.com/mrmrcoleman/python_webapp
$ docker build -t python_webapp .
```

现在，放一些代码到容器中运行起来。具体的操作方式是，在刚才的Docker镜像上启动一个新项目，而不是直接修改原来的项目。

按照下面的目录结构创建一个新项目：
```
├── Dockerfile
 ├── example_app
 │   ├── app
 │   │   ├── __init__.py
 │   │   └── views.py
 │   └── __init__.py
 ├── example_app.wsgi
```

或者从<https://github.com/amouat/example_app.git>中克隆一个示例项目。

在example_app/app/__init__.py中写入：

```
from flask import Flask

app = Flask(__name__)
from app import views
```

其他的\_\_init\_\_.py留空，在views.py中写入：

```
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

这就是最小的flask“hello world”应用需要的所有代码了。我用的代码和这个[教程](http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)的很像，所以如果你完全不了解Flask或者Python，你可以跟着教程学习，把其中的virtualenv替换为Docker。

为了能在Docker容器中运行起来，还需要继续。example_app.wsgi文件中已经包含了Python和Web服务器交互的步骤（我们使用的是Apache）。这个文件应该包含：

```
import site
site.addsitedir('/opt/example_app/')
from app import app as application
```

最后，我们需要一个Dockerfile来构建和运行容器。我们是这么写的：

```
FROM python_webapp

MAINTAINER amouat

ADD example_app.wsgi /var/www/flaskapp/flaskapp.wsgi
CMD service apache2 start && tail -F /var/log/apache2/error.log
```

ADD这一行插入了一些启动WSGI的代码，CMD这一行在启动容器的时候启动Apache服务器，所有的错误都输出到stdout。

如果你像下面这样运行：

```
$ docker build -t example_app .
$ docker run -p 5000:5000 -v $(pwd)/example_app:/opt/example_app/ -i -t example_app
```

你就启动了一个网站，运行在localhost:5000，可以通过Web浏览器访问。如果是在VM或者vagrant中运行，记得要开放5000端口。

现在，我们运行了一个Web服务器，和产品中的使用方式就很接近了 （我特地用Apache来启动，而不是用默认的Python服务器，就是为了强调这一点）。既然我们在容器中插入了映射主机空间到容器的代码（-v 参数），我们还可以在Dockerfile中用ADD行插入代码，但是我们改了代码之后，每次都需要重新构建容器。

但是，这还不够好。在开发中，我们真是想用Python的Web服务器，它能极大的帮我们调试。谢天谢地，我们不需要修改Dockerfile就能完成。首先，在example_app目录下创建run.py文件，内容如下：

```
!flask/bin/python
from app import app
app.run(debug = True, host='0.0.0.0')
```

这会启动一个Python的Web服务器，能够调试并监听所有的网络接口，所以我们能够从容器外面访问。现在用下面的命令重启一下容器：

```
$ docker run -p 5000:5000 -v $(pwd)/example_app:/opt/example_app/ -i -t example_app python /opt/example_app/run.py
```

你应该又能够访问Web页面了。这次我们重写了Dockerfile中的CMD行，明确提供运行的命令（“python /opt/example_app/run.py”）。再修改主机上的代码，你就能够立马在Web页面上看到变化。

我们一起来看看现在完成什么了:

	* Web应用是运行在一个隔离的容器中的，容器里同时包含了Python依赖和系统依赖；
	* 能够用我们原有的编辑器或者IDE工具来开发代码，能够直接的看到变化，就像在本地编辑一样；
	* 开发环境比之前更接近产品环境了；
	* 站点中没有virtualenv。


如果你想知道是怎么开始的，可以看看[Mark Coleman的博客](http://continuousdelivery.uglyduckling.nl/continuous-delivery/rapid-prototyping-a-python-web-application-with-vagrant-and-docker-part-1-development/)。

不幸的是，还没这么轻松搞定所有的事。还有下面的这些问题：

	* 你很可能还有一些问题，需要使用virtualenv或者类似的解决方案，但是和OS的库版本或者程序的版本冲突；
	* 我们还没有完全解决数据托管的问题 -- 我们还需要测试一下；
	* 我假定的是“产品”就是一个Docker容器，但是现在在规则和Docker托管都不完善的时候，假定是不太成立的；

尽管如此，我仍然相信我们在开发环境上前进了一大步，未来部署软件和管理依赖的痛苦会大幅度减少。


***
这篇文章由 [Adrian Mouat](http://continuousdelivery.uglyduckling.nl/uncategorized/using-docker-as-a-python-development-environment/)  撰写，[我不围观](http://weibo.com/ooutman) 翻译。点击 [这里](http://continuousdelivery.uglyduckling.nl/uncategorized/using-docker-as-a-python-development-environment/) 阅读原文。
***