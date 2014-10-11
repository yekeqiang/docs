#将 Docker image 放在其他的分区

#####作者：[Alexander Holbreich](https://twitter.com/shuron)

#####译者：[Bo Wen](http://weibo.com/u/2537862844)

***
缺省情况下， Docker 会将所有的数据，包括 image 都放在  ```/var/lib/docker``` （至少在Debian中），这可能会引发磁盘空间问题。鉴于我家里的服务器就遇到了这样的情况，我更改 docker 的位置，把 ```/var/lib/docker``` 挂载到新的空间。因为我的 ```/var``` 和 ```/usr``` 挂载在不同的分区和磁盘下，我尝试通过将 docker 的缺省位置挂载到 ```/usr/local/docker``` 下来解决空间的问题。

**首先，备份 fstab**

```bash
sudo cp /etc/fstab /etc/fstab.$(date +%Y-%m-%d)
```

**然后停止 docker ，通过 ```rsync``` 拷贝所有文件，保留所有属性。**

```bash
sudo service docker stop
sudo mkdir /usr/local/docker
sudo rsync -aXS /var/lib/docker/. /usr/local/docker/ [/bash]
```

**检查所有的东西是否被正确拷贝**，这步非常重要重要。我试过肉眼检查， ```diff -r``` 命令也很有用。创建新的挂载点并使其在 ```fstab``` 中长期存在也很关键。下面是我的 ```fstab``` 的配置。

```bash
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# ...
/usr/local/docker /var/lib/docker none bind 0 0
```

;file system> <mount point> <type> <options> <dump> <pass>
不用重新启动，加载新的配置。


```bash
mount -a
```

现在 Docker 就有足够的空间了。

***
#####这篇文章由 [Alexander Holbreich](https://twitter.com/shuron) 撰写，[Bo Wen](http://weibo.com/u/2537862844) 翻译。点击 [这里](http://alexander.holbreich.org/2014/07/moving-docker-images-different-partition/) 阅读原文。

#####The article was contributed by [Alexander Holbreich](https://twitter.com/shuron), click [here](http://alexander.holbreich.org/2014/07/moving-docker-images-different-partition/) to read the original publication.
