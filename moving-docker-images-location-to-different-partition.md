
##将Docker image放在其他的分区

>注: 原作者[shuron](http://alexander.holbreich.org/author/admin/),原文[地址](http://alexander.holbreich.org/2014/07/moving-docker-images-different-partition/)

缺省情况下，Docker会将所有的数据，包括image都放在```/var/lib/docker```(至少在Debian中)。这可能会引发磁盘空间的问题。我家里的服务器就遇到了这样的情况，所以我必须更改docker的位置。我必须把```/var/lib/docker```挂载到新的空间。因为我的```/var```和```/usr```挂载在不同的分区和磁盘下，我尝试通过显式的将docker的缺省位置挂载到```/usr/local/docker```下来解决空间的问题。

首先，备份fstab

```bash
sudo cp /etc/fstab /etc/fstab.$(date +%Y-%m-%d)
```
然后停止docker，通过```rsync```拷贝所有文件，保留所有的属性。
```bash
sudo service docker stop
sudo mkdir /usr/local/docker
sudo rsync -aXS /var/lib/docker/. /usr/local/docker/ [/bash]
```
检查所有的东西是否正确的拷贝很重要。我试过肉眼检查，```diff -r```命令也很有用。好吧，创建新的挂载点并使其在```fstab```中长期存在很关键。下面是我的```fstab```的配置。
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

现在Docker就有了足够的空间了。
