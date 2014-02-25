# PaaS under the hood, episode 1: kernel namespaces
# PaaS的底层实现1：内核命名空间
Posted on November 28, 2012 by Jérôme Petazzoni

Making things simple is a lot of work. At dotCloud, we package terribly complex things – such as deploying and scaling web applications – into the simplest possible experience for developers. But how does it work behind the scenes? From kernel-level virtualization to monitoring, from high-throughput network routing to distributed locks, from dealing with EBS issues to collecting millions of system metrics per minute… As someone once commented, scaling a PaaS is “like disneyland for systems engineers on crack”. Still with us? Read on!

把事情变得简单是一件繁重的工作。在dotCloud，我们把像部署，扩展Web应用程序这样超级复杂的事情，打包成对于开发者来说最简单的体验。但我们是如何做到这点的呢？从内核层级的虚拟化到监控，从高吞吐量的网络路由到分布式锁，从处理EBS问题到每分钟收集无数的系统指标……有人说，扩展一个PaaS是一个不可能的任务，对我们来说是这样么？继续往下看吧！

This is the 1st installment of a series of posts exploring the architecture and internals of platorm-as-a-service in general, and dotCloud in particular. For our first episode, we will introduce namespaces, a specific feature of the Linux kernel used by the dotCloud platform to isolate applications from each other.

这个系列的文章探索了PaaS的架构和内部实现，包括了广义的PaaS和狭义的dotCloud。这是第一篇文章，我们会介绍名称空间（namespaces），这是一个dotCloud平台用来隔离应用程序的Linux内核的特性。

## Part 1: Namespaces

The first time I was introduced to Linux Containers (LXC), I got the (very wrong) impression that they relied mainly on control groups (or “cgroup” in short). It’s an easy mistake to make: each time you create a new container named e.g. “jose”, a cgroup of the same name appears, e.g. in “/cgroup/jose”. But actually, even if control groups are useful to Linux Containers (as we will see in part 2), the really important infrastructure is provided by namespaces.

当我第一次认识 Linux Containers (LXC) 时，我有一个（非常错误的）印象——他们主要依赖于control groups（cgroups）。这是一个很容易犯的错误：当你每次创建一个新的container，比如叫“jose“，就会出现一个同样名字的cgroup，如"/cgroup/jose"。但事实上，虽然cgroups对于Linux Container来说非常有用，但真正重要的底层支持是来自于namespaces。

Namespaces are the real magic behind containers. There are different kinds of namespaces, as we will see; each kind of namespace applies to a specific resource. And, each namespace creates barriers between processes. Those barriers can be at different levels.

Container的隔离主要是由namespaces实现的。下面我们将会看到很多不同的namespaces，他们各自应用于相应的资源，并且从不同的层级将进程隔离开来。

### The pid namespace

This is probably the most useful for basic isolation.

这大概算是最有用的基本隔离了。

Each pid namespace has its own process numbering. Different pid namespaces form a hierarchy: the kernel keeps track of which namespace created which other. A “parent” namespace can see its children namespaces, and it can affect them (for instance, with signals); but a child namespace cannot do anything to its parent namespace. As a consequence:

每一个pid namespace都有自己的进程号。所有的pid namespaces构造成一个层级结构：内核会记录每个namespace的父子关系。父namespace可以看到子namespace，并且可以影响他们（例如，可以向他们发出signals）；但是反之则不行。因此：

- each pid namespace has its own “PID 1” init-like process;
- processes living in a namespace cannot affect processes living in parent or sibling namespaces with system calls like kill or ptrace, since process ids are meaningful only inside a given namespace;
- if a pseudo-filesystem like proc is mounted by a process within a pid namespace, it will only show the processes belonging to the namespace;
- since the numbering is different in each namespace, it means that a process in a child namespace will have multiple PIDs: one in its own namespace, and a different PID in its parent namespace.

