# Creating containers - Part 1: Namespaces

---

Author: Michael Crosby

---


This is part one of a series of blog posts detailing how docker creates containers. We will dig deep into the various pieces that are stitched together to see what it takes to make `docker run ...` awesome.

## First, what is a container?

I think the various pieces of technology that goes into creating a container is fairly commonplace. You should have seen a few cool looking charts in various presentations about Docker where you get a quick "Docker uses namespaces, cgroups, chroot, etc." to create containers. But why does it take all these pieces to create a contaienr?
Why is it not a simple syscall and it's all done for me? The fact is that container's don't exist, they are made up. There is no such thing as a "linux container" in the kernel. A container is a userland concept.

## Namespaces

In part one I'll talk about how to create Linux namespaces in the context of how they are used within docker. In later posts we will look into how namespaces are combined with other features like cgroups and an isolated filesystem to create something useful.

First off we need a high level explanation of what a namespace does and why it's useful. Basically, a namespace is a scoped view of your underlying Linux system. There are a few different types of namespaces implemented inside the kernel. As we dig into each of the different namespaces below you can follow along by running `docker run -it --privileged --net host crosbymichael/make-containers`. This has a few preloaded files and configuration to get your started. Even though we will be creating namespaces inside an container that docker runs for us, don't let that trip you up. I opted for this approach as providing a container preloaded with all the dependencies that you need to run the examples is why we are doing this in the first place. To make things a little easier, I'm using the `--net host` flag so that we are able to see your host's network interfaces within our demo container. This will be useful in the network examples. We also need to provide the `--privilged` flag so that we have the correct permissions to create new namespaces within our container.

If you are interested in what the Dockerfile looks like then here it is:

```
FROM debian:jessie

RUN apt-get update && apt-get install -y \
    gcc \
    vim \
    emacs

COPY containers/ /containers/
WORKDIR /containers
CMD ["bash"]
```

I'll be doing the examples in C, as it's sometimes easier to explain the lower level details better than the abstractions that Go provides. So lets start...

## NET Namespace

The network namespaces provides your own view of the network stack of your system. This can include your very own `localhost`. Make sure you are in the `crosbymichael/make-containers` and run the command `ip a` to view all the network interfaces of your host machine.

```
> ip a
root@development:/containers# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:19:ca:f2 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe19:caf2/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:20:84:47 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.103/24 brd 192.168.56.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe20:8447/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 56:84:7a:fe:97:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::5484:7aff:fefe:9799/64 scope link
       valid_lft forever preferred_lft forever
```

Ok cool, so this is all the network interfaces currently on my host system. Yours may look a little different but you get the idea. Now let's write some code to create a new network namespace. For this we will write a skeleton of a small C binary that uses the `clone` syscall. We will start by using clone to run binaries that are already installed inside our demo container. The file `skeleton.c` should be in the working directory of the demo container. We will use this file as the basis of all our examples. Here is the code incase you don't feel like running the container right now.


```
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <sched.h>
#include <sys/wait.h>
#include <errno.h>

#define STACKSIZE (1024*1024)
static char child_stack[STACKSIZE];

struct clone_args {
        char **argv;
};

// child_exec is the func that will be executed as the result of clone
static int child_exec(void *stuff)
{
        struct clone_args *args = (struct clone_args *)stuff;
        if (execvp(args->argv[0], args->argv) != 0) {
                fprintf(stderr, "failed to execvp argments %s\n",
                        strerror(errno));
                exit(-1);
        }
        // we should never reach here!
        exit(EXIT_FAILURE);
}

int main(int argc, char **argv)
{
        struct clone_args args;
        args.argv = &argv[1];

        int clone_flags = SIGCHLD;

        // the result of this call is that our child_exec will be run in another
        // process returning it's pid
        pid_t pid =
            clone(child_exec, child_stack + STACKSIZE, clone_flags, &args);
        if (pid < 0) {
                fprintf(stderr, "clone failed WTF!!!! %s\n", strerror(errno));
                exit(EXIT_FAILURE);
        }
        // lets wait on our child process here before we, the parent, exits
        if (waitpid(pid, NULL, 0) == -1) {
                fprintf(stderr, "failed to wait pid %d\n", pid);
                exit(EXIT_FAILURE);
        }
        exit(EXIT_SUCCESS);
}
```

This is a small C binary that will allow you to run processes like `./a.out ip a`. It uses the arguments that you pass on the cli as the arguments to whatever process you want to use. Don't worry about the specific implementation too much as it's the changes we will be making that are the interesting aspects. Remember, this will execute the binary and arguments of whatever program you want, this means if you want to run one of these demos below and have it spawn a shell session so that you can poke around in your new namespace then go ahead. It is a great way to explore and inspect these different namespaces at your own pace. So to get started let's make a copy of this file to start working with the network namespace.

