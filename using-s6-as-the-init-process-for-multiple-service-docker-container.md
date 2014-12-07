# Docker: 使用 s6 作為多服務容器的啟動管理程序

##### 作者：林革非（kfei）

---

## 前言

最近在打包 [docktorrent](https://github.com/kfei/docktorrent) 的過程中，遇到了在 container 裡多服務的啟動和管理問題，包含 process init，signal handle，monitor，orphan reaping 等等。把 init process 的 survey 心得及最後決定的 image 架構記錄一下，有錯請指正。

## 在一個容器裡跑多個服務？你確定？

雖然 Docker 官方總是建議一個 container 只跑一個特定服務，然而在很多情況下，將幾個高度耦合的服務封裝在同一個 container 中一起跑也是很乾脆的做法，相對於內建難用的 linking 或透過 fig 管理多 container，有時候還方便的多。 

聽聽 [Phusion](http://phusion.github.io/baseimage-docker/) 對於這個問題是怎麼說的：

> "But I thought Docker is about running a single process in a container?"

> Absolutely not true. Docker runs fine with multiple processes in a container. In fact, there is no technical reason why you should limit yourself to one process – it only makes things harder for you and breaks all kinds of essential system functionality, e.g. syslog.

> **We encourage you to use multiple processes.**

## 沒有 init process 會怎樣？

然而，一旦跑了多個 process，誰當老大（PID = 1）就會是個很重要的問題。在 docktorrent 一開始的打包方法中，我的 entrypoint 腳本大概是像這樣：

```
# Start satellite services
/etc/init.d/nginx start
/etc/init.d/php5-fpm start

# Turn to rTorrent
exec rtorrent
```

也就是說把 nginx 和 php5-fpm 跑起來後，執行 rTorrent 並且把 PID 1 交給他。會讓 rTorrent 作為 PID 1 的原因在於 `docker stop` 只會把 SIGTERM 送給 container 中的 PID 1 process，而 rTorrent 是整個 container 中唯一需要 graceful shutdown 的，所以讓他去接 signal。但這種做法馬上就會發現 `ps -ef` 出現一大堆 `[php] <defunct>` process，或許這是那些 PHP process 自己的問題，但作為 PID 1 的 rTorrent 不能好好回收這些孤兒，似乎也不太理想。一般來說，作為系統頭號 process 應該要負幾個基本責任：

  1. 依序啟動所有該啟動的服務（process）然後常駐背景
  2. 在系統終止時正常關閉（graceful shutdown）這些服務
  3. 隨時回收系統上所有無主孤魂（orphan process）
  4. 可以 respawn 異常掛掉的服務

因此就有必要找個（或者自幹一個）專職的 PID 1，也就是 [init](https://en.wikipedia.org/wiki/Init) 程序來作為 container 執行時的入口。當然，考慮到最後的 image 要盡量精簡，像 Upstart，Systemd 或是 OpenRC 這種重型的方案在 container 環境就暫且先不考慮了。

## Supervisord

於是首先看看 Docker [官方推薦](https://docs.docker.com/articles/using_supervisord/) 的 [Supervisord](http://supervisord.org/)，透過設定檔來配置服務的方式，試用了一下發現還算簡單易用，一個 `supervisord.conf` 的內容大概是這樣寫：

```
[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && /usr/sbin/apache2 -DFOREGROUND"
```
可惜 Supervisord 的設計並不是用來作為 init process 使用的，這在其首頁上已有宣告：

> It shares some of the same goals of programs like launchd, daemontools, and runit. Unlike some of these programs, it is not meant to be run as a substitute for init as “process id 1”. Instead it is meant to be used to control processes related to a project or a customer, and is meant to start like any other program at boot time.

經過測試，Supervisord 在接收到 `docker stop` 送來的 SIGTERM 時，會把訊號轉送給其他 process，等待所有服務 graceful shutdown。然而，孤兒 processes 的問題也確實沒有解決，整個 container 還是充滿 zombie。再說 Supervisord 是 Python 寫成的，要是疊加在 `debian:jessie` 上足足要增加 45MB 的 image size，加上 Python 在執行階段的成本，僅僅為了跑多服務似乎不大划算。

## Runit

下一位，[Runit](http://smarden.org/runit/)。Written in C，足夠輕量，疊加在 `debian:jessie` 上只增加約 10MB 大小，貌似不錯。知名的 Phusion baseimage 系列也使用 Runit（不過只是用來做服務啟動和 supervision，對於 init process 他們自己寫了一隻叫 `my_init` 的 Python 小程式）。

`/usr/bin/runsvdir -P /etc/sv` 是 Runit 掃描並啟動服務的方式，其中 `/etc/sv` 是符合 Runit [規定格式](http://smarden.org/runit/faq.html#create)的 service directory。舉例來說在 `phusion/baseimage` 裡的目錄結構是這樣：

```
# tree /etc/service
/etc/service
|-- cron
|   |-- run
|   `-- supervise
|       |-- control
|       |-- lock
|       |-- ok
|       |-- pid
|       |-- stat
|       `-- status
|-- sshd
|   |-- run
|   `-- supervise
|       |-- control
|       |-- lock
|       |-- ok
|       |-- pid
|       |-- stat
|       `-- status
`-- syslog-ng
    |-- run
    `-- supervise
        |-- control
        |-- lock
        |-- ok
        |-- pid
        |-- stat
        `-- status
```

掃到服務之後 `runsvdir` 會將服務交由另一隻 `runsv` 去啟動並監管（supervise）。看起來也算簡單明瞭，但可惜的是 `runsvdir` 並不適合直接作為 PID 1 使用，原因是他不做訊號轉送，因此 `docker stop` 會導致 container 裡所有服務直接終結。

Runit 裡真正負責 PID 1 的是另一隻叫 `runit-init` 的程式，`runit-init` 啟動之後會把自己替換為 `runit`，然後再叫出 `runsvdir`，於是整個 process tree 大概就會變這樣：

```
runit ─── runsvdir ─┬─ runsv syslog-ng ─── syslog-ng -F
                    ├─ runsv sshd ─── /usr/sbin/sshd -D
                    ├─ runsv crond ─── /usr/sbin/crond -f
```

除了 process tree 深度增加一層不太爽之外，`runit-init` 還另外引進了 run level 的問題，同樣的，只是為了在 container 裡多跑幾個服務要搞這麼多東西似乎有點小題大做了。

## s6

終於來到今天的主角了。

[s6](http://skarnet.org/software/s6/index.html)，Written in C，所有 binary 加起來不到 900KB，非常輕量（也非常冷門）。主要有兩隻程式在工作：`s6-svscan` 作為 PID 1 掃描所有服務並交由 `s6-supervise` 實際啟動並接管。

關於 `s6-svscan` 的幾件事：
  - 收到 `docker stop` 送進來的 SIGTERM 後，轉送給所有 `s6-supervise` process 後接著執行 `.s6-svscan/finish`，如果執行過程中 fail 掉就改跑 `.s6-svscan/crash`。
  - service directory 裡每個 service 可以有三個可執行檔：run/finish/crash，顧名思義簡單易用。
  - 支援特殊 logger service，透過新建名為 `log` 的 service directory 可以自定義 logger。
  - 會一直在背景默默的持續 scan 目錄並回收系統裡的 orphan。

關於 `s6-supervise` 的幾件事：
  - 會先切換到服務的 service directory，接著預設執行 `./run`。
  - `./run` 跑完了如果目錄裡存在 `./finish` 就執行。
  - `./finish` 如果在五秒內沒有退出，會直接被 kill 掉，然後再回頭執行 `./run`（這個行為可以自己控制）。

我的 service directory 結構長這樣：

```
# tree /service
/service
|-- nginx
|   |-- event
|   |-- run
|   `-- supervise
|       |-- control
|       |-- lock
|       `-- status
|-- php5-fpm
|   |-- event
|   |-- run
|   `-- supervise
|       |-- control
|       |-- lock
|       `-- status
`-- rtorrent
    |-- event
    |-- finish
    |-- run
    `-- supervise
        |-- control
        |-- lock
        `-- status
```

試用過後發現 init process 四個基本需求都有滿足，也夠簡單。但是發現另一個問題：`s6-svscan` 把 SIGTERM 送給 `s6-supervise` 後馬上就退出了，導致 `s6-supervise` 管轄的服務根本沒時間 graceful shutdown，這樣子 SIGTERM 有送豈不等於沒送？當然了，在 `.s6-svscan/finish` 裡加一段 sleep 可以 work around，但這樣搞實在不太科學。後來才知道 s6 裡有一隻叫 `s6-svwait` 的 user interface 可以用來等待特定或所有服務，於是在 `.s6-svscan/finish` 加上一段等待主要服務結束的指令：`exec s6-svwait -d /service/rtorrent`，這個問題算是解決。

到此，一個輕量又簡單易用的 init process 解決方案就有了。

## 取得 s6 的 static binaries

參考網路上找到的[做法](https://github.com/jprjr/docker-misc/blob/s6-builder/dockerfiles/arch-s6-builder/build.sh)，先 build 一個基於 Arch 的 utility image 提供 s6 編譯環境，然後在其他要用到 s6 的 image repository 直接取用編好的 binary [tar ball](https://github.com/kfei/s6-builder/blob/master/dist/s6-1.1.3.2-musl-static.tar.xz?raw=true)。具體用法可以參考我的 GitHub [repo](https://github.com/kfei/s6-builder)，或者直接把 binary 抓去試試也行。

## 寫在後面

  - s6 似乎沒有處理啟動順序相依性的框架，如果需要可能要自己用 hook 的方法 work around。但話說回來，在 container 裡跑多服務跑到出現複雜的啟動相依性，好像也有點怪怪的（container VM 化？）。不過如果真的要這樣搞，那你需要的應該是更完整（肥）的 init process。
  - 如果只是要處理 signal forward 讓服務可以 graceful shutdown，則完全沒有必要引入 init process，透過一隻 Bash wrapper 加幾行 trap/wait 就可以辦到。
  - Survey 的過程中發現[有人](http://blog.chazomatic.us/2014/06/18/multiple-processes-inside-docker/)遇到一樣的問題，然後寫了一個叫 [minit](https://github.com/chazomaticus/minit) 的輕量 init process，看起來也還不錯，但我沒有試，如果你有使用心得歡迎分享。
 
 
## References:
  - http://phusion.github.io/baseimage-docker/
  - http://blog.tutum.co/2014/12/02/docker-and-s6-my-new-favorite-process-supervisor/
  - http://blog.chazomatic.us/2014/06/18/multiple-processes-inside-docker/

---

本文原載于作者的 [Blog](http://kfei.logdown.com/posts/245469-using-s6-as-the-init-process-for-muliple-service-docker-container)，歡迎通過 [GitHub](https://github.com/kfei) 與作者交流。
