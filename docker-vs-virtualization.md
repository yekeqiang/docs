# Docker vs Virtualization

---

##### Author: Sleekd

---

The motivation behind this post is to answer a simple question: What’s the difference between Docker and classic Virtualization techniques? I set out to research this topic in depth and I will share my findings. I am by no means an expert in either Docker or virtualization so feel free to comment if you find any inconsistencies.

I will start out by briefly talking about Operating Systems and the Kernel. Then move on to the Kernel’s role in virtualization. Finally I will explain how Docker works and how it differs from classic virtualization.

## Operating Systems

This is a broad subject but I will keep this overview very short, there’s plenty of literature out there. The Kernel is the component of the OS that provides an abstraction layer between Device Drivers and Software. The Applications running in the OS use the Kernel System API to request access to services from the Kernel (things like storage, memory, network or process management). For example, if you call File.open in Ruby, at some point in the execution the open system call will be executed and the Kernel will abstract away the interaction with the physical hard drive. If you’re interested to read more about operating systems I suggest check out the [Operating Systems: Three Easy Pieces](http://pages.cs.wisc.edu/~remzi/OSTEP/).

## Virtualization

The key component in any virtualization software is the Hypervisor, also known as the virtual machine monitor (VMM). The hypervisor can be thought of as an API that provides access to the hardware level for the virtual machines.

There are two types of hypervisors: hosted and bare-metal. Most desktop virtualization software such as VirtualBox or Vmware Fusion/Player/Workstation use a hosted hypervisor. That means the hypervisor runs as an application and is letting your Operating System’s Kernel deal with hardware drivers or resource management. The bare-metal hypervisor on the other hand runs directly on the host machine’s hardware. Think of it as a specialized OS that has extra instructions built-in to deal with the virtual machine’s access to the actual hardware and resources.

![alt](http://resource.docker.cn/hypervisor-types.png)

The way the Kernel is handling System Calls from Virtual Machines is the main difference between virtualization solutions. With **paravirtualization**, the OS running on the virtual machine has a modified Kernel that accesses system resources using a Hypervisor Call rather than a System Call. This requires a modified OS on the virtual machine because a vanilla OS will not know to use a HyperCall instead of a System Call. **Full virtualization** simulates the hardware of the host machine completely and commands are executed as if they would be running on dedicated hardware (through a System Call). This has the advantage that we don’t need to run a modified OS. The only downside is that the System Call inside the virtual machine needs to be translated and sent to the host machine’s Kernel. This extra step reduces performance. Processors like Intel VT-x and AMD-V fix this problem by providing virtualization hardware instructions and eliminating the System Call translation step.

## Docker

Docker doesn’t run different virtual machines. Instead it uses built-in Linux Kernel containment features like CGroups, Namespaces, UnionFS, chroot (more on these later) to run applications in virtual environments. Those virtual environments – called Docker containers, have separate user lists, file systems or network devices.

Initially Docker was built as an abstraction layer on top of Linux Containers (LXC). LXC itself is a just an API for the Linux containment features. Starting with Docker 0.9, LXC is not the default anymore and has been replaced with a custom library ([libcontainer](https://github.com/docker/libcontainer)) written in Go. Overall libcontainer’s advantage is a more consistent interface to the Kernel across various Linux distributions. The only gotcha is that it requires Linux 3.8 and higher.

Let’s look at some of those Kernel features used by Docker.

### Namespaces and Containers

Namespaces isolate processes such as users lists, network devices, process lists and filesystems. There are currently 6 namespaces implemented to date:

- mnt (mount points, filesystems)
- pid (processes)
- net (network stack)
- ipc (System V IPC)
- uts (hostname)
- user (UIDs)

Namespaces are not a new concept, the first one to be implemented – the mount namespace was added to Linux 2.4.19 on 2002.

### CGroups

CGroups is another Kernel feature heavily used by Docker. While the namespace isolates various interactions with the Kernel, the role of CGroups is to isolate or limit resource usage (CPU, memory, I/O).

### Union file systems

This Linux service allows you to mount files and directories from other file systems (ie. a namespace isolated file system) and combine them to form a single file system. You can read more about it in this [Wikipedia article](http://en.wikipedia.org/wiki/UnionFS).

When Docker boots a container from an image it first mounts the root file system as read only. After that, instead of making the file system read-write, Docker attaches another file system layer to that container using union mounts. This process continues every time a change to the file system of the container happens. You will notice that when you push an image you create to the docker registry there are many images getting pushed, some of them already exist there, some do not and take longer to upload. UnionFS allows Docker to create a repository of file system changes and this is a wicked cool feature! It saves space and allows you to diff changes to containers very easily.

You can see this hierarchy by running: docker images –tree. By the way, this functionality is being removed from the core docker client and it’s being worked on as a [separate project](https://github.com/justone/dockviz). Here you can see how my two ruby images are based on the main rbenv image.

```
$ docker images --tree
  └─a4d37 Virtual Size: 407.1 MB Tags: tzumby/rbenv:latest
    ├─03a7 Virtual Size: 508.4 MB Tags: tzumby/ruby-2.0.0:latest
      └─f6ae Virtual Size: 521.7 MB Tags: tzumby/ruby-2.1.0:latest
```

Let’s inspect the UnionFS layers. If you are running Ubuntu you can cd straight into the docker lib folder at `/var/lib/docker`. If you are on OS X your docker daemon is most likely running in a VirtualBox VM and you can access that by running:

```
$ boot2docker ssh
```

Now you can cd into the docker lib folder and check out the UnionFS layers it created for all your containers.

```
$ cd /var/lib/docker/aufs/diff
```

I will explore this in more depth in future articles but if you are curious you can sort all the folders by date (*ls -ltr*) and check their contents as you install packages on your container. For example, after I installed rbenv on the system I could find the folder that had just the rbenv changes to the file system. Pretty neat!

## Conclusion

We just quickly went over virtualization and the Docker architecture. Although both Docker and modern virtualization are relatively new, the underlying technologies are not new at all. Before Docker we would run processes using chroot or Jails in FreeBSD for improved security for example.

So should you use Docker or classic virtualization? In reality virtualization and Docker can and are used together in modern dev-ops. Most VPS providers are running bare-metal full virtualization technologies like Xen and Docker usually runs on top of a virtualized Ubuntu instance.

---

Original source: [Docker vs Virtualization](http://sleekd.com/servers/docker-vs-virtualization/)
