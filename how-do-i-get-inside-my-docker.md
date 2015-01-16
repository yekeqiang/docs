# 如何进入docker容器中执行命令的另类方法

---

##### 作者：[孙圣翔](https://github.com/codeskyblue)

---

Stackover 上有人提了一个问题，[如何在一个已经运行的 docker 容器中执行命令](http://stackoverflow.com/questions/17903705/is-it-possible-to-start-a-shell-session-in-a-running-container-without-ssh) 。当然第一想到的就是开一个 sshd 。不过很快就有大神发了一篇文章，[为什么在 docker 容器中启动 sshd 是不好的](http://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/) 。

大神既然不让开 ssh ，标题上的这个需求仍然是需要满足的。 [nsenter](https://github.com/jpetazzo/nsenter) 、[nsinit](https://gist.github.com/ubergarm/ed42ebbea293350c30a6) 这类东西，在某些机器上偏偏跑不起来。机器上的 docker 版本都到1.2.0了。

我大概想了一个办法，思路也很简单。 docker 上开一个 server 端，监听 sock 文件。然后另起一个 client 端，通过这个 sock ，把命令发给 server 。


我说的有点简单了点，实现起来，还是要花点时间的。代码我写好了，放在了 [github](https://github.com/codeskyblue/unsafessh) 上。

## 演示

先启动一个docker容器

```
docker run --rm -ti -v /tmp --name=demoussh codeskyblue/unsafessh bash
echo 123456 > /hello # run in container
```

然后

```
docker run --rm --volumes-from=demoussh codeskyblue/unsafessh unsafessh exec -- cat /hello
# 出现123456就算正常了
```

是不是很简单，核心在于有了 unsafessh 这个程序。花了我两天时间，哎，感觉自身还是有很大的提升空间。

---

本文原载于作者的 [博客](http://shengxiang.me) ，原文是 [如何进入docker容器中执行命令的另类方法](http://shengxiang.me/article/23/how-do-i-get-inside-my-docker.html) 。
