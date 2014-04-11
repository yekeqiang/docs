# Docker 0.10：质量和运维工具

今天，我们很高兴发布Docker 0.10。我们希望你也会喜欢。

我们应当感谢社区里为这个版本作出贡献的伙计：Tianon Gravi，Alexander Larsson，Vincent Batts，Dan Walsh，Andy Kipp，Ken Ichikawa，Alexandr Morozov，Kato Kazuyoshi，Timothy Hobbs，Brian Goff，Daniel Norberg，Brandon Philips，Scott Collier，Fabio Falci Rodrigues，Liang-Chi Hsieh，Sridhar Ratnakumar，Johan Euphrosine，Paul Nasrat以及所有在Docker工作的那些出色的家伙。

这次发布是走向Docker 1.0的新的一步。更改日志特别大，主要关注在质量和改善运维工具上。

## 质量

首先，在接近1.0版本时，我们持续关注质量。这个版本包括为期一周的冲刺，这个冲刺里解决问题，加强测试和文档，清理界面的小问题等等。单单是那一周，我们关闭了48个问题单，合并了68个推送申请。这里展示的只是其中的一部分：

 - 新的[集成测试套件](https://github.com/dotcloud/docker/tree/v0.10.0/integration-cli)会可以让我们以命令行的方式限制回归测试。
 - `docker build`输出的问题已经修正
 - 在一台机器上跑几千个容器时可能发生的各种性能和稳定性问题已经被修正。
 - `docker build`处理符号链接的问题已经被修正
 - 命令`docker build`从私有Git库拉文件时，可以使用客户端授权。
 - 在使用设备映射存储器时，各种可靠性和性能改进
 - `docker build`时的缓存问题已经被修正
 - 在容器内可以使用`df`，`mount`和类似的工具
 - `docker build`现在可以调用需要MKNOD能力的命令
 - 众多文档上的小改进
 - 更完善的覆盖全盘的测试
 - 修正了shell的补全
 - 在容器内可以使用tmux和其他终端工具
 - `docker cp`时的内容检测已经被修正
 - lxc执行驱动现在工作在lxc 1.0上
 - 高吞吐分配的网络端口问题已经被修正
 - `docker push`现在支持只推送一个镜像到[Docker index](https://index.docker.io/)，而不是所有有这个标签的镜像
 - 增加了内容和卷相关的错误信息的可读性。
 - `docker run -volumes-from`执行的问题已经被修正
 - 某些Ubuntu版本下Apparmor的问题已经被修正
 - 条件竞争，缓慢的内存泄漏和线程泄漏的问题已经被修正
 - 对某些命令的输出进行排序，以便更容易理解

就像你们看到的，有些问题独立看真的是很小的问题。但是对这些小问题的修正聚集起来，就有了很大的改变。我们计划在下一个版本继续修正问题，不管大小。如果你希望加快某个问题的修正进度，请立刻提醒我们！你可以[开一个新的issue](https://github.com/dotcloud/docker/issues/new)或者在已有的issue下回复，来表达你的意见。当然，如果你希望能做出贡献，我们也很高兴给你指定一些需要注意的issue，并且帮助你开始修正工作。来#docker IRC频道里[打个招呼](https://www.docker.io/community/)吧！

## 运维工具

这次发布，我们在通往1.0的道路上开启了新的一页：运维可读性。为了在生产环境里使用，Docker只是不崩溃是远远不够的。还需要很好的和系统管理员现在使用的工具集成在一起：日志，系统初始化，监控，远程管理，备份，等等。

显然，我们不可能一个版本就达到对系统管理员很友好的理想状态，不过在0.10，我们在这个方向上做了很多重要的努力：

 - *终止行为*：`docker stop`的默认行为，基于“应用安全”的考虑，已经改为返回错误的形式。尤其是，如果一个应用程序无法响应SIGTERM信号，docker会返回一个错误，而不是使用SIGKILL强行中止应用。这意味着`docker stop`可以安全地承载大多数重要或者脆弱的应用程序，而不用考虑数据损坏或者其他副效应的风险。（注意，你依旧需要设计应用程序在意外退出时的恢复工作，Docker无法阻止拔电源！）

 - *信号处理*：Docker进程本身现在可以用同样的方法处理信号。如果收到了SIGTERM，Docker会把这个信号发给所有正在执行的容器，等待它们顺利的退出，然后自己才退出。如果容器无法顺利退出，Docker会忠实的暴露无法退出的行为，持续等待。这就需要让外部工具在 1) 继续等待；2) 放弃退出，或者 3) 使用SIGKILL强制终止 这几个选项里做选择。注意，由于SIGKILL的工作原理，Docker无法把这个信号发送个应用：而是在下次启动时，如果检测到有上次运行遗留的“孤儿”容器，先使用SIGKILL让它们退出。简单来说：Docker永远永远不会发送SIGKILL给容器，除非自己收到了SIGKILL信号。

 - TLS认证：一个被大家呼吁过很多次的特性是，能够在网络上安全的暴露Docker的远程API。现在有了内建的TLS/SSL认证安全支持，Docker 0.10终于可以实现这个特性了。你可以使用SSL对受限访问的API做认证，只允许持有适当证书的主机或用户才能访问。这只是保证远程API安全的第一步，我们在未来有计划提供粒度更细的基于角色的访问控制和其他形式的认证。

 - Systemd切片：Docker现在和一个systemd插件一起发布，这个插件会自动检测宿主环境是否使用systemd做初始化。如果检测到systemd，Docker会自动使用systemd的底层API来管理控制组，而不是默认的直接访问/proc的方式。对现在已经在使用systemd工具来管理资源分配的管理员来说，这意味着每个独立的Docker容器都会在systemd工具里自动出现。

 - 发布哈希值：每个Docker的发布现在都会包含所有生成构建的SHA256和MD5的哈希值。这些哈希值将会发布到文档网站和下载页面，让大家可以确定安装文件没有被篡改。比如，你可以涌入下命令验证官方的Linux和Darwin二进制构建包的SHA256：

        curl https://get.docker.io/builds/Linux/x86_64/docker-0.10.0.sha256
        curl https://get.docker.io/builds/Darwin/x86_64/docker-0.10.0.sha256

最后，为了将来能发布Docker 1.0，我们希望你们能积极测试Docker 0.10。请记录问题单，让我们知道你的反馈意见！

谢谢每个人的支持和帮助，玩得开心！

Docker维护者