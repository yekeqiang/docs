# Using Docker in your development environment

---

Author: Deni Bertovic

---

Here at GoodCode we love [Docker](https://www.docker.com/). Some of us have been fans since we saw the intro in a lightning talk at PyCon. We’re fans for a reason. Docker solves a lot of problems for us. We do mostly custom webapps some of which have complicated architectures and many moving parts. Managing those parts isn’t easy and that’s where Docker comes in. In this post we’ll try and show you how Docker can make it easy to run your whole production stack locally (we’re not going to talk about deployment – that’s a post for another time).

## What is Docker?

Docker is an “Open platform for developers and sysadmins to build, ship and run distributed applications”. That’s the official definition anyway. Docker utilizes light weight containers to isolate/sandbox processes running in those containers. It’s based on the Linux kernel (and LXC – Linux containers). Most distros nowadays support it, provided you are running a recent enough kernel.

## How does Docker compare to a traditional VM?

![alt](http://resource.docker.cn/vm-vs-docker.png)

*image credit: [docker.com](https://www.docker.com)*

When we say containers are light weight we mean it. In a traditional VM you would have your Host operating system on top of which sits some kind of Hypervisor which runs your virtual machines that each have a Guest operating system running inside. That in turn produces much overhead.

Contrast that to Docker where you don’t have a guest operating system sitting inside each container but rather each container runs on top and shares the Host operating systems kernel. This is much less resource intensive and has lesss overhead in terms of IO access, disk space usage etc.

## Containers vs. Images

Before we go any further it’s important to define containers and images. Images are a sort of immutable snapshot, a template, for starting containers from. You can start as many containers from an image as you want. The key take-away is that images are immutable. Which means you can’t change and existing image, you can add stuff to it an create a new image from that. Every image (well apart from the base ones) has a parent image, so you get a hierarchy, not unlike you have with git, where every commit has a parent commit.

From these images we then start containers, which are running processes. So for instance, we can have node-js whose parent image is ubuntu. In that node-js image we install everything we need to run our app and then we start our app in a container based on that image.

One important note to make is that containers themselves are ephemeral. This is so because a container can’t change an image it started from, it’s immutable. So when you stop a container, and run a new one from the same image you’re starting from ground zero again.

Also, containers use something called a layer fs (popular implementation are AUFS and BTRFS) which enables a container to not copy the whole image onto a different part of disk (like full blown VMs do) but rather it just writes the diffs in regards to the image it started from. This in turn uses far less disk space. Coupled with other features I’ve talked about this enables you to run thousands of containers.

## So how do I use it?


![alt](http://resource.docker.cn/docker-flow.png)

*image credit: [docker.com](https://www.docker.com)*

Typically as a user you would first pull an image from the central Hub. The Hub is a place where people can share images (public or private) with each other. So you can make for instance a PostgreSQL image and share it with you co-workers.:

```
docker login
docker pull denibertovic/postgres
```

It’s worth noting that user images are prefixed with a username, but official images don’t have that prefix.
Once you’ve downloaded the image you can then run a service from that image:

```
docker run -d -t denibertovic/postgres
```

This will run an instance of postgres in the container (in daemon mode). Of course you can then stop/kill/view logs of that container using the docker stop/kill/logs command.

This is all pretty basic. So let’s see how we can connect to that postgres instance we’ve just started. Strangely `psql -Upostgres` doesn’t work. This is because we haven’t exposed any ports from within the container to the outside world yet. We will do that using the `-p` flag.

```
docker run -d -p 5432:5432 -t denibertovic/postgres
```

And now we can connect to the database from the host machine:

```
psql -Upostgres -h localhost
```

What the `-p` flag did is expose a container port from within and mapped it to a port on the host machine.

One other thing to take care of is to make sure we don’t loose the database data when we shut down the container. We do this using volumes:

```
docker run -d -p 5432:5432 -v `pwd`/data:/var/lib/postgresql \
    -t denibertovic/postgres
```

This makes it so when the container thinks that it’s writing to /var/lib/postgresql it’s actually writing to the data directory on the host system. This way when you stop the container and start a new one you retain you database data. Now, there are a couple of caveats, namely you need to make sure that the data directory on the host has the files and ownerships that postgres expects. Also mounting a volume from the host for this purpose is sometimes considered bad practice because it makes the containers less portable (ie. it depends on a specific host path to be available).

A better solution is to use volumes from a data container.

```
docker run --name data_container -d -t denibertovic/postgres \
    /bin/bash -c "echo Started data container"
```

Notice that we just ran the container and printed something to stdout after which point the container exited. There was no need to start the postgresql process. Data containers don’t actually need to be running for other containers to use them.

```
docker run -d -p 5432:5432 --volumes-from data_container \
    -t denibertovic/postgres
```

What this does is it writes the database data in the data_container so when you restart your postgresql container all the data will still be there (provided you use the same data container again). Now you just need to take care to not accidentally delete the data container.

For development I prefer the host mount option more because I tend to delete all of my stopped containers a lot, so I would have to worry about not accidentally deleting my data container as well. That being said, data containers are considered a best practice.

## Container links

Now repeat this process for every service you need (database, redis, memcached etc.) at which point you can use a feature called container links to link all of these containers to a webapp container (not using volumes for brevity).

```
docker run --name postgres -d -t denibertovic/postgres
docker run --name redis -d -t denibertovic/redis
docker run --name memcached -d -t denibertovic/memcached
```

We gave each container a name so we don’t need to work with UUIDs. At this point we can start our webapp container like so:

```
docker run --link postgres:postgres --link redis:redis \
    --link memcached:memcached -d -t webapp
```

What this does is it populates `/etc/hosts` of the webapp container with the hostnames postgres, redis and memcached which point to the IP’s that each container got. You can then use those names to setup you settings files to correctly point to each service. Notice that we didn’t have to expose any ports to the host system because the containers are talking over their private IP range. This is very neat.

You may ask yourself but why should I run my webapp in a container? Why not just run the supporting services and run the webapp on the host like I’m used to. Surely this is a solved problem since every programming language worth mentioning has some sort of package manager and virtual environment solution that you can use to install the project dependencies in a sandbox-like-thing without using docker?

Well this is kinda true, but it all falls apart once you start installing libraries that depend on C headers being present on the host system. A popular example in the Python world is trying to install the python imaging library, where, depending on the host, you end up with JPEG support on one workstation but only PNG support on another co-workers workstation and so on. You can see the problem I’m sure.

By running not just the supporting service, but the webapp as well, in a container you benefit from a single identical runtime on every machine, down to the versions of various C libraries. This is obviously beneficial as it will eliminate the “it’s works on my workstation” mantra.

Hopefully we were able to show you the benefits and how easy it is to run docker locally. If you find some parts too dense or unclear head over [here](https://www.youtube.com/watch?v=Z_o5eaNZhZQ) and watch a talk Deni gave at WebcampZG this year about Docker. In the talk he goes into more detail what each command does and shows how you can replace the CLI shown here with a tool called [fig](http://www.fig.sh/) which basically let’s you specify everything in a nice YAML file and then run everything with just one command `fig up`.

---

Original source: [Using Docker in your development environment](http://goodcode.io/blog/docker-in-development-environment/)