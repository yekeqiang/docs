#Windows 下 Docker 安装教程


>作者注：你的CPU一定要支持VT才可以，笔者的笔记本T6400不支持VT是装不上的，所以一定要支持VT，好在笔者的MAC很OK，公司电脑也给力。


上一节我们介绍了在 ubuntu 和 centos 下的安装，当然都是基于64位系统的，在学习过程中，你可能没有这些东西，当然你可以用 virtualbox 或者 Vmware 虚拟化出来，今天我们介绍的是官网给我们提供的 using vagrant 方案

通过使用 virtualbox 这样的 VM ，Docker 可以在 Windows 上运行，然后你也可以在 VM 里运行 Linux 。

##安装警告

1. 官方申明 docker 还是在开发完善中，不建议在运营的产品中使用。请关注 [官方博客](http://blog.docker.io/2013/08/getting-to-docker-1-0/) 。

2. windows 的安装是社区贡献出来的，唯一的官方认可的的安装方法使用是 [ubuntu](http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-linux) 。不过鉴于版本更新速度，本文提及的版本可能过期。

##安装前准备

步骤1. 安装 virtualbox ，下载地址：https://www.virtualbox.org

*译者注：如果你不会安装或者感觉下载速度慢，可以用360或者QQ软件管家下载自动安装*

步骤2. 安装 vagrant ，下载地址：http://www.vagrantup.com

选择安装路径一路next就可以了

步骤3. 下载安装带有 ssh 的 git ，下载地址：http://git-scm.com/downloads

这个其实也是一路 next（向 github 提交过代码的应该最清楚）

**官方推荐至少有2GB的磁盘空间和2GB的内存！**

###运行命令提示符

首先要打开 cmd 命令提示符，你可以同时按住 windows 键(非官方备注：ctrl键旁边那个微软图标)+R，然后输入 cmd ,按回车（Enter）就可以了，当然你也可以在你的计算机中搜索cmd.exe

*译者注：如果你跟我一样用 win8 ，可以 "windows键 + x" 选择命令提示符管理员那个*

当然你可以用Cygwin终端或者git bash这些命令行都可以，操作都是一样的

###安装 Ubuntu virtual server

让我们下载和运行一个安装好 docker 文件的 ubuntu 镜像

```
git clone https://github.com/dotcloud/docker.git
cd docker
vagrant up
```

![alt](http://resource.docker.cn/installation-windows-1.png)

![alt](http://resource.docker.cn/installation-windows-2.png)

![alt](http://resource.docker.cn/installation-windows-3.png)

###官方文档没有提到但是你会遇到的情况

更新内核完成后，就出现一些字段，譬如升级完内核可能出现  vagrant halt 的字样，这个时候你就要输入 vagrant halt ，然后再输入 vagrant up ，可能会出现如下结果：

![alt](http://resource.docker.cn/installation-windows-4.png)

这个时候你就要输入 `vagrant provision` 然后会检测继续更新安装，然后再 `vagrant ssh` 就可以了

*这里你要稍等比较长的时间，去打个游戏玩会吧！因为它会下载很多东西，而且我们访问美国的网速一般都比较慢，所以我建议你还是先干点别的！*

我发现我安装的场景跟官方提供的显示一点都不一样，不过安装好了之后是一样的，我就拿实际的给大家看！

![alt](http://resource.docker.cn/installation-windows-5.png)

出现上边的截图后，输入 `vagrant halt` ，然后输出 `vagrant up` 来开启机器，当然你可以在 virtualbox 里边关闭它！

![alt](http://resource.docker.cn/installation-windows-6.png)

现在你可以庆祝了，你正在运行着装好 docker 的 unbuntu 服务器了，但是你看不到它，因为它一直在后台运行（非官方备注：可以从 virtualbox 中看到它，如下图）。

![alt](http://resource.docker.cn/installation-windows-7.png)


##登录 unbuntu 服务器

现在登录你的ubuntu服务器，你现在有两个选择

- 运用vagrant的命令行来操作
- 运用ssh（我用的putty）

###运用windows命令行来操作

`vagrant ssh`

这个时候你可能看到错误信息 “ **ssh executable not found** ”，产生错误的原因是你的 ssh 没有加入到可执行 PATH 路径中。这个时候，你可以用 set 命令来添加路径，譬如你的 ssh.exe 在你的 “ C:Program Files (x86)Gitbin ” 这个目录中，你就只要输入命令
`set PATH=%PATH%;C:\Program Files (x86)\Git\bin`
就OK了。

`vagrant ssh` 登录之后是这样的:

![alt](http://resource.docker.cn/installation-windows-8.png)



如果这个时候你出现错误 “ *The program ‘docker’ is currently not installed* ”，那就很遗憾你只能从头开始重新安装了。

###运用 ssh 客户端登录

首先，你要拿到你登录的IP和端口，输入

`vagrant ssh-config`

这个时候,你会看到输出了 hostname 就是你登录的 ip ，端口号 `2222` ，用户默认的 vagrant ，密码一样都是 vagrant ，然后你就可以用 ssh 登录了。我用的是 putty ，官方用的也是 putty ssh 登录。

![alt](http://resource.docker.cn/installation-windows-9.png)

![alt](http://resource.docker.cn/installation-windows-10.png)

当然如果你用 git bash 这种终端运行的时候也可以输入命令，然后账号密码也都是 vagrant 。

`ssh vagrant@127.0.0.1 –p 2222`


##运行docker

首先获得 root

`sudo su`

这个时候你就可以运行 **demo hello word** 了

![alt](http://resource.docker.cn/installation-windows-11.png)

下边是我用 virtualbox 安装 ubuntu 然后用 ubuntu 安装的 docker

![alt](http://resource.docker.cn/installation-windows-12.png)

---
#####这篇文章由 [widuu](www.weibo.com/widuu) 结合 Docker 官方 [教程](http://docs.docker.io/en/latest/installation/windows/) 撰写而成，最早发表于 [微度网络](http://www.widuu.com/docker/docker-windows.html) 。Docker 中文社区获得作者许可转载。您也可以点击 [这里](http://docs.docker.io/en/latest/installation/windows/) 阅读英文原版教程。
