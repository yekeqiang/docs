# Docker 1.3 发布，增加数字签名校验

---

Linux 容器引擎 Docker 1.3 正式发布了，该版本会自动的使用数字签名验证所有官方仓库的出处和一致性。官方仓库是来自 Docker 社区贡献的优秀的 Docker 镜像，一个可靠的签名保证该镜像的信任程度，而且不会被篡改。目前该特性还是技术预览阶段。

此外新版本还支持 docker exec 注入新的进程，例如：

?
1
$ docker exec ubuntu_bash -it bash
支持使用 docker create 调整容器的生命周期，增加额外的安全参数 --security-opt，boot2docker 实现在 OS X 下的共享目录等等，详细的内容请看发行说明。
