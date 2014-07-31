
##Docker教程系列，第九部: 镜像的10个Docker远程API命令

>注:作者[flux7](http://www.flux7.com/), 原文发于[flux7](http://blog.flux7.com/blogs/docker/docker-tutorial-series-part-9-10-docker-remote-api-commands-for-images)

在本系列的上一篇文章中，我们讨论了[Docker远程API](http://flux7.com/blogs/docker/docker-tutorial-series-part-8-docker-remote-api/)，尝试了用于容器的命令。本文将着重介绍镜像相关的命令
####创建一个镜像

镜像可以用如下之一的方式来创建:

- 通过从registry pull的方式
- 通过导入镜像的方式

```
POST /images/create
```

下面的截图是一个示例。
![create sample](http://cdn2.hubspot.net/hub/411552/file-1222274949-jpg/blog-files/create-an-image.jpg?t=1406803574300)

####从容器创建镜像

从容器的提交创建一个镜像，使用:

```
POST /commit
```
下面的截图是一个示例。
![commit sample](http://cdn2.hubspot.net/hub/411552/file-1222274964-png/blog-files/docker-create-image-from-container.png?t=1406803574300)

####镜像的列表

为了获取镜像的列表，使用:
```
GET /images/json
```
下面的截图是一个示例。
![list sample](http://cdn2.hubspot.net/hub/411552/file-1222274979-png/blog-files/docker-list-images.png?t=1406803574300)

####插入一个文件

To insert a file at a specific path, use:
往特定的路径插入一个文件，使用:

```
POST /images/(name)/insert
```
下面的截图是一个示例。
![insert sample](http://cdn2.hubspot.net/hub/411552/file-1222274994-jpg/blog-files/docker-image-insert-file.jpg?t=1406803574300)

####删除镜像


按照名字删除镜像，使用:

```
DELETE /images/(name)
```
下面的截图是一个示例。
![delete sample](http://cdn2.hubspot.net/hub/411552/file-1222275009-jpg/blog-files/delete-an-image.jpg?t=1406803574300)

####提交到Registry

往registry提交镜像，使用:

```
POST /images/(name)/push
```
下面的截图是一个执行的示例。
![push sample](http://cdn2.hubspot.net/hub/411552/file-1222275024-png/blog-files/docker-push-image-to-remote-repo.png?t=1406803574300)

####标注镜像

要给镜像做标注，使用:

```
POST /images/(name)/tag
```
下面的截图是一个执行的示例。
![tag sample](http://cdn2.hubspot.net/hub/411552/file-1222275039-jpg/blog-files/tag-an-image.jpg?t=1406803574300)

####搜索一个镜像

要搜索一个镜像，使用:

```
GET /images/search
```
下面的截图是一个执行的示例。
![search sample](http://cdn2.hubspot.net/hub/411552/file-1222275054-png/blog-files/docker-search-an-image.png?t=1406803574300)

####历史

查看镜像的历史，使用:

```
GET /images/(name)/history
```
下面的截图是一个执行的示例。
![history sample](http://cdn2.hubspot.net/hub/411552/file-1222275069-jpg/blog-files/docker-get-image-history.jpg?t=1406803574300)

####构建一个镜像

可以用Dockerfile来构建一个镜像:

```
POST /build
```
下面的截图是一个执行的示例。
![build sample](http://cdn2.hubspot.net/hub/411552/file-1222275084-png/blog-files/docker-build-image-from-dockerfile.png?t=1406803574300)

我们现在完成了Docker API的三条腿的旅程。每个礼拜四，你都会在这里看到Docker教程系列的下一篇文章。

同时，来探索发现今天的[Flux7的Docker评估程序](http://flux7.com/docker-assessment-package/)。这个程序是用来回顾和解如何用Docker来优化程序员的工作流。了解更多关于评估和使用Docker的内容，只需要询问[info@flux7.com](mailto:info@flux7.com)或者点击[这里](http://flux7.com/docker-assessment-package/)。亦或者，直接访问我们的[flux7.com/docker-solution/](flux7.com/docker-solution/)。
