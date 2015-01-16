# 透过数字看 Docker Hub


##### 作者：[Kiyoto Tamura](https://twitter.com/kiyototamura)

##### 译者：[BOWEN](https://github.com/iambowen)

***
上个周六我参加了 [DockerCon Hackathon](http://www.meetup.com/Docker-meetups/events/170030222/) 活动。本人是一名 Docker 深度用户（主要是快速创建新容器，去验证或者重现 [Fluentd](http://www.fluentd.org/) 项目提交的问题）。

毫无疑问，Docker 很棒，这个礼拜海量公告证明了它的受欢迎和关心的程度。不过 Docker （或者其他通常意义上的的开源软件）真正吸引我的地方还是社区。


在编程马拉松活动中，我借着自己“前数据科学家”（解读：定量金融中的最小量子）的头衔，收集了 [Docker Hub](http://hub.docker.io/) （ 一周前还叫 Docker Index-_-!) 的数据，在 Excel 上绘制了一些不那么"性感"的图表。

>注: 数据收集于2014年6月9号，我尽力了。


## 数据集: 大部分的工作！

我用了Docker提供的 [API](http://docs.docker.com/reference/api/docker-io_api/) 发送一系列的 API 请求去获取代码库和镜像的元数据。比如，为了扒代码库名字，我使用了搜索 API 并且查找了所有名字长度小于等于3的代码库。为了加速搜集的过程，我创建了几个 EC2 节点，然后在节点上运行脚本（你可以在 [这里](https://github.com/kiyoto/docker-hackathon) 看到我的成果）。

对，这就是让人觉得搞笑的地方。这是 Docker 编程马拉松，而我却在用 EC2 节点。

## 数据


### 13,475 ：Docker Hub 上（我能找到的）代码库的数量

Docker 刚刚 [宣布](https://www.dockboard.org/announcing-docker-hub-and-official-repositories/) 它们已经拥有超过14,000个代码库，因此我的数据已经很接近了……或者是他们把数据取整了^_^ 。

下面这幅图展示了每月创建的代码库的数量：

![alt](http://resource.docker.cn/repos-created-per-month.png)

(六月份数据比较少，毕竟这个月才开始)

![alt](http://resource.docker.cn/m2m-growth.png)

代码库数量每月的中位数增长率为 43.99% 。变化很大，因此我们也无法估量未来的走势。但是如果它能保持每月 40% 的净增长率，到2015年9月份，代码库总量会达到 100 万。（参考 Github ，在运营3年后，他们 [在2011年拥有了200万个代码库](https://github.com/blog/841-those-are-some-big-numbers) ）

### 5,091: Docker Hub 的用户数量

这是我估测的代码库拥有者的数量。这意味着每个用户有13,475/5091 = 2.64 个代码库。中位数是1，人数的标准差是3.96 。

下面是按照代码库数量（或者 Docker 化应用）排名前20的代码库用户的柱状图:

![alt](http://resource.docker.cn/top-20-repo-owners.png)


### 101,477: Docker Hub 上镜像总数

诚然，很多的这些镜像并不是代码库的“顶端”。我通过它们优秀的 Ancestry API 去重新构建了大部分的层次树。

举例说吧，我能找到的从根到叶子的最深路径是 afrantisak/ska ，有120层父镜像。

下面是排名前20“最深”代码库：

![alt](http://resource.docker.cn/top-20-deepest-repos.png)

## 将来的工作

一些将来我想要做的事情：

1. Dockerfile 相似度计算：基于我的经验，我们最终会经常拷贝修改现有的 Dockerfile 。此外，由于很多 Dockerfile 中的命令都是顺序运行，很多 Dockerfile 看上去一定是很相似的。从数学角度来看，如果我们将每个 Dockerfile 看成是单独的线构成的有限序列，并且一个 Dockerfile 是从另外一个继承而来，它们共用一个大的子序列就变得很有可能。这样也许会很有趣，定义两个 Dockerfile 的非格律距离，将其作为最大子序列的长度，通过所有的 Dockerfile 以集群的方式运行在 Docker Hub 上。

2. 基于 From 继承创建一个 Dockerfile 树：这个点子来 自[@ewindisch](http://github.com/ewindisch) 。目前，可信的版本（托管在 Github 上的 Dockerfile ）可以从一个不可信的版本继承，因为不要求可信版本必须从一个可信的版本构建。看到有多少可信的版本有正真可信的血统是一件很有趣的事情。


## 感谢 Docker 公司

最后，但也是最重要的， Docker 带给了我很多的乐趣。作为一名开发人员营销人员，编码没有以前多了，而且明显的感觉越来越生疏了。但是能够再次挽起袖子，和全球各地的开发人员一起探索发现新事物（我实现了自己第一份 Go 代码，学习了很多关于 Docker 的 API 的知识）的感觉很棒。因此，感谢每一个参与到 Docker 编程马拉松活动的每个人 :) 。

***

##### 这篇文章由 [Kiyoto Tamura](https://twitter.com/kiyototamura) 撰写， [BOWEN](https://github.com/iambowen) 翻译。点击 [这里](http://kiyototamura.tumblr.com/post/88454980152/docker-hub-by-the-numbers-my-docker-hackathon) 阅读原文。

##### The article was contributed by [Kiyoto Tamura](https://twitter.com/kiyototamura), click [here](http://kiyototamura.tumblr.com/post/88454980152/docker-hub-by-the-numbers-my-docker-hackathon) to read the original publication.
