# Docker 中文社区

[Docker 中文社区](http://www.dockboard.org) 是一个开放的开源社区，一方面在 [网站](http://www.dockboard.org) 组织 **Docker** 技术文章的翻译和原创，另一方面在各地组织 Meetup 活动及其它社区的技术宣讲。关于 **Docker** 技术的问答请访问 [Docker 中文技术社区](http://www.dockboard.org) 和 [SegmentFault](http://segmentfault.com) 合作的 [问答子站](http://segmentfault.com/docker) 。

## 如何贡献翻译和原创文章

[网站](http://www.dockboard.org) 负责发布 **Docker** 技术文章的翻译和原创内容， 发布 Meetup 活动信息。

### 如何进行文章的翻译协作

我们使用 [Github](http://github.com) 进行翻译和编写的协作，任何人都可以 [fork](https://help.github.com/articles/fork-a-repo) 我们的文章 [仓库](https://github.com/dockboard/docs) ，进行文章编写和翻译。最后向我们提交 [pull request](https://help.github.com/articles/creating-a-pull-request) 。内容管理团队的成员会根据您翻译的情况选择是否 [merge](https://help.github.com/articles/merging-a-pull-request) 。

如果您想翻译某篇文章，先和我们的 [微博官方账号](http://weibo.com/dockboard) 或 [fengzhao116 AT gmail.com](fengzhao116@gmail.com) 联系协调。如果是原创文章直接提交 [pull request](https://help.github.com/articles/creating-a-pull-request) 即可。一旦我们 [merge](https://help.github.com/articles/merging-a-pull-request) 了您的 [pull request](https://help.github.com/articles/creating-a-pull-request) ，我们就会对文章进行两次以上的校对；如果对文章有不同理解，我们会通过多重方式（ 更多是 Email 、Comments 等 ）与您联络讨论翻译的是否合理，力求文章的 **信达雅**。

当我们得到了满意的文章版本时，内容管理团队的成员会将文章发布到 [网站](http://www.dockboard.org) 并在社交媒体进行推广。

### 文章翻译的格式要求

* 文章必须使用 [Markdown](https://help.github.com/articles/markdown-basics) 作为文章格式
* 文件名使用 [Permlink](http://codex.wordpress.org/Using_Permalinks) 模式，只能使用中划线分隔英文单词和数字，文件后缀名是 **.md**
* 正文中的所有数字和字母两边都必须有一个空格；单词首字母不许大写，句子以英文单词开头除外
* 代码不能使用 Tab 缩进，在代码的上一行和下一行使用 `````` 符号
* 原文的链接在译文中保持
* 译者注的单独段落，段首使用 > 符号开头
* 文章发布时需要在翻译的最后加入版权声明
* 文章发布时需要在版权声明后加入译者的信息及联系方式

## 如何参与 Docker 中文社区的活动

### 加入并组织 Meetup

我们使用 [Meetup](http://meetup.com) 作为线下 Meetup 活动组织和发布的工具。如果您有意在本地组织 Meetup 活动或参与 Meetup 的组织可以 [Email](mailto://fengzhao116@gmail.com) 与我们联系，确认后您所在城市将出现在 [Meetup](http://meetup.com) 中。

目前我们已经有两个 Meetup 群组：[Docker Beijing](http://www.meetup.com/Docker-Beijing/) 和 [Docker Shanghai](http://www.meetup.com/Docker-Shanghai/) ，我们欢迎更多的城市和开发者加入这个充满活力的生态系统。

> 如果您长期坚持参与翻译和原创文章，并且参与我们的线下 Docker Meetup 活动，我们会邀请您加入到社区管理团队中，和我们一起共建这个社区。

## 如何参与 Docker 中文社区的开源项目

[Docker 中文社区](http://www.dockboard.org) 有若干个关于 **Docker** 的开源项目，如果您对这些项目感兴趣可以通过 [pull request](https://help.github.com/articles/creating-a-pull-request) 的形式参与，或者通过 **Issue** 同开发成员讨论 **Feature** 或者提交 **Bug**。

### Docker Registry

[Docker Registry](https://github.com/dockboard/docker-registry) 是 [Golang](http://golang.org) 版本的 [docker-registry](https://github.com/dotcloud/docker-registry)。[Docker Registry](https://github.com/dockboard/docker-registry) 使用 [Golang](http://golang.org) 语言开发可以减少部署的依赖，使用 [七牛](http://qiniu.com) 和 [阿里开放存储服务](http://www.aliyun.com/product/oss) 存储镜像可以提高在国内的访问速度。

### Docker Proxy
[Docker Proxy](https://github.com/dockboard/docker-proxy) 是为 **Docker** 从 [index.docker.io](http://index.docker.io) 下载开发的代理程序。使用 [Golang](http://golang.org) 语言，Proxy 部分使用 [Goproxy Library](https://github.com/elazarl/goproxy) 。

### Docker Dashboard
[Docker Dockboard](https://github.com/dockboard/dockboard) 是一个 Docker 的 Dashboard，为 Docker Daemon 提供一个 UI 界面。

> 非常希望 Contributor 加入我们的全职团队，有意请联系 [Meaglith Ma](http://weibo.com/genedna)。
