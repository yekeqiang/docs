#使用 Docker 构建 Komodo

![](http://resource.docker.cn/komodo-x11-ssh.png)


#####作者：[Nathan Rijksen](https://twitter.com/NathanRijksen)

#####译者：[Fiona Feng](https://twitter.com/usuede)

***



今天又是周一，也就是“周一宏”博客更新的日子。由于忙着构建 Komodo ，我这次没有准备关于宏的文章，不过有篇替代文章：使 Komodo 构建运行，以及我是如何无意当中让 Komodo 的构建变得更简单的。很抱歉这次没有宏的文章，我保证下周会写！


###故事背景

上星期五我确认自己已经受够了 Ubuntu ，厌倦了等待合适的存储卡提供需要的软件，以及安装一些软件后存储库就各种混乱。在尝试失败后，我决定安装 Arch Linux 并且完全替换。同时我也不想在设置上浪费时间，于是选择安装了 Antergos ，它基本上也是 Arch ，不过有一个相当酷炫的安装器。

每件事都有条不紊。 Arch/Antergos 平稳运行着我所有的应用程序，并且设置也可以快速切换，剩下的就是让 Komodo 的构建运行。我就是完成这个苦差事的人。

我会省略很多细节。不过 Arch 默认使用 Python 3，当我使用 Python 2 时就遇到了一大堆 unicode 的连接问题。我用了两天多来修改脚本，让 Komodo 构建运行，不过在 99% 的时候卡住了。无奈之下，我在 IRC 大吐苦水，表示多希望自己能在 Docker 里跑 GUI 应用程序；这时 Mark Yen （ Komodo 的开发者之一）简单说一句“你可以用 SSH 伴随着 X11 转发”。

我知道 X11 转发，不过总认为它功能有限，只能适用于最基本的 X11 窗口，而不是像 Komodo 这样的复杂应用。我错了！我马上将折磨我两天的问题抛之脑后，一头扎进了 Docker 。差不多一天后我就可以通过 Docker 运行功能齐全的 Komodo 构建了。太美妙了！

在对 Docker 镜像的执行进行优化后，是时候与大家分享了。这能让你获得自己的 Linux Komodo 构建并且无需担心安装依赖就可以运行了。

###使用 Docker 来构建 Komodo

在用 Docker 构建 Komodo 之前请先到我们的 [GitHub](https://github.com/Komodo/KomodoEdit) 页面了解 [操作指南](https://github.com/Komodo/KomodoEdit#building-with-docker) 。

要想在 Komodo 内运行构建，你需要在终端（必须安装并运行 Docker ）使用下列命令：

```
cd <project-root>/util/docker
sudo ./docklet build # takes a few minutes
sudo ./docklet start
sudo ./docklet ssh # you will now be sshed into the docker container
cd /root/komodo/mozilla
python build.py configure -k 9.10
python build.py distclean all # this will take a while, between 15 mins and xx hours depending on your hardware
cd ..
bk configure -V 9.10.0-devel
bk build # takes a few minutes
bk run
```

了解更多细节请查看 [安装指南](https://github.com/Komodo/KomodoEdit/blob/trunk/BUILD.txt) 。

一旦获得构建运行，大多数时间只需最后两条命令，也不会占用很长时间。


###进一步使用Docker

一旦你的 Docker 构建能够运行，你可能会想“现在怎么办？我去哪里编辑文档？” 感谢 Docker 让这一切变得简单，在 Docker 容器之上加载一个宿主文件夹。这个文件夹在使用上文提及的 docklet 脚本的同时生成。一旦需要运行构建，你就可以用喜欢的任何方式在宿主机的项目根目录下，方便地编辑文件。（我听说 Komodo IDE 非常精于此道。） :)


###友情提示

请注意： Docker 脚本是全新的东西，在成为能够直接在宿主机上运行构建的可靠的解决方案之前，还需要多加打磨。


###最后的思考

我要感谢 [pelle.io](http://pelle.io/) 给我提供了需要的资源，帮助我解决 docker 相关问题。他的博客和脚本给了我巨大帮助。

我希望能在将来进一步改进现有的 Docker 脚本，不过这取决于有多少人使用。我特别想通过提供构建好的 mozilla 依赖关系来简化这个构建过程，预计能减少一半的上述步骤。

如果你觉得这个脚本有用并且愿意使用，请告诉我们。

我希望将来会有更有趣的东西和大家分享！ :)

***

#####这篇文章由 [Nathan Rijksen](https://twitter.com/NathanRijksen) 撰写，[Fiona Feng](https://twitter.com/usuede) 翻译。点击 [这里](http://komodoide.com/blog/2014-07/building-komodo-with-docker/) 阅读原文。

#####The article was contributed by [Nathan Rijksen](https://twitter.com/NathanRijksen) , click [here](http://komodoide.com/blog/2014-07/building-komodo-with-docker/) to read the original publication.
