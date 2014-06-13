# Docker Hub by the Numbers (我的Docker编程马拉松) #
>注: 原文地址[Docker Hub by the Numbers (My Docker Hackathon)](http://kiyototamura.tumblr.com/post/88454980152/docker-hub-by-the-numbers-my-docker-hackathon), 作者Kiyoto Tamura


上个周六我参加了DockerCon的编程马拉松活动。本人是一个Docker的深度用户(主要是为了快速建立新的容器，去验证或者重现[Fluentd](http://www.fluentd.org/)项目提交的问题)。

毫无疑问，Docker很棒，这个礼拜海量公告证明了它的受欢迎和关心的程度。但是Docker(或者其他通常意义上的的开源软件)真正吸引我的地方是社区。


在编程马拉松活动中，我带上了自己“前数据科学家”(解读:定量金融中的卑微的定量)的帽子，收集了[Docker Hub](http://hub.docker.io/)(一周前还叫Docker Index-_-!)的数据, 在Excel上绘制了一些不那么"性感"的图表。


注意: 数据收集于2014年6月9号，我尽力了。


## 数据集: 大部分的工作！

我用了Docker提供的[API](http://docs.docker.com/reference/api/docker-io_api/)发送一系列的API请求去获取代码库和镜像的元数据。比如，为了扒代码库名字，我使用了搜索API并且查找了所有名字长度小于等于3的代码库。为了加速搜集的过程，我创建了几个EC2节点，然后在节点上运行脚本(你可以在[这里](https://github.com/kiyoto/docker-hackathon)看到我的成果)。

对，这就是让人觉得搞笑的地方。这是Docker编程马拉松，而我却在用EC2节点。

##数据


## 13,475: Docker Hub上(我能找到的)代码库的数量

Docker刚刚[宣布](http://blog.docker.com/2014/06/announcing-docker-hub-and-official-repositories/)它们已经拥有超过14,000个代码库，因此我的数据已经很接近了……或者是他们把数据取整了^_^。

下面这幅图展示了每月创建的代码库的数量:

![ ](http://media.tumblr.com/0e7028f5587ec708b8ca203b55d3867c/tumblr_inline_n6yqtg2hmD1sq4ex2.png)

(六月份数据比较少，毕竟这个月才开始)

![ ](http://media.tumblr.com/9224e8f730a8cf4ca9a9814c6ccee657/tumblr_inline_n6yrdvzgB21sq4ex2.png)

代码库数量每月的中位数增长率为43.99%。变化依旧很大，因此我们也无法估量未来的走势。但是如果它能保持每月40%的净增长率，到2015年9月份，代码库总量会达到100万。(参考Github，在运营3年后，他们[在2011年拥有了200万个代码库](https://github.com/blog/841-those-are-some-big-numbers)。)

##5,091: Docker Hub的用户数量

这是我估测的代码库拥有者的数量。这意味着每个用户有13,475/5091 = 2.64 个代码库。中位数是1，人数的标准差是3.96.

下面是按照代码库数量(或者Docker化的应用)排名前20的代码库用户的柱状图:

![ ](http://media.tumblr.com/56eff48bd4f12291cbe1332a700c949b/tumblr_inline_n6zsbxbJp71sq4ex2.png)


##101,477: Docker Hub上镜像总数

诚然，很多的这些镜像并不是代码库的“顶端”。我通过它们优秀的Ancestry API去重新构建了大部分的层次树。

举例说吧，我能找到的从根到叶子的最深路径是afrantisak/ska，有120层父镜像。

下面是排名前20“最深”代码库的图标：


![ ](http://media.tumblr.com/878148d5d4a456a24794af3f5a6be445/tumblr_inline_n6yrw4g8lr1sq4ex2.png)


## 将来的工作

一些将来我想要做的事情：

1. Dockerfile相似度计算: 基于我的经验，我们最终会经常拷贝修改现有的Dockerfile。此外，由于很多Dockerfile中的命令都是顺序运行，很多Dockerfile看上去一定是很相似的。从数学角度来看，如果我们将每个Dockerfile看成是单独的线构成的有限序列，并且一个Dockerfile是从另外一个继承而来，它们共用一个大的子序列就变得很有可能。这样也许会很有趣，定义两个Dockerfile的非格律距离，将其作为最大子序列的长度，通过所有的Dockerfile以集群的方式运行在Docker Hub上。

2. 基于 From 继承创建一个Dockerfile树: 这个点子来自[@ewindisch](http://github.com/ewindisch)。目前，可信的版本(意味着Dockerfile托管在Github上)可以从一个不可信的版本继承，因为可信版本可以不从一个可信的版本构建。看到有多少可信的版本有正真可信的血统是一件很有趣的事情。


##感谢Docker公司

最后，但也是最重要的，Docker带给了我很多的乐趣。作为一名开发人员营销人员，编码没有以前多了，而且明显的感觉越来越生疏了。但是能够再次挽起袖子，和全球各地的开发人员一起探索发现新事物(我实现了自己第一份Go代码，学习了很多关于Docker的API的知识)的感觉很棒。因此，感谢每一个参与到Docker编程马拉松活动的每个人=)。