```
> cp skeleton.c network.c
```

Ok, within this file there is a very special var called `clone_flags`. This is where most of our changes will happen throughout this post. Namespaces are primarily controlled via the clone flags. The clone flag for the network namespace is `CLONE_NEWNET`. We need to change the line in the file `int clone_flags = SIGCHLD;` to `int clone_flags = CLONE_NEWNET | SIGCHLD;` so that the call to `clone` creates a new network namespace for our process. Make this change in `network.c` then compile and run.

```
> gcc -o net network.c
> ./net ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

The result of this run now looks very different from the first time we ran the `ip a` command. We only see a `loopback` interface in the output. This is because our process that was created only has a view of its network namespace and not of the host. And that's it. That is how you create a new network namespace.

Right now this is pretty useless as you don't have any usable interfaces. Docker uses this new network namespace to setup a `veth` interface so that your container has it's own ip address allocated on a bridge, usually `docker0`. We won't go down the path of how to setup interfaces in namespaces at this time. We can save that for another post.

So now that we know how a network namespace is created lets look at the mount namespace.

## MNT Namespace

The mount namespace gives you a scoped view of the mounts on your system. It's often confused with jailing a process inside a `chroot` or similar. This is not true! The mount namespaces does not equal a filesystem jail. So the next time you hear someone say that a container uses the mount namespace to "jail" the process inside it's own root filesystem you can call bullshit because they don't know what they are talking about. Do it, it's fun :)

Let's start by making a copy of `skeleton.c` again for our mount related changes. We can do a quick build and run to see what our current mount points looks like with the `mount` command.

```
> cp skeleton.c mount.c
> gcc -o mount mount.c
> ./mount mount
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,mode=755)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
/dev/disk/by-uuid/d3aa2880-c290-4586-9da6-2f526e381f41 on /etc/resolv.conf type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/disk/by-uuid/d3aa2880-c290-4586-9da6-2f526e381f41 on /etc/hostname type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/disk/by-uuid/d3aa2880-c290-4586-9da6-2f526e381f41 on /etc/hosts type ext4 (rw,relatime,errors=remount-ro,data=ordered)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
```

This is what the mount points look like from within my demo container, yours may look different. In order to create a new mount namespace we use the flag `CLONE_NEWNS`. You may notice something may look weird here. Why is this flag not `CLONE_NEWMOUNT` or `CLONE_NEWMNT`? This is because the mount namespace was the first Linux namespace introduced and the name was just an undersight. If you write code you will understand that as you start building a feature or an application, you don't often have a full picture of the end result. Anyway, let's add `CLONE_NEWNS` to our `clone_flags` variable. It should look something like `int clone_flags = CLONE_NEWNS | SIGCHLD;`.

Lets go ahead and build `mount.c` again and run the same command.

```
> cp skeleton.c mount.c
> gcc -o mount mount.c
> ./mount mount
```

Nothing changed. Whaaat??? This is because the process that we run inside the new mount namespace still has a view on `/proc` of the underlying system. The result is that the new process sort of inherits a view on the underlying mounts. There are a few ways that we can prevent this, like using `pivot_root`, but we will leave that for an additional post on filesystem jails and how `chroot` / `pivot_root` interact with the container's mount namespace.

However, one way that we can try out our new mount namespace is to, well, mount something. Let's create a new `tmpfs` mount in `/mytmp` for this demo. We will do this mount in C and continue to run our same mount command as the args to our mount binary. In order to do a mount inside of our mount namespace we need to add the code to the `child_exec` function, before the call to `execvp`. The code within the `child_exec` function is run inside the newly created process, i.e., inside our new namespace. The code in `child_exec` should look like this:

```
// child_exec is the func that will be executed as the result of clone
static int child_exec(void *stuff)
{
        struct clone_args *args = (struct clone_args *)stuff;
        if (mount("none", "/mytmp", "tmpfs", 0, "") != 0) {
                fprintf(stderr, "failed to mount tmpfs %s\n",
                        strerror(errno));
                exit(-1);
        }
        if (execvp(args->argv[0], args->argv) != 0) {
                fprintf(stderr, "failed to execvp argments %s\n",
                        strerror(errno));
                exit(-1);
        }
        // we should never reach here!
        exit(EXIT_FAILURE);
}
```

We need to first create a directory `/mytmp` before we compile and run with the changes above.

```
> mkdir /mytmp
> gcc -o mount mount.c
> ./mount mount
# cutting out the common output...
none on /mytmp type tmpfs (rw,relatime)
```

I cut out the common output above from the first time I ran `mount`.

The result is that you should see a new mount point for our `tmpfs` mount. Nice! Go ahead and run mount in the current shell just for comparison.

Notice how the `tmpfs` mount is not displayed? That is because we created the mount inside our own mount namespace, not in the parent's namespace.

Remember how I said that the mount namepaces does not equal a filesystem jail? Go ahead and run our `./mount` binary with the `ls` command. Everything is there. Now you have proof!

## UTS Namespace

The next namespace is the UTS namespace that is responsible for system identification. This includes the `hostname` and `domainname`. It allows a container to have it's own hostname independently from the host system along with other containers. Let's start by making a copy of `skeleton.c` and running the `hostname` command with it.

```
> cp skeleton.c uts.c
> gcc -o uts uts.c
> ./uts hostname
development
```

This should display your system's hostname (`development` in my case). Like earlier, let's add the clone flag for the UTS namespace to the clone_flags variable. The flag should be `CLONE_NEWUTS`. If you compile and run then you should see the exact same output. This is totally fine. The values in the UTS namespace are inherited from the parent. However, within this new namespace, we can change the hostname without it affecting the parent or other container's that have a separate UTS namespace.

Let's modify the hostname in the `child_exec` function. To do that, you will need to add the `#include <unistd.h>` header to gain access to the sethostname function, as well as the `#include <string.h>` header to use `strlen` needed by `sethostname`. The new body of the `child_exec` function should look like the following:

