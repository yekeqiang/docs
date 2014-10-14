# 使用 Docker 快速自动化 Rackspace 云数据库

##### 作者：[Adam Alexander](https://twitter.com/adamalex)
##### 译者：[Peter Zhang](https://github.com/duobei)


***
[ Rackspace 云数据库](http://www.rackspace.com/cloud/databases/) 使用了 OpenStack 的 DBaaS 组件 [ Trove ](https://wiki.openstack.org/wiki/Trove)，并且通过 [Trove API](http://docs.openstack.org/developer/trove/) 使其自动化。

为了使命令行易于使用，此 Docker 镜像封装了 [python-troveclient](https://pypi.python.org/pypi/python-troveclient) 。

## 示例文件

* 为了更简单的使用所有功能， `Dockerfile` 定义了一个最小限度的 Docker 镜像，将 [python-troveclient](https://pypi.python.org/pypi/python-troveclient) 暴露作为镜像 [ENTRYPOINT](http://docs.docker.com/reference/builder/#entrypoint) 。此镜像通过 Docker 实现这个 API 客户端的所有功能，甚至无需使用 Python 或不兼容的 Python 安装。


* `trove.sh` 介绍了怎样配置和运行这个镜像。传入这个脚本的命令行参数通过 Docker 传进 [python-troveclient](https://pypi.python.org/pypi/python-troveclient) ，通过运行 `trove.sh help` 可以获得所有可用的 API 操作。

## 配置

想了解更多配置 Trove 客户端变量的细节，请参阅这个 [文档]( http://docs.rackspace.com/cdb/api/v1.0/cdb-getting-started/content/Install_Trove_Client.html) 。

## 用例

```
bash
# pull 预先配置的镜像，这样你第一次运行时可以快速很多
$ docker pull adamalex/trove:1.0.5

# 复制此 git 仓库
$ git clone https://github.com/adamalex/docker-trove.git

# 配置例子脚本 -- 配置变量的文档在 http://docs.rackspace.com/cdb/api/v1.0/cdb-getting-started/content/Install_Trove_Client.html。
$ cd docker-trove && vi trove.sh

# 获得使用帮助
$ ./trove.sh help

# 备份一个云数据库
$ ./trove.sh backup-create <backup-name> <cloud-db-uuid>
```

## 联系方式

如果有问题或建议，请联系 [Twitter](https://twitter.com/adamalex) 或 [开启一个 issue](https://github.com/adamalex/docker-trove/issues) 。

***

##### 这篇文章由 [Adam Alexander](https://twitter.com/adamalex) 撰写， [Peter Zhang](https://github.com/duobei) 翻译。点击 [这里](https://github.com/adamalex/docker-trove) 阅读原文。

##### The article was contributed by [Adam Alexander](https://twitter.com/adamalex), click [here](https://github.com/adamalex/docker-trove) to read the original publication. 

