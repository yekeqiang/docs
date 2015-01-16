# Deis 1.0 正式版发布，可用于生产环境！

---

##### 译者：Fiona Feng

---

![alt](http://resource.docker.cn/deis-1-0.jpeg)


基于 Docker 的开源 PaaS 系统 [Deis 1.0](https://github.com/deis/deis/releases/tag/v1.0.0) 正式版发布了，这是 Deis 的首个稳定版本，可应用于生产环境，这也是 Deis 首个基于 Docker 构建的产品级别的 PaaS 系统。

Deis 1.0 提供了稳定的 API、丰富的功能特性以及可靠的组件架构。包括：

- 平台质量 - Deis 由久经沙场的社区进行测试，可处理企业级产品负载
- 安装便捷 - Deis 可通过一个简单的命令行工具在 30 分钟内安装到 CoreOS 集群中
- 可用性高 - 整个 Deis 平台具有高度可用性，可在集群中实现容错
- 成熟的流程 - Deis 提供三种部署流程，包括：Heroku Buildpacks, Dockerfiles 和原生的 Docker Images
- 文档完善 - 完善和改进为开发者和管理员提供的文档，并提供独立的文档站点
- 可在任何平台运行 – Deis 可运行于公有云、私有云和裸机，目前已经通过了包括 AWS、Google Compute Engine、Digital Ocean、Rackspace、OpenStack 和 VMware 在内的认证

我们要感谢由 Deis 用户和贡献者组成的庞大的社区，他们愿意将 Deis 的早期版本应用于生产，使得我们得以解决那些在实战环境中才能出现的问题。我们感谢你们的耐心和等待。


在过去16个月的开发周期里， Deis 与同侪中的佼佼者 Docker 、CoreOS 、 Ceph 、 Heroku 紧密合作，我们要向这些项目背后的诸位致谢。正是你们持续的支持才让 Deis 作为一个开源 PaaS 取得如此巨大的成功。

如果你是 Deis 早期版本的用户，可参考 "[Upgrading Deis](http://docs.deis.io/en/latest/managing_deis/upgrading-deis/)" 文档进行升级。

## Deis 1.0 的改进概要：

### 新特性

- [http://docs.deis.io/](http://docs.deis.io/) 独立的文档站点，对文档内容进行重新组织
- 原来在 README 文件中的文档移到文档站点
- 添加新的 DigitalOcean 指南
- 解决与 Docker 镜像和文档相关的错误内容
- 提供 Docker 1.3.1 TLS 认证的测试套件
- 修正老版本发布容器后不能发布到路由的错误
- `deisctl help <command>` 能够持续打印有用的使用信息
- `deis` CLI 使用 `$DEIS_DRINK_OF_CHOICE` 作为环境变量

### 组件更新

- 更新到 [CoreOS 494.0.0](https://coreos.com/releases/#494.0.0)
- 更新到 [Ceph 0.87 "giant"](http://ceph.com/docs/master/release-notes/#v0-87-giant)
- **builder** 、 **controller** 和 `deis` CLI 要求 python 2.4.3
- **controller** 更新 Django REST framework 至 2.4.4
- **controller** 更新 python-etcd 至 0.3.2
- **controller** 更新 South 至 1.0.1

完整记录请查看 [CHANGELOG.md](https://github.com/deis/deis/blob/master/CHANGELOG.md)。

#### 社区贡献

我们要向以下社区成员致谢，他们在 GitHub 的 Deis 项目中贡献良多：

- @abonas: 在 vagrant setup 中缺乏对生成 ssh key 的指导
- @achannarasappa: dies 组件会卡在 auto-restart/failed 状态，请求的 URL /  在此服务器上未发现
- @adrianmoya: deis-store-metadata inactive/dead ，首次安装 0.15.1 导致 database/builder 组件错误，deisctl 更新失败
- @aledbf: 注册会跳转到 https? ，deis-store-volume 挂载错误， deis 客户端错误， 修改 "make discovery-url" 有效性指令，协助配置 s3cmd 访问存储数据，修复（路由）：移除存储关口限制，适配 coreos ：在工具箱中定制镜像，适配 publisher ：使用 godep ， 适配 logspout ：使用 godep ，参考 builder ：从 builder 中移除 dockerfile 语法分析程序
- @bilts: 在启动 node reboot 后 controller and builder 服务失败
- @EvanK: store-volume 卡在start-pre/auto-restart 并循环 `deisctl start platform`
- @gahissy: deis 日志错误
- @gebrits: Deis for apps + Smartstack 后端服务。 Good combination?
- @grengojbo: deis push 无效
- @hugo-zheng: push refs 到 git 时失败
- @Imdsm: 适配 provision-rackspace ：向指定环境添加可选参数，文档（ installing_deis ）: 修正格式问题
- @jamespacileo: PHP 项目失败， "deisctl start platform" 卡在 deis-store-monitor
- @jannispl: 从 0.14.1 后升级 deis-store-volume 不能启动
- @johanneswuerbach: 将 deis 安装和测试迁移到 beta 频道，针对 cores alpha channel 进行测试， 提案：针对网速慢提供 in-house 镜像， Building does not stop on compilation errors, 记录哪些 key 会丢失，允许使用任一 S3 适配存储，添加存储问题解决文档， 根据 cores 更新更新文档，包括 feat(): 不提交本地 vagrant 设置，fix(): 修正所需 coreos 版本, fix(tests):允许 TLS
- @jorihardman: nginx X-Forwarded-Proto breaks https endpoint forwarding
- @Joshua-Anderson: feat(client): 当写设置失败时添加错误信息
- @justinhennessy: New cluster register call failing ..., router: waiting for confd
- @kingcody: 客户端：从当前仓库中移除 git remote ， 安装行为？；平台：移走一个引起配置错误的 deis-* 容器，客户端：Exception.message is deprecated for >= python2.7, 文档：broken link for install_deisctl, fix(deis/client): `Exception.message` is deprecated for >= python2.7
- @laczoka: deisctl 使用 deis-store-gateway.service 启动时平台会卡住
- @loungerider: 在 digitalocean 集群中部署 helloworld dockerfile 示例不工作
- @lsnyder: deisctl --request-timeout 选项不适用于预期的超时
- @lukedemi: Fleet 间歇性崩溃
- @matiasdecarli: 在一台虚拟机重启后 units 不能重启
- @ngpestelos: Deis CLI 命令超时， deis-store-volume 重启失败， `deisctl start platform` 超时， docs(deisctl): 重写评论， docs: 澄清 tags_set 用法
- @Paperback: docs(README.md): 替代被移走的 DNS 链接
- @rpaladugu1: deisctl 配置平台确定为 domain=
- @viztor: dies 是否支持多个位置部署？
-- @Xe: LICENSE 需要更新到2014，客户端：auth:logout and auth:clear 貌似多余, Confd 在 deis-builder 中死循环，使用CentOS 6 注册用户名除外，fix(contrib/userdata): nse uses `docker exec`

Deis 社区持续壮大，可能 Deis 不能在这里将你们一一列举。如果你发现自己的名字被漏掉，请告诉我们，我们会随时更新。

## 目前已知的问题：

### Docker 1.3.1

从 Docker 1.3.1 开始使用 TLS 用于所有 registry 之间的通讯。这导致使用私有 registry 时的一系列问题，目前 Deis 正在尝试解决这个问题，因此目前 Deis 只支持 Docker 1.3.0。

### 升级中的日志丢失问题

当从早期版本升级到 1.0 时，可能会丢失一些平台日志数据，这是因为 Ceph 组件升级的原因导致。因此我们建议升级前请阅读 [备份和恢复过程](http://docs.deis.io/en/latest/managing_deis/backing_up_data/) 文档来确保升级过程中数据不丢失。

## 未来计划

### 交互式管理命令

尽管 `deis run` 可在容器中执行管理命令，目前还不支持长时间运行的交互式命令，例如 `deis run bash` 。

### 服务网关

Deis 必须简化可重用的后端服务的发布，例如数据库、队列、存储等，并允许开发者方便的将服务绑定到应用中。这将会是一种松耦合的方式进行。你可通过 [GitHub issue](https://github.com/opdemand/deis/issues/231) 了解最新进展情况。

## 参与 Deis 项目

- 为我们的 [GitHub repository](https://github.com/opdemand/deis) 加星
- 跟随我们的 Twitter 帐号 [@opendeis](http://twitter.com/opendeis) 并扩散我们的推文
- 加入 Freenode 上的 [#deis channel] 频道，参与讨论 
- 选择一个 [easy-fix issue](https://github.com/deis/deis/issues?labels=easy-fix&state=open) ，贡献代码！

访问我们的网站，了解更多 [get involved](http://deis.io/get-involved/) 的相关信息。

---

官方发行说明请看：[Deis v1.0 - Production Ready](http://deis.io/deis-1-0-production-ready/)