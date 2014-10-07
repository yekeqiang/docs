#在 Ubuntu 13.10 Saucy  Salamander 上使用 Docker and IPv6  

***


随着我对 Docker 的渐渐熟悉，我想把它用在最新发布的 Ubuntu 13.10 上。现在我仍然没有将 Docker 本地安装在我的个人电脑上，但是我经常用 Vagrant 去尝试最新的版本。在我下载了最新版的 Ubuntu 的镜像文件之后，我创建了一个 VirtualBox 的镜像并且安装了 Docker0.8.0。我试着将我的另一些小玩意儿，比如 CouchDB 和 Elasticsearch 在 Docker containers 上运行，不一会儿外设就再也没法儿用了。


在 Linux 13.04 和 13.10 版本上的安装文件提到可以通过修改 UFW中 DEFAULT_FORWARD_POLICY 配置来接受全部流量。不过 UFW 没法儿用，我照常在 StackOverflow 、 GitHub 还有开发者博客上寻找帮助。多少人提到无法通过IPv6设置系统核心参数，不过一个叫做 Jérôme Petazzoni 的开发者在 StackOverflow 上的回答让我找到了一个解决方案。


卸载 IPv6 后一些人发现能使其工作的内在组件。 Andreas Neuhaus 告诉我们一些能使容器支持 Ipv6 的原始 LXC 命令。 Marek Goldmann 也告诉我们另外一些在多个主机上使容器相连的 LXC 命令。希望我的 pullrequest  能够让 Docker 尽快支持 Ipv6 ，尽管我的问题看起来并不像是丢失了支持 Ipv6 的内部组件。

程序调试，桥式接线，ip信息包过滤系统以及接口安装，然后再回头查看系统配置条目。你或许会惊讶每一个非法进入，甚至看起来合法的进入都会被记录：我允许 Ipv6 使用所有接口用 net.ipv6.conf.all.forwarding=1 在 /etc/sysctl.conf 。快速启动很快完成，并通 过dockerized CouchDB 反馈回来。终于轻松了！有时事情还是很简单的。

最有趣的部分到了，通过容器再次连接 CouchDB River Plugin 和 ElasticSearch 。