```
// child_exec is the func that will be executed as the result of clone
static int child_exec(void *stuff)
{
        struct clone_args *args = (struct clone_args *)stuff;
        const char * new_hostname = "myhostname";
        if (sethostname(new_hostname, strlen(new_hostname)) != 0) {
                fprintf(stderr, "failed to execvp argments %s\n",
                        strerror(errno));
                exit(-1);
        }
        if (execvp(args->argv[0], args->argv) != 0) {
                fprintf(stderr, "failed to execvp argments %s\n",
                        strerror(errno));
                exit(-1);
        }
        // we should never reach here!
        exit(EXIT_FAILURE);
}
```

Ensure that `clone_flags` within your main look like `int clone_flags = CLONE_NEWUTS | SIGCHLD`; then compile and run the binary with the same args. You should now see the value that we set returned from the `hostname` command. To verify that this change did not affect our current shell go ahead and run `hostname` and make sure that you have your original value back.

```
> gcc -o uts uts.c
> ./uts hostname
myhostname
> hostname
development
```

Awesome! We are doing good.

## IPC Namespace

The IPC namespace is used for isolating interprocess communication, things like SysV message queues. Let's make a copy of `skeleton.c` for this namespace.

```
> cp skeleton.c ipc.c
```

The way we are going to test the IPC namespace is by creating a message queue on the host, and ensuring that we cannot see it when we spawn a new process inside it's own IPC namespace. Let's first create a message queue in our current shell then compile and run our copy of the skeleton code to view the queue.

```
> ipcmk -Q
Message queue id: 65536
> gcc -o ipc ipc.c
> ./ipc ipcs -q
------ Message Queues --------
key        msqid      owner      perms      used-bytes   message
0xfe7f09d1 65536      root       644        0            0
```

Without a new IPC namespace you can see the same message queue that was created. Now let's add the `CLONE_NEWIPC` flag to our `clone_flags` var to create a new IPC namespace for our process. The `clone_flags` var should look like `int clone_flags = CLONE_NEWIPC | SIGCHLD;`. Recompile and run the same command again:

```
> gcc -o ipc ipc.c
> ./ipc ipcs -q
------ Message Queues --------
key        msqid      owner      perms      used-bytes   message
```

Done! The child process is now in a new IPC namespace and has completely separate view and access to message queues.

## PID Namespace

This one is fun. The PID namespace is a way to carve up the PIDs that one process can view and interact with. When we create a new PID namespace the first process will get to be the loved PID 1. If this process exits the kernel kills everyone else within the namespace. Let's start by making a copy of `skeleton.c` for our changes.

```
> cp skeleton.c pid.c
```

To create a new PID namespace, we will have to set the `clone_flags` with `CLONE_NEWPID`. The variable should look like `int clone_flags = CLONE_NEWPID | SIGCHLD;`. Let's test by running `ps aux` in our shell and then compile and run our pid.c binary with the same arguments.

```
> ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  20332  3388 ?        Ss   21:50   0:00 bash
root       147  0.0  0.1  17492  2088 ?        R+   22:49   0:00 ps aux
> gcc -o pid pid.c
> ./pid ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  20332  3388 ?        Ss   21:50   0:00 bash
root       153  0.0  0.0   5092   728 ?        S+   22:50   0:00 ./pid ps aux
root       154  0.0  0.1  17492  2064 ?        R+   22:50   0:00 ps aux
```

