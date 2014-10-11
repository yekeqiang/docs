# Docker vs LXC

##### 作者：[Joe McWilliams](https://twitter.com/doughyjoey5)

##### 译者：[铁威](http://weibo.com/2392252662)

---

我关注 docker 有一段时间了，最近开始讨论在公司使用，因为小伙伴们想使用 LXC ，也就是 docker 背后的那个技术。 以下是我研究 docker 和 LXC 后总结的一些区别。

## 标准的配置方法

每个 LXC "容器" 之间或许不兼容，但是 docker 采用了一种标准的配置方法使得由不同 docker 创建出的 LXC 能够完全兼容。

## 基于应用

LXC 的定位是作为一种虚拟机的替代方案。虽然所有的软件都可以安装在由 LXC 或者 docker 管理的容器中， 
但是 docker 更倾向于在一个容器中运行一个应用。

## 自动构建

Docker 的容器是根据 dockerfile 构建的，你可以在构建 image 的过程根据需要中运行任何命令和程序。 
这意味着你不用调整现有的 image 构建方式，如果你使用 puppet，你可以在生成容器的时候执行 puppet 命令。

## 版本控制

Docker 实现了类似 git 的容器版本管理方法，并且能够进行增量更新。

## 组件复用

可以创建 base image 并将其保存在远程仓库 (repository) 中以便复用，其他容器可以在其基础上进行修改并保存为新的 image。

## 远程仓库

Docker 管理着一个公开的 image 库方便用户分享 image ，同时公司也可以构造自己私有的 image 库。

## 生态系统

因为 docker 越来越流行，有大量的方法能够将其轻易地集成到开发过程中，比如可以采用统一的方法来构造用于持续集成的环境和开发环境的容器。

目前有不少反对者声称 docker 还是一个非常年轻的项目，并不能用于生产环境。但 docker 只是将一些已有的技术进行封装和组合并增加其业务逻辑，来避免大家直接使用 LXC 需要面对的一些麻烦。我相信如果直接使用 LXC ，会最终做出和 docker 功能类似的工具，然而不会得到已经转向 docker 的开发者社区的青睐。

---
##### 这篇文章由 [Joe McWilliams](https://twitter.com/doughyjoey5) 发表，[铁威](http://weibo.com/2392252662) 翻译。你可以点击 [这里](http://joemcwilliams.com/post/81189071262/docker) 阅读原文。

##### The article was contributed by [Joe McWilliams](https://twitter.com/doughyjoey5), click [here](http://joemcwilliams.com/post/81189071262/docker) to read the original publication.
