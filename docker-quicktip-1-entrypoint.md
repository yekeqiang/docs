#Docker Quicktip #1: Entrypoint
***
#Docker 快手/速学/易用/简单有效 技巧 #1：Entrypoint
***
#简单有效的Docker技巧 #1:Entrypoint
***
#Docker速成技巧 #1:Entrypoint
***
#Docker速成小贴士
***
#Docker快速运用小技巧系列 1：Entrypoint
***
#Docker即学即用小技巧系列 1：Entrypoint
***
#Docker实用小技巧系列 1：Entrypoint
---

The first tip is aptly named “Entrypoint”. In this tips I kind of expect that you’ve played around with Docker a bit, probably even have some containers running for your dev environment. So, in short, if you haven’t played yet, go play and come back!


Entrypoint不但是docker中的一个命令参数，而且"Entrypoint"这个词本身还具有“切入点”的意思，正好用来做我们的第一条技巧。本例中，我假设你已玩了一阵子Docker，如果你正在自己的开发环境中使用docker容器那就更好了。总之，你得对Docker有一定的了解，没玩过的要先玩一玩再来看这篇文章哦。

Entrypoint is great. It’s pretty much like CMD but essentially let’s you use re-purpose CMD as runtime arguments to entrypoint. For example…

Instead of:

Entrypoint非常好用。它可以把docker命令作为参数传递给容器，产生灵活的变化，例如：

***上面这句是我根据上下文翻译的，请@meaglith看看怎么翻译更合适***

之前运行命令，你可能是：




`docker run -i -t -rm busybox /bin/echo foo`

You can do:

有了Entrypoint，你可以这样做：

`docker run -i -t -rm -entrypoint /bin/echo busybox foo`


This sets the entrypoint, or the command that is executed when the container starts, to call /bin/echo, and then passes “foo” as an argument to /bin/echo.

其实Entrypoint就是容器开始运行时执行的命令。因此上面的语句就是让docker容器在开始运行时把"foo"作为参数来调用/bin/echo。




Or you can do, in a Dockerfile:

你也可以在Dockerfile中指定entrypoint：

    FROM busybox
    ENTRYPOINT ["/bin/echo", "foo"]

    docker build -rm -t me/echo .
    docker run -i -t -rm me/echo bar

This passes bar as an additional argument into /bin/echo foo, resulting in “/bin/echo foo bar”

这样，就把"bar"作为额外的参数传递给了`/bin/echo foo`，最终相当于执行`/bin/echo foo bar`

Why would you want this? You can think of it as turning CMD into a set of optional arguments for running the container. You can use it to make the container much more versatile. This will lead into the next tip “Exec it”

为什么要用Entrypoint呢？可以认为，通过Entrypoint，我们可以把docker的命令变成一组可选的参数来运行容器。通过它，可以让容器变得更加灵活多用。由此我们引出下一个技巧："Exec it".



