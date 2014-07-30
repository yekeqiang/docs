# 使用Bash在Tutum上创建并运行Docker容器

[Tutum](https://www.tutum.co/) 提供非常棒的 Docker 容器托管服务.
目前我使用它托管运行在
[jsDelivr](https://hacks.mozilla.org/2014/03/jsdelivr-the-advanced-open-source-public-cdn/)
上的一个机器人项目
[libgrabber](https://github.com/jsdelivr/libgrabber)
来自动更新我们的项目.

我所做的就是为这个机器人创建一个简单的 Docker 镜像, 其中包含所有它运行所需要的东西.
把这个镜像上传到 Tutum 的私有仓库并利用一段 bash 脚本和`crontab`每 30 分钟执行它一次.

libgrabber 每次会工作 3 ~ 5 分钟, 之后这个容器就会被销毁.
我使用的是一个"Small"大小的容器并将计费周期从 30 分钟转为 10 分钟.
(即每次 Tutum 会收取从容器启动开始 10 分钟的使用费用, 超出的时间会按分钟收费)

这种计费方式每次仅 0.182￠, 每天 8.7￠, 每月只需要 $2.6!
它比 DigitalOcean 更便宜. 加之我得到了卓越的自动化并且不必为基础架构和服务器而烦恼.

这里有一些我喜欢的 features:
  - 按分钟计费( 10 分钟以后)
  - 免费的私有 docker 仓库
  - 简单的 API
  - 易于使用
  - 良好的支持

之前 Tutum 有一个非常简单的 API 让你可以用一行代码就创建并运行一个容器.
遗憾的是 Tutum 做了一些调整让它变得稍显繁复, 现在这个过程为:
  1. 创建容器
  2. 从返回的 JSON 响应中获取容器 ID
  3. 使用容器 ID 发送一个启动命令

我想要用一些真正的语言来解析 JSON, 比如在 bash 下完成上述过程.

最简单的方法是使用
[jsawk](https://github.com/micha/jsawk),
但是我懒得鼓捣它,
我的解决方法是:

```bash
uuid=$(curl -s -H "Authorization: ApiKey username: YOURKEY" -H "Content-Type: application/json" -d
'{"image":"r.tutum.co/user/image", "name":"libgrabber", "autodestroy":"ALWAYS", "container_size":"S"}'
https://app.tutum.co/api/v1/container/ | grep -Po '"'"uuid"'"\s*:\s*"\K([^"]*)' $1)

curl -s -H "Authorization: ApiKey username: YOURKEY" -X POST
https://app.tutum.co/api/v1/container/${uuid}/start/
```

大功告成! 现在你可以保存这个脚本并创建一个`cronjob`以随时执行它.

> 原文由 Dmitriy Akulov 发表, [原文链接](http://dakulov.com/create-and-run-tutum-docker-containers-with-bash-and-cron/)

> 译 sina weibo: http://weibo.com/hexteto, mail: hexteto@tsinghua.edu.cn
