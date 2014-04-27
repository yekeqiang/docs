# Resizing Docker containers with the Device Mapper plugin
# 使用Device Mapper插件来改变Docker容器的大小

If you're using Docker on CentOS, RHEL, Fedora, or any other distro that doesn't ship by default with AUFS support, you are probably using the Device Mapper storage plugin. By default, this plugin will store all your containers in a 100 GB sparse file, and each container will be limited to 10 GB. This article will explain how you can change that limit, and move container storage to a dedicated partition or LVM volume.

如果你正在CentOS，REHL，Fedora或者其他磨人没有AUFS支持的Linux分发包上使用Docker，你可能会使用Device Mapper的存储插件。默认这个插件会把你所有的容器存储到一个100G的稀疏文件中，并且每个容器最大限制为10GB。这篇文章将会展示如何突破这个限制，并且把容器的存储移动到一个指定的分区或者LVM卷中。

## How it works
## 它是如何工作的


To really understand what we're going to do, let's look how the Device Mapper plugin works.

要真正理解我们将要做什么，让我们来看一下Device Mapper的插件是怎么工作的。

It is based on the Device Mapper "thin target". It's actually a snapshot target, but it is called "thin" because it allows thin provisioning. Thin provisioning means that you have a (hopefully big) pool of available storage blocks, and you create block devices (virtual disks, if you will) of arbitrary size from that pool; but the blocks will be marked as used (or "taken" from the pool) only when you actually write to it.

