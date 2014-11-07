# 基于Docker的Rails集群+Ruby负载均衡代理

---

- 原作者：[Muriel Salvan](http://x-aeon.com/wp/author/Muriel/)
- 原文地址：[Rails cluster with Ruby load balancer using Docker](http://x-aeon.com/wp/2014/02/06/rails-cluster-with-ruby-load-balancer-using-docker/)
- 翻译：[liubin](http://liubin.org/)

---

最近我发现了 [Docker](http://www.docker.io/) 这个东西。

并且我在 [2014年2月的 rivierarb的meet up](http://rivierarb.fr/2014/02/04/Drinkup/) 上做了一次演讲，资料可以在slideshare上下载（文档 [下载](![alt](http://resource.docker.cn/ruby-and-docker-on-rails.pdf) ，无需翻墙）。

我打算做一个比较酷的使用Docker的例子：一个Rails集群，以及集群前面的一个Ruby负载均衡服务。我曾想过使用若干不同的机器（每台机器都有自己的IP地址和端口）来运行同样的Rails服务，然后在外部配置一台负载均衡的代理服务器，在各Railf服务之间进行请求分配。

不过使用Docker的话，在一台机器上就可以模拟集群环境了。

![alt](http://resource.docker.cn/setup.png)

这里我们将演示一下如何来创建这个集群环境。

你可以从我的Github项目上下载所有这次测试的源代码。

## 创建Docker镜像

首先我将创建两个Docker镜像，1个用来运行Rails服务，1个运行Ruby代理服务。

1. 首先我制作了一个可信Docker构建（Trusted Docker build），构建脚本也在我的Github上（ Github project docker-ruby ）。使用这个构建脚本创建的镜像为 murielsalvan/ruby ，它基于Ubuntu Precise，并且安装了Ruby 2.1.0p0。

```
docker pull murielsalvan/ruby
```

2. 接着我基于murielsalvan/ruby来创建一个Docker容器，用来执行一个bash shell，然后在里面安装Rails并创建一个Rails应用程序。这个应用程序只有一页，它将打印出它所在服务器的主机名和IP地址（集群里的每台Rails服务都有不同的主机名和IP地址）。你可以从这里查看这个测试程序的源代码。这步的最后，我将把这个容器提交，生成一个新的镜像，并命名为murielsalvan/server，这个镜像执行的时候，会启动Rails服务，并监听3000端口号（容器内的端口号）。

```
docker run -t -i murielsalvan/ruby bash
```

```
docker commit -m=”Test server” -author=”Muriel Salvan <muriel@x-aeon.com>” -run='{“WorkingDir”: “/root/server/”, “Cmd”: ["rails", "s"], “PortSpecs”: ["3000"]}’ acf566f7d155 murielsalvan/server
```

3. 最后我们来基于murielsalvan/ruby创建第二个容器。这个容器也是启动一个bash然后安装一个Ruby代理服务（我用了 [em-proxy](https://github.com/igrigorik/em-proxy)，非常不错的东西，值得一试）。这个Ruby代理服务将接收一个地址列表作为输入参数，并且创建一个基于随机策略的负载均衡服务，轮询所有给定的IP地址以及3000端口。这个代理服务的源代码可以在 [这里](https://github.com/Muriel-Salvan/rails-cluster-docker/tree/master/proxy) 查看。最后我把这个容器也做了提交操作，生成了一个新的镜像 murielsalvan/proxy，它也将监听3000端口。

```
docker run -t -i murielsalvan/ruby bash

docker commit -m=”Proxy server” -author=”Muriel Salvan <muriel@x-aeon.com>” -run='{“PortSpecs”: ["3000"]}’ 7d2431c16b14 murielsalvan/proxy
```

## 运行Rails cluster

在所有的镜像都创建完成之后，我们就可以启动容器了。我写了个小脚本来启动所有需要运行Rails服务的容器，它接收一个N参数，为Rails服务器个数。等Rails服务全部启动完成后，这个脚本会打印出这些服务的IP地址列表。

这个脚本的另一个工作就是将运行Rails服务的容器内的3000端口，绑定到本机的5000+i 端口上，这样就可以非常方便的透过代理服务直接通过 `wget -S -O – http://localhost:5000` 命令来确认每台Rails服务是否运行正常。

```
# run_cluster.rb

nbr_servers = ARGV[0].to_i
 
pipes_in = {}
nbr_servers.times do |idx|
  port = 5000 + idx
  pipe_cmd_in, pipe_cmd_out = IO.pipe
  cmd_pid = Process.spawn("docker run -p #{port}:3000 murielsalvan/server", :out => pipe_cmd_out, :err => pipe_cmd_out)
  puts "Launch server on port #{port}: PID=#{cmd_pid}"
  Process.detach(cmd_pid)
  pipe_cmd_out.close
  pipes_in[cmd_pid] = pipe_cmd_in
end
# Wait for all servers to be up
pipes_in.each do |pid, pipe_in|
  puts "Waiting for PID #{pid} to be listening..."
  found_info = false
  while !found_info
    out = pipe_in.readline.chomp
    puts out
    found_info = out.match(/WEBrick::HTTPServer/) != nil
    sleep 0.01 if !found_info
  end
end
 
puts 'All servers up and running.'
 
# Get their IP addresses
ips = []
`docker ps | sed -e 's/^\\(............\\).*$/\\1/' | tail -#{nbr_servers}`.split("\n").each do |container_id|
  ips << `docker inspect #{container_id} | grep IPAddress | sed -e 's/.*: \\"\\(.*\\)\\".*/\\1/g'`.chomp
end
 
puts ips.join(' ')
```

这段代码执行后输出结果如下所示，请注意最后的那IP地址，这些地址是Rails服务所监听的IP地址，我们在后面的Ruby代理服务器中会使用到这些地址。

```
> ruby -w run_cluster.rb 5

Launch server on port 5000: PID=6559
Launch server on port 5001: PID=6561
Launch server on port 5002: PID=6565
Launch server on port 5003: PID=6571
Launch server on port 5004: PID=6573
Waiting for PID 6559 to be listening...
[2014-02-05 18:19:44] INFO  WEBrick 1.3.1
[2014-02-05 18:19:44] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:44] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6561 to be listening...
[2014-02-05 18:19:42] INFO  WEBrick 1.3.1
[2014-02-05 18:19:42] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:42] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6565 to be listening...
[2014-02-05 18:19:44] INFO  WEBrick 1.3.1
[2014-02-05 18:19:44] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:44] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6571 to be listening...
[2014-02-05 18:19:43] INFO  WEBrick 1.3.1
[2014-02-05 18:19:43] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:43] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6573 to be listening...
[2014-02-05 18:19:41] INFO  WEBrick 1.3.1
[2014-02-05 18:19:41] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:41] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
All servers up and running.
172.17.0.16 172.17.0.14 172.17.0.15 172.17.0.13 172.17.0.12
```

## 启动Ruby代理

同样，我们在这里也通过一段Ruby代码来完成启动代理服务器的工作：

```
#run_proxy.rb
lst_ips = ARGV.cloneProcess.wait(Process.spawn("docker run -p 3000:3000 -t murielsalvan/proxy ruby -w /root/run_proxy.rb #{lst_ips.join(' ')}"))
```

这段代码将启动运行Ruby代理服务的Docker容器，其执行结果如下：

```
> ruby -w run_proxy.rb 172.17.0.16 172.17.0.14 172.17.0.15 172.17.0.13 172.17.0.12

/root/run_proxy.rb:149: warning: `&' interpreted as argument prefix
/root/run_proxy.rb:150: warning: `&' interpreted as argument prefix
/root/run_proxy.rb:151: warning: `&' interpreted as argument prefix
/root/run_proxy.rb:152: warning: `&' interpreted as argument prefix
/usr/local/lib/ruby/gems/2.1.0/gems/em-proxy-0.1.8/lib/em-proxy/backend.rb:37: warning: method redefined; discarding old debug
/usr/local/lib/ruby/gems/2.1.0/gems/em-proxy-0.1.8/lib/em-proxy/connection.rb:126: warning: method redefined; discarding old debug
/root/run_proxy.rb:168: warning: method redefined; discarding old stop
/usr/local/lib/ruby/gems/2.1.0/gems/em-proxy-0.1.8/lib/em-proxy/proxy.rb:17: warning: previous definition of stop was here
Launching proxy at 0.0.0.0:3000...
```

这将启动Ruby代理服务程序，并且监听3000端口。（请先忽略 上面的警告信息吧，也许em-proxy需要做一些清理操作吧 :-)）

## 打开浏览器

服务程序都启动之后，就可以打开浏览器访问了。输入网址 http://localhost:3000，可以确认返回结果里的主机名和IP地址。为了确保每次请求都会发给代理服务来处理，要在每次请求这个网址的时候先清空一下缓存（强制刷新）。

下面的图是两次请求的结果，从中我们可以看出，两次请求是分别由两台不同的Rails服务器返回的。

![alt](http://resource.docker.cn/test1.png)

刷新浏览器之后：

![alt](http://resource.docker.cn/test2.png)

怎样，是不是很简单的一种Rails集群方案？你也可以在自己的机器上尝试一下。

我个人的测试环境如下：64位的Windows 7 主机，然后通过VirtualBox运行了Ubuntu 14.04 Alpha，所有的操作都在虚拟机里进行。性能也还算说的过去（每个请求的响应时间都低于1秒）。

---

本文转载于 [LIUBIN](http://liubin.org/) 的博客，译文地址：[基于Docker的Rails集群+Ruby负载均衡代理](http://liubin.org/2014/02/18/rails-cluster-with-ruby-load-balancer-using-docker/)