WTF??? We expected `ps aux` to be PID 1 or atleast not see any other pids from the parent. Why is that? proc. The process that we spawned still has a view of `/proc` from the parent, i.e. `/proc` mounted on the host system. So how do we fix this? How to we ensure that our new process can only view pids within it's namespace? We can start by remounting `/proc`.

Because we will be dealing with mounts, we can take the opportunity to take what we learned from the MNT namespace and combine it with our PID namespace so that we don't mess with the `/proc` of our host system.

We can start by including the clone flag for the mount namespace along side the clone flag for pid. It should look something like `int clone_flags = CLONE_NEWPID | CLONE_NEWNS | SIGCHLD;`. We need to edit the `child_exec` function and remount proc. This will be a simple `unmount` and `mount` syscall for the proc filesystem. Because we are creating a new mount namespace we know that this will not mess up our host system. The result should look like this:

```
// child_exec is the func that will be executed as the result of clone
static int child_exec(void *stuff)
{
        struct clone_args *args = (struct clone_args *)stuff;
        if (umount("/proc", 0) != 0) {
                fprintf(stderr, "failed unmount /proc %s\n",
                        strerror(errno));
                exit(-1);
        }
        if (mount("proc", "/proc", "proc", 0, "") != 0) {
                fprintf(stderr, "failed mount /proc %s\n",
                        strerror(errno));
                exit(-1);
        }
        if (execvp(args->argv[0], args->argv) != 0) {
                fprintf(stderr, "failed to execvp argments %s\n",
                        strerror(errno));
                exit(-1);
        }
        // we should never reach here!
        exit(EXIT_FAILURE);
}
```

Build and run this again to see what happens.

```
> gcc -o pid pid.c
> ./pid ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   9076   784 ?        R+   23:05   0:00 ps aux
```

Perfect! Our new PID namespace is now fully operational with the help of the mount namespace!

## USER Namespace

The last namespace is the user namespace. This namespace is the new kid on the block and allows you to have users within this namespace that are not equal users outside of the namespace. This is accomplished via GID and UID mappings.

This one has a simple demo application without specifying a mapping, even though it's a totally useless demo. If we add the flag `CLONE_NEWUSER` to our `clone_flags` then run something like `id` or `ls -la` you will notice that we get `nobody` within the user namespace. This is because the current user is undefined right now.

```
> cp skeleton.c user.c
# add the clone flag
> gcc -o user user.c
> ./user ls -la
total 84
drwxr-xr-x 1 nobody nogroup 4096 Nov 16 23:10 .
drwxr-xr-x 1 nobody nogroup 4096 Nov 16 22:17 ..
-rwxr-xr-x 1 nobody nogroup 8336 Nov 16 22:15 mount
-rw-r--r-- 1 nobody nogroup 1577 Nov 16 22:15 mount.c
-rwxr-xr-x 1 nobody nogroup 8064 Nov 16 21:52 net
-rw-r--r-- 1 nobody nogroup 1441 Nov 16 21:52 network.c
-rwxr-xr-x 1 nobody nogroup 8544 Nov 16 23:05 pid
-rw-r--r-- 1 nobody nogroup 1772 Nov 16 23:02 pid.c
-rw-r--r-- 1 nobody nogroup 1426 Nov 16 21:59 skeleton.c
-rwxr-xr-x 1 nobody nogroup 8056 Nov 16 23:10 user
-rw-r--r-- 1 nobody nogroup 1442 Nov 16 23:10 user.c
-rwxr-xr-x 1 nobody nogroup 8408 Nov 16 22:40 uts
-rw-r--r-- 1 nobody nogroup 1694 Nov 16 22:36 uts.c
```

This is a very simple example of the user namespace but you can go much deeper with it. We will save this for another post but the idea and hopes for the user namespace is that this will allow us to run as "root" within the container but not as "root" on the host system. Don't forget you can always change `ls -la` to `bash` and have a shell inside the namespace to poke around and learn more.

## In the end...

So to recap we went over the mount, network, user, PID, UTS, and IPC Linux namespaces. The majority of the code that we changed was not much, just adding a flag most of the time.
The "hard work" is mostly managing the interactions between the various kernel subsystems in order to meet our requirements. Like most of the descriptions before this, namespaces are just one of the tools that we use to make a container. I hope the PID example is a glimpse of how we use multiple namespaces together in order isolate and begin the creation of a container.

In future posts we will go into detail on how we jail the container's processes inside a root filesystem, aka a docker image, as well as using cgroups and Linux capabilities. By the end we should be able to pull all these things together to create a container.

Also a thanks to tibor and everyone that helps review my brain dump of a first draft ;)

---

Original source: [Creating containers - Part 1: Namespaces](http://crosbymichael.com/creating-containers-part-1.html)