它是基于Device Mapper的“精简目标”的特性。它实际上是一个目标块设备的一个快照，但是它被称为“精简”是因为它允许精简配置。精简配置意味着你有一个(希望是很大的)可用存储块的池，接着你可以从那个池中创建任意大小的块设备(虚拟磁盘，如果你有需要)；当你真的要去写的时候，这些存储块将会被标记为已使用(或者从池中拿走）

This means that you can oversubscribe the pool; e.g. create thousands of 10 GB volumes with a 100 GB pool, or even a 100 TB volume on a 1 GB pool. As long as you don't actually write more blocks than you actually have in the pool, everything will be fine.

这个意味着你是可以超额使用这个池的;比如在一个100GB的池里面创建几千个10GB的卷
,甚至一个100TB的卷在一个1GB的池里面。只要你的真正写的块的容量不大于池的大小，你怎么做都是OK的

Additionally, the thin target is able to perform snapshots. It means that at any time, you can create a shallow copy of an existing volume. From a user point of view, it's exactly as if you now had two identical volumes, that can be changed independently. As if you had made a full copy, except that it was instantaneous (even for large volumes), and they don't use twice the storage. Additional storage is used only when changes are made in one of the volumes. Then the thin target allocates new blocks from the storage pool.

除此之外，精简目标的方式是可以做快照的。这个意味着任何嗣后，你都可以创建一个存在的卷的浅拷贝。从一个拥护的观点来看，这个就好像你有两个一样的卷,它们可以独立地各自修改。即使你做了一个完整的拷贝，除了在时间上它是瞬间的(即使是很大的卷)之外，它们不会使用两次重复的存储。额外的存储只有当其中任何有变化的时候才会发生。然后精简目标会从池里面分配一个存储快

Under the hood, the "thin target" actually uses two storage devices: a (large) one for the pool itself, and a smaller one to hold metadata. This metadata contains information about volumes, snapshots, and the mapping between the blocks of each volume or snapshot, and the blocks in the storage pool.

从本质上来看，“精简目标”实际上使用了两个存储设备:一个(大)的是存储块池自己,还有一个小的存储了一些元数据。这些元数据中包括了卷，快照，以及每个卷的块或者快照同存储池中块的映射的信息。

When Docker uses the Device Mapper storage plugin, it will create two files (if they don't already exist) in /var/lib/docker/devicemapper/devicemapper/data and /var/lib/docker/devicemapper/devicemapper/metadata to hold respectively the storage pool and associated metadata. This is very convenient, because no specific setup is required on your side (you don't need an extra partition to store Docker containers, or to setup LVM or anything like that). However, it has two drawbacks:

当Docker使用Device Mapper存储插件的时候，它会创建两个文件(如果它们不存在)在`/var/lib/docker/devicemapper/devicemapper/data`和`/var/lib/docker/devicemapper/devicemapper/metadata`来存储对应的存储池和相关的元数据。这个非常方便，因为你不需要做任何安装部署的工作(你不需要额外的分区来存储Docker容器，或者建立LVM或其他类似的东西)。然而它也有两个缺点:

－ the storage pool will have a default size of 100 GB

－ 存储池会有一个默认的100GB的容量

－ it will be backed by a sparse file, which is great from a disk usage point of view (because just like volumes in the thin pool, it starts small, and actually uses disk blocks only when it gets written to) but less great from a performance point of view, because the VFS layer adds some overhead, especially for the "first write" scenario.

－ 它将会被稀疏文件所支持，从磁盘的使用效率的观点来看它是不错的(因为就好像在精简池中的卷，它一开始是小的，只有当真的需要写的时候才会使用磁盘的存储块)但是从性能的角度来看就不那么好了，因为VFS增加了一些额外的负担，特别是"第一次写的时候"

Before checking how to resize a container, we will see how to make more room in that pool.

在我们看一下如何调整容器的大小之前，我们将要来试试看如何给池增加更多的空间

## We need a bigger pool
## 我们需要一个更大的池

`Warning`: the following will delete all your containers and all your images. Make sure that you backup any precious data!

`警告`: 下面的操作会删除你所有的容器和镜像，确保你已经把你的之前的数据做了备份!

Remember what we said above: Docker will create the data and metadata files if they don't exist. So the solution is pretty simple: just create the files for Docker, before starting it!

记住我们上面所说,当数据和元类信息文件不存在的时候Docker会创建它们。所以我们的解决方案非常简单:在我们启动它们之前，在Docker里创建这些文件!

1.Stop the Docker daemon, because we are going to reset the storage plugin, and if we remove files while it is running, Bad Things Will Happen©.

1.停止Docker守护进程，因为我们将要重新设置我们的存储插件，如果我们在运行的时候移除文件，那么糟糕的事情就将发生

2.Wipe out /var/lib/docker. Warning: as mentioned above, this will delete all your containers all all your images.

2.擦去`/var/lib/docker`.警告:正如前面提到的，这个会把你所有的容器和镜像都删除掉.

3.Create the storage directory: mkdir -p /var/lib/docker/devicemapper/devicemapper.

3.创建存储目录: `mkdir -p /var/lib/docker/devicemapper/devicemapper`.

4.Create your pool: dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=250 will create a sparse file of 250G. If you specify bs=1G count=250 (without the seek option) then it will create a normal file (instead of a sparse file).

4.创建你的池:`dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=250`,将会创建一个250G的稀疏文件。如果你指定`bs=1G count=250`(不使用`seek`选项)，那么它会创建一个普通文件(而不是一个稀疏文件).

5.Restart the Docker daemon. Note: by default, if you have AUFS support, Docker will use it; so if you want to enforce the use of the Device Mapper plugin, you should add -s devicemapper to the command-line flags of the daemon.

5.重启Docker守护进程. 提示:在默认情况下，如果你有AUFS的支持，Docker会使用它;所以如果你要强制使用Device Mapper的插件，你需要在启动Docker的命令中增加`-s devicemapper`的选项.

6.Check with docker info that Data Space Total reflects the correct amount.

6.使用`docker info`来检查`Data Space Total`是不是反映了正确的大小.

## We need a faster pool
## 我们需要一个更快的池

`Warning`: the following will also delete all your containers and images. Make sure you pull your important images to a registry, and save any important data you might have in your containers.

`警告`:下面的操作也会删除你所有的容器和镜像.确保把你重要的镜像保存在registry中，保存你容器里面重要的数据.

An easy way to get a faster pool is to use a real device instead of a file-backed loop device. The procedure is almost the same. Assuming that you have a completely empty hard disk, /dev/sdb, and that you want to use it entirely for container storage, you can do this:

要获得一个更快速的池，最简单的办法就是使用一个真实的设备而不是一个基于文件的循环设备。过程几乎是一样的。假设你有一个完全空的硬盘,`/dev/sdb`,你想把它完全用于容器的存储，你可以这样做:

1.Stop the Docker daemon.

1.停止Docker的守护进程

2.Wipe out /var/lib/docker. (That should sound familiar, right?)

2.移除`/var/lib/docker`,(似曾相识，对么?)

3.Create the storage directory: mkdir -p /var/lib/docker/devicemapper/devicemapper.

3.创建一个存储目录: `mkdir -p /var/lib/docker/devicemapper/devicemapper`

4.Create a data symbolic link in that directory, pointing to the device: ln -s /dev/sdb /var/lib/docker/devicemapper/devicemapper/data.

4。在那个目录下创建一个数据软链接，指向那个设备:`ln -s /dev/sdb /var/lib/docker/devicemapper/devicemapper/data`

5.Restart Docker.

5.重启Docker

6.Check with docker info that the Data Space Total value is correct.

6.使用``docker info`来检查`Data Space Total`的值是正确的

## Using RAID and LVM
## 使用RAID和LVM

If you want to consolidate multiple similar disks, you can use software RAID10. You will end up with a /dev/mdX device, and will link to that. Another very good option is to turn your disks (or RAID arrays) into LVM Physical Volumes, and then create two Logical Volumes, one for data, another for metadata. I don't have specific advices regarding the optimal size of the metadata pool; it looks like 1% of the data pool would be a good idea.

如果你希望合并多块相似的磁盘，你可以使用基于软RADID10.这个会以链接到`/dev/md`而高中。另外一个非常好的选项是把你的磁盘(或者RAID磁盘阵列)放到[LVM](http://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)的物理卷中，并且创建两个逻辑卷,一个是数据，一个是元数据。对于元数据池的最佳的大小没有什么特别的建议;看起来数据池的1%会是一个不错的选择.

Just like above, stop Docker, wipe out its data directory, then create symbolic links to the devices in /dev/mapper, and restart Docker.

就像前面一样，停止Docker，移除它的数据目录，然后创建一个指向`/dev/mapper`设备的符号链接，然后重启Docker

If you need to learn more about LVM, check the LVM howto.

如果你需要更多关于LVM的知识，看这里[LVM howto](http://tldp.org/HOWTO/LVM-HOWTO/)

## Growing containers
## 扩容容器


By default, if you use the Device Mapper storage plugin, all images and containers are created from an initial filesystem of 10 GB. Let's see how to get a bigger filesystem for a given container.

默认来说，如果你使用Device Mapper的存储插件，所有的镜像和容器是从一个初始10G的文件系统中创建的。让我们来看看如何从一个更大的文件系统中创建一个容器.

First, let's create our container from the Ubuntu image. We don't need to run anything in this container; we just need it (or rather, its associated filesystem) to exist. For demonstration purposes, we will run df in this container, to see the size of its root filesystem.

首先，我们用Ubuntu的镜像来创建我们的容器.我们不需要在这个容器里运行人和东西;我们只需要(换句话说，它的文件系统)存在.为了演示，我们会在这个容器里运行`df`,来看一下根文件系统的大小.

```
$ docker run -d ubuntu df -h /
4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
```

We now have to run some commands as root, because we will be affecting the volumes managed by the Device Mapper. In the instructions below, all the commands denoted with # have to run as root. You can run the other commands (starting with the $ prompt) as your regular user, as long as it can access the Docker socket, of course.

我们现在需要用root的身份来运行一些命令,因为我们将会修改Device Mapper管理中的一些卷的信息.在下面的指令，所有以＃开头的命令都必须以root身份来执行。你可以用普通用户的身份来执行其他的命令(以$开头)，当然它要能访问Docker的Socket服务

Let's look into /dev/mapper; there should be a symbolic link corresponding to this container's filesystem. It will be prefixed with docker-X:Y-Z-:

让我们看一下`/dev/mapper`,那里应该有一个对应容器文件系统的符号链接.它以`docker-X:Y-Z-`开头:

```
# ls -l /dev/mapper/docker-*-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
lrwxrwxrwx 1 root root 7 Jan 31 21:04 /dev/mapper/docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603 -> ../dm-8
```

Note that full name; we will need it. First, let's have a look at the current table for this volume:

注意那个全名，我们未来会需要.首先让我们来看以下当前的卷的信息表

```
# dmsetup table docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
0 20971520 thin 254:0 7
```
The second number is the size of the device, in 512-bytes sectors. The value above corresponds to 10 GB.

第二个数字是设备的大小,表示有多少个512－bytes的扇区. 这个值对应与10GB的大小

Let's compute how many sectors we need for a 42 GB volume:

我们来计算一下一个42GB的卷需要多少扇区

```
$ echo $((42*1024*1024*1024/512))
88080384
```

An amazing feature of the thin snapshot target is that it doesn't limit the size of the volumes. When you create it, a thin provisioned volume uses zero blocks, and as you start writing to those blocks, they are allocated from the common block pool. But you can start writing block 0, or block 1 billion: it doesn't matter to the thin snapshot target. The only thing determining the size of the filesystem is the Device Mapper table.

精简快照目标的一个神奇的特点是它不会限制卷的大小。当你创建它的时候，一个精简的卷使用0个块，当你开始往块里面写入的时候，它们会从共用的块池中进行分配。但是你可以写0个块，或者是10亿个块，这个和精简快照目标没关系。文件系统的大小只和Device Mapper表有关系

Confused? Don't worry. The TL,DR is: we just need to load a new table, which will be exactly the same as before, but with more sectors. Nothing else.

觉得困惑？不要担心.我们只是需要装载一个新的表，这个完全和之前的是一样的，但是有更多的扇区。仅此而已

The old table was 0 20971520 thin 254:0 7. We will change the second number, and be extremely careful about leaving everything else exactly as it is. Your volume will probably not be 7, so use the right values!

旧表是`0 20971520 thin 254:0 7`.我们会改变第二个数字，要绝对小心保持其他的值不变。你的卷可能不是`7`,所以要使用正确的值!

So let's do this:

让我们这样做：

```
# echo 0 88080384 thin 254:0 7 | dmsetup load docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
```

Now, if we check the table again, it will still be the same because the new table has to ba activated first, with the following command:

现在如果我们再次检查表的信息，这个仍然和之前是一样的，因为新的先使用下面的命令进行激活:

```
# dmsetup resume docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
```

After that command, check the table one more time, and it will have the new number of sectors.

执行完那个命令后，再次检查以下表的信息，发现它会使用新的扇区数量

We have resized the block device, but we still need to resize the filesystem. This is done with

我们已经调整了块设备的大小，但是我们仍然需要调整文件系统的大小，我们使用`resize2fs`来操作:

```
# resize2fs /dev/mapper/docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
resize2fs 1.42.5 (29-Jul-2012)
Filesystem at /dev/mapper/docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603 is mounted on /var/lib/docker/devicemapper/mnt/4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 3
The filesystem on /dev/mapper/docker-0:37-1471009-4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603 is now 11010048 blocks long
```

As an optional step, we will restart the container, to check that we indeed have the right amount of free space:

一个可选的步骤，我们会重启容器，检查一下我们的确有了正确大小的空闲空间:

```
$ docker start 4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
$ docker logs 4ab0bdde0a0dd663d35993e401055ee0a66c63892ba960680b3386938bda3603
df: Warning: cannot read table of mounted file systems: No such file or directory
Filesystem      Size  Used Avail Use% Mounted on
-               9.8G  164M  9.1G   2% /
df: Warning: cannot read table of mounted file systems: No such file or directory
Filesystem      Size  Used Avail Use% Mounted on
-                42G  172M   40G   1% /
```

Want to automate the whole process? Sure:

想把整个过程自动化起来？当然没问题:

```
CID=$(docker run -d ubuntu df -h /)
DEV=$(basename $(echo /dev/mapper/docker-*-$CID))
dmsetup table $DEV | sed "s/0 [0-9]* thin/0 $((42*1024*1024*1024/512)) thin/" | dmsetup load $DEV
dmsetup resume $DEV
resize2fs /dev/mapper/$DEV
docker start $CID
docker logs $CID
```

## Growing images
## 扩容镜像

Unfortunately, the current version of Docker won't let you grow an image as easily. You can grow the block device associated with an image, then create a new container from it, but the new container won't have the right size.

不幸运的是，当前版本的Docker不能够让我们很方便的扩容镜像。你可以把镜像对应的块设备进行扩容，然后从它来创建一个容器，但是新的容器不会有正确的大小

Likewise, if you commit a large container, the resulting image won't be bigger (this is due to the way that Docker will prepare the filesystem for this image).

同样，如果你提交了一个很大的容器，最后生成的镜像也不会很大(这个是因为Docker给这个镜像准备文件系统的方法造成的)

It means that currently, if a container is really more than 10 GB, you won't be able to commit it correctly without additional tricks.

这个意味着现在如果一个容器真的超过了10GB。在不使用一些其他的小技巧的情况下，你没法正确的把它提交为一个镜像


## Conclusions
## 总结

Docker will certainly expose nicer ways to grow containers, because the code changes required are very small. Managing the thin pool and its associated metadata is a bit more complex (since there are many different scenarios involved, and a potential data migration, that we did not cover here, since we wiped out everything when building the new pool), but the solutions that we described will take you pretty far already.

Docker将会自然的提供一些更好的方法来扩容容器，因为所需要的代码的变化是很小的。管理一个精简的池和它的对应的元信息比较复杂(因为这个需要很多不同的操作流程，以及一个潜在的数据迁移，这个我们这里没有提到，因为我们移除了所有的东西来构件新的池)，但是我们今天提到的一些解决方案相信已经对你有所帮助

As usual, if you have further questions or comments, don't hesitate to ping me on IRC (jpetazzo on Freenode) or Twitter (@jpetazzo)!


与往常一样，如果你有问题或者评论，不要犹豫，在IRC上ping我(jpetazzo on Freenode)或者是Twitter([@jpetazzo](https://twitter.com/jpetazzo))