- 每一个pid namespace都有自己的，像/sbin/init一样pid=1的进程；
- 每个namespace中的进程不能用类似kill或ptrace这样的系统调用来影响父，或兄弟namespace，因为进程id仅在指定的namespace内部才有意义。
- 如果一个像proc这样的pseudo-filesystem被一个pid namespace中的进程挂载了，/proc目录只会显示这个namespace中的进程。
- 因为每个namespace中的进程编号是独立的，所以一个子namespace中的进程会有多个不同的pid——在自己namespace中的pid和在父namespace中的pid。

The last item means that from the top-level pid namespace, you will be able to see all processes running in all namespaces, but with different PIDs. Of course, a process can have more than 2 PIDs if there are more than two levels of hierarchy in the namespaces.

最后一条意味着顶层的从pid namespace可以看到所有namespace中运行的任意进程，但进程的pid和在子namespace中看到的不同。当然，如果namespace的层级超过2级的话，一个进程也可以有超过2个的pid。

### The net namespace

With the pid namespace, you can start processes in multiple isolated environments (let’s bite the bullet and call them “containers” once and for all). But if you want to run e.g. a different Apache in each container, you will have a problem: there can be only one process listening to port 80/tcp at a time. You could configure your instances of Apache to listen on different ports… or you could use the net namespace.

通过pid namespace，你可以在多个隔离的环境（以后统称容器）中启动进程，但如果你想要在每个容器中运行像Apache这样的网络程序，就会遇到一个问题：一次只能有一个进程监听80/tcp端口。把每个Apache配置为监听不同的端口当然能解决问题……但更简单的方法是，使用net namespace。

As its name implies, the net namespace is about networking. Each different net namespace can have different network interfaces. Even lo, the loopback interface supporting 127.0.0.1, will be different in each different net namespace.

从名字就可以看出，net namespace与网络相关。不同的net namespace可以有不同的网络接口。即使是lo——用于支持127.0.0.1的loopback接口，在每个net namespace中都不相同。

It is possible to create pairs of special interfaces, which will appear in two different net namespaces, and allow a net namespace to talk to the outside world.

我们可以创建一对特殊的网络接口（如eth0），这对接口会出现在两个不同的net namespace，然后允许一个net namespace连接外部的网络。

A typical container will have its own loopback interface (lo), as well as one end of such a special interface, generally named eth0. The other end of the special interface will be in the “original” namespace, and will bear a poetic name like veth42xyz0. It is then possible to put those special interfaces together within an Ethernet bridge (to achieve switching between containers), or route packets between them, etc. (If you are familiar with the Xen networking model, this is probably no news to you!)

一个典型的容器会有自己的loopback接口（lo），和一个特殊的接口，一般叫eth0。eth0的另一端会连接到最顶层的namespace，并且具有一个诗意的名字，如veth42xyz0。然后用桥接的方法把所有的这些eth0和主机连接起来，这样就能实现容器之间的切换，和在容器之间路由数据包等等。（如果你了解Xen的网络模型，那你应该对此感到很熟悉！）

######**译者注：docker默认采用veth方式，lxc的5种网络类型可见http://manpages.ubuntu.com/manpages/precise/man5/lxc.conf.5.html

Note that each net namespace has its own meaning for INADDR_ANY, a.k.a. 0.0.0.0; so when your Apache process binds to *:80 within its namespace, it will only receive connections directed to the IP addresses and interfaces of its namespace – thus allowing you, at the end of the day, to run multiple Apache instances, with their default configuration listening on port 80.

要注意的是，每个net namespace都有自己的，对于INADDR_ANY（即0.0.0.0）的定义；所以当你的Apache进程绑定到namespace内部的*:80时，它就只会接收那些指向这个namespace的ip地址和端口的连接。这让你最终能够运行多个Apache实例，并且使用他们监听80端口的默认配置。

In case you were wondering: each net namespace has its own routing table, but also its own iptables chains and rules.

顺便提一下：每一个net namespace都有自己的路由表，自己的iptables的chains和rules。
