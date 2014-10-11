#编译 CyanogenMod 的 Docker 模板



编译 [CyanogenMod](http://www.cyanogenmod.org) 需要很多的工作。你需要安装大量的依赖包，你还需要阅读很多文档。[Docker](http://docker.io) 是一个在容器自动内部署应用的软件。



这个 Docker 容器已经包含了编译 CyanogenMod 所依赖的所有环境。 它非常容易安装，装好即用。 Github 项目页面有更多如何使用安装使用的信息。



注意：你需要根据 Docker 的文档先安装 Docker ：[https://www.docker.io/gettingstarted/](https://www.docker.io/gettingstarted/)

##编译：


```
git clone https://github.com/stucki/docker-cyanogenmod.git
cd docker-cyanogenmod
./build.sh
```


##运行：



```
cd docker-cyanogenmod
./run.sh
```



##为你的设备编译 CyanogenMode：


```
repo init -u git://github.com/CyanogenMod/android.git -b cm-11.0
repo sync
vendor/cm/get-prebuilts
source build/envsetup.sh
breakfast <device codename>   # example: breakfast grouper
brunch <device codename>      # example: brunch grouper
```


##下载：

Github URL: https://github.com/stucki/docker-cyanogenmod

Github 库地址：[https://github.com/stucki/docker-cyanogenmod](https://github.com/stucki/docker-cyanogenmod)



##ChangeLog:



```
2014-02-20

* Add note about running get-prebuilts

2014-02-16

* Initial release
```

***


欢迎提交反馈。

---

#####这篇文章由 [michael_ch](http://forum.xda-developers.com/member.php?u=2113874) 发表，点击 [此处](http://forum.xda-developers.com/showthread.php?t=2650345) 可查阅原文。

#####The article was contributed by [michael_ch](http://forum.xda-developers.com/member.php?u=2113874), click [here](http://forum.xda-developers.com/showthread.php?t=2650345) to read the original publication. 
