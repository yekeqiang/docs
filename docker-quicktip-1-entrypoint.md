# Docker Quicktip #1: Entrypoint

![alt](http://resource.docker.cn/quick-tip.jpg)

##### 作者：[Brian Goff](https://github.com/cpuguy83) 

##### 译者：[巨震](https://github.com/crystaldust)

---

" Entrypoint " 这个词本身还具有“切入点”的意思，正好用来做我们介绍的第一条技巧。本文中假设你已使用 Docker 一段时间，如果你自己的开发环境中正运行着一些 Docker 容器那就更好了。总之，你要对 Docker 有一定的了解，如果没用过 Docker 的话还是先试用一下再来看这篇文章吧。

Entrypoint 非常好用。它非常像 CMD ，不同的是它是在容器开始运行时把命令作为运行时参数传入到容器中。例如：


之前运行命令可能是：

`docker run -i -t -rm busybox /bin/echo foo`


有了 Entrypoint，你可以这样：

`docker run -i -t -rm -entrypoint /bin/echo busybox foo`

其实 Entrypoint 就是容器开始运行时执行的命令。因此上面的语句就是让 Docker 容器在开始运行时把 "foo" 作为参数来调用 /bin/echo 。


你也可以在 Dockerfile 中指定 Entrypoint：

```
    FROM busybox
    ENTRYPOINT ["/bin/echo", "foo"]

    docker build -rm -t me/echo .
    docker run -i -t -rm me/echo bar
```

这样，就把 " bar " 作为额外的参数传递给了 `/bin/echo foo` ，最终相当于执行 `/bin/echo foo bar` 。


为什么要用 Entrypoint 呢？可以认为，通过 Entrypoint，我们可以把 Docker 的命令变成一组可选的参数来运行容器。通过它，可以让容器变得更加灵活多用。由此可以引出下一个技巧：" Exec it "，我们下篇再叙。


---
##### 这篇文章由 [Brian Goff](https://github.com/cpuguy83) 发表，点击 [此处](http://www.tech-d.net/2014/01/27/docker-quicktip-1-entrypoint/)可查阅原文。

##### The article was contributed by [Brian Goff](https://github.com/cpuguy83) , click [here](http://www.tech-d.net/2014/01/27/docker-quicktip-1-entrypoint/) to read the original publication.


