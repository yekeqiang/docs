# Storage Concepts in Docker: Shared Storage and the VOLUME directive

---

##### Author: Mark Lamourine  

---

In the next few posts  I'm going to take a break from the concrete work of creating images for Pulp in Docker.  The next step in my project requires some work with storage and it's going to take a bit of time for exploration and then some careful planning.  Note that when I get to moving them to Kubernetes I'll have to revisit some of this, as Kubernetes Pods place some constraints (and provide some capabilities) that Docker alone doesn't.

This is going to take at least three posts:

- Shared Storage in Docker (this post)
- [Persistent Storage in Docker](http://cloud-mechanic.blogspot.com/2014/10/storage-concepts-in-docker-persistent.html)
- [Persistent Storage in Kubernetes](http://cloud-mechanic.blogspot.com/2014/10/storage-concepts-in-docker-network-and.html)

It could take a fourth post for Persistent Storage in Kubernetes, but that would be a fairly short post because the answer right now is "you really can't do that yet". People are working hard to figure out how to get persistent storage into a Kubernetes cluster, but it's not ready yet.

For now I'm going to take them one large bite at a time.

## Storage in a Containerized World


The whole point of containers is that they don't leak. Nothing should escape or invade. The storage that is used by each container has a life span only as long as the container itself. Very quickly though one finds that truly closed containers aren't very useful.  To make them do real work you have to punch some holes.

The most common holes are network ports, both inbound and out.  A daemon in a container listens for connections and serves responses to queries.  It may also initiate new outbound queries to gather information to do its job.  Network connections are generally point-to-point and ephemeral.  New connections are created and dropped all the time.  If a connection fails during a transaction, no problem, just create a new connection and resend the message. Sometimes though, what you really need is something that lasts.

## Shared and Persistent Storage


Sometimes a process doesn't just want to send messages to other processes.  Sometimes it needs to create an artifact and put it someplace that another process can find and use it.  In this case network connections aren't really appropriate for trading that information.  It needs disk storage. Both processes need access to the same bit of storage.  The storage must be shared.

Another primary characteristic of Docker images (and containerized applications in general) is that they are 100% reproducible.  This also makes them disposable.  If it's trivial to make arbitrary numbers of copies of an image, then there's no problem throwing one away.  You just make another.

When you're dealing with shared storage the life span of a container can be a problem too.  If the two containers which share the storage both have the same life span then the storage can be "private", shared just between them.  When either container dies, they both do and the storage can be reclaimed. If the contents of the storage has a life span longer than the containers, or if they container processes have different life spans then the storage needs to be persistent.


## Pulp and Docker Storage


The purpose of the Pulp application is to act as a repository for long-term storage.  The payload are files mirrored from remote repositories and offered locally. This can minimize long-haul network traffic and allow for network boundary security (controlled proxies) which might prohibit normal point-to-point connections between a local client and a remote content server.

Two processes work with the payload content directly.  The Pulp worker process is responsible for scanning the remote repositories, detecting new content and initiating a sync to the local mirror. The Apache process publishes the local content out to the clients which are the customers for the Pulp service. It consumes the local mirror content that has been provided by the Pulp workers.  These two processes must both have access to the same storage to do their jobs.

For demonstration purposes, shared storage is sufficient.  The characteristics of shared storage in Docker and Kubernetes is complex enough to start without trying to solve the problem of persistence as well.  In fact, persistent storage is still a largely an unsolved problem.  This is because local persistent storage isn't very useful as soon as you try to run containers on different hosts.  At that point you need a SAN/NAS or some other kind of network storage like OpenStack Cinder or AWS/EBS or Google Cloud Storage.

So, this post is about the care and feeding of shared storage in Docker applications.

## Docker Image: the VOLUME directive


The Dockerfile has a number of directives which specify ways to poke holes in containers.  [The VOLUME directive](https://docs.docker.com/reference/builder/#volume) is used to indicate that a container wants to use external or shared storage.


![alt](http://resource.docker.cn/pulp-image-volumes.png)

The diagram above shows the effect of a VOLUME directive when creating a new image. It indicates that this image has two mount points which can be attached to external (to the container) storage.

Here's the complete Dockerfile for the pulp-content image.

```
FROM markllama/pulp-base
MAINTAINER Mark Lamourine <markllama@gmail.com>
# For Testing Only
VOLUME ["/var/www", "/var/lib/pulp"]
CMD ["sleep", "86400"]
view rawdocker_pulp_content_volumes_dockerfile hosted with ❤ by GitHub
```

Here's where the window metaphor breaks down.  The VOLUME directive indicates a node in a file path where an external filesystem may be mounted.  It's a dividing line, inside and outside.  What happens to the files in the image that would fall on the outside?

Docker places those files into their own filesystem as well.  If the container is created without specifying an external volume to mount there, this default  filesystem is mounted.  The VOLUME directive defines a place where files can be imported or exported.

So what happens if you just start a container with that image, but don't specify an external mount?

## Defaulted Volumes


To continue with the flawed metaphor, every window has two sides. The VOLUME directive only specifies the boundary.  It says "some filesystem may be provided to mount here". But if I don't provide a file tree to mount there (using the -v option) Docker mounts the file tree that was inside the image when it was built. I can run the pulp-content image with a shell and inspect the contents.  I'll look at it both from the inside and the outside.

I'm going to start an interactive pulp-content container with a shell so I can inspect the contents.

```
Container Mounts with Volumes

docker run -it --name volume-demo markllama/pulp-content /bin/sh
sh-4.2# mount
/dev/mapper/docker-8:4-2758071-77b5c9ba618358600e5b59c3657256d1a748aac1c14e2be3d9c505adddc92ce3 on / type ext4 (rw,relatime,context="system_u:object_r:svirt_sandbox_file_t:s0:c585,c908",discard,stripe=16,data=ordered)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,context="system_u:object_r:svirt_sandbox_file_t:s0:c585,c908",mode=755)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,context="system_u:object_r:svirt_sandbox_file_t:s0:c585,c908",size=65536k)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,context="system_u:object_r:svirt_sandbox_file_t:s0:c585,c908",gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime,seclabel)
/dev/sda4 on /etc/resolv.conf type ext4 (rw,relatime,seclabel,data=ordered)
/dev/sda4 on /etc/hostname type ext4 (rw,relatime,seclabel,data=ordered)
/dev/sda4 on /etc/hosts type ext4 (rw,relatime,seclabel,data=ordered)
/dev/sda4 on /var/lib/pulp type ext4 (rw,relatime,seclabel,data=ordered)
/dev/sda4 on /var/www type ext4 (rw,relatime,seclabel,data=ordered)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
proc on /proc/sys type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/sysrq-trigger type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/irq type proc (ro,nosuid,nodev,noexec,relatime)
proc on /proc/bus type proc (ro,nosuid,nodev,noexec,relatime)
tmpfs on /proc/kcore type tmpfs (rw,nosuid,context="system_u:object_r:svirt_sandbox_file_t:s0:c585,c908",mode=755)
```

So there is a filesystem mounted on those two mount points. But what's in them? 

```
Default Volume Contents

sh-4.2# find /var/www
/var/www
/var/www/pub
/var/www/html
/var/www/cgi-bin
 
sh-4.2# find /var/lib/pulp
/var/lib/pulp
/var/lib/pulp/published
/var/lib/pulp/published/yum
/var/lib/pulp/published/yum/https
/var/lib/pulp/published/yum/http
/var/lib/pulp/published/puppet
/var/lib/pulp/published/puppet/https
/var/lib/pulp/published/puppet/http
/var/lib/pulp/uploads
/var/lib/pulp/celery
/var/lib/pulp/static
/var/lib/pulp/static/rsa_pub.key
```

That's what it looks like on the inside. But what's the view from outside? I can find out using [docker inspect](https://docs.docker.com/reference/commandline/cli/#inspect).

```
Docker Volume Configuration

docker inspect --format '{{.Config.Volumes}}' volume-demo
map[/var/lib/pulp:map[] /var/www:map[]]
```

First I ask what the volume configuration is for the container.  That result tells me that I didn't provide any mapping for the two volumes.  Next I check what volumes are actually provided.

```
Docker Volume Information

docker inspect --format '{{.Volumes}}' volume-demo
map[
   /var/lib/pulp:/var/lib/docker/vfs/dir/3a11750bd3c31a8025f0cba8b825e568dafff39638fa1a45a17487df545b0f6a
   /var/www:/var/lib/docker/vfs/dir/0a86bd1085468f04feaeb47cc32cfdb0c05fd10e5c7b470790042107d9c02b70
]
```

These are the volumes that are actually mounted on the container. I can see that /var/lib/pulp and /var/www have something mounted on them and that the volumes are actually stored in the host filesystem under /var/lib/docker/vfs/dir. Graphically, here's what that looks like:


![alt](http://resource.docker.cn/pulp-content-container-no-external-volumes-1.png)

So now I have a container running with some storage that is, in a sense "outside" the container.  I need to mount that same storage into another container. This is where the Docker --volumes-from option picks up.

## Shared Volumes in Docker


Once a container exists with marked volumes it is possible to mount those volumes into other containers. Docker provides an option which allows all of the volumes from an existing container to be mounted one-to-one into another container.

For this demonstration I'm going to just create another container from the pulp-content image, but this time I'm going to tell it to mount the volumes from the existing container:

```
A container using --volumes-from

docker run -it --name volumes-from-demo --volumes-from volume-demo markllama/pulp-content /bin/sh
```

If you're following along you can use mount to show the internal mount points, and observe that they match those of the original container. From the outside I can use docker inspect to show that both containers are sharing the same volumes.

```
A container using --volumes-from

docker inspect --format '{{.Volumes}}' volume-demo
map[
 /var/lib/pulp:/var/lib/docker/vfs/dir/3a11750bd3c31a8025f0cba8b825e568dafff39638fa1a45a17487df545b0f6a
 /var/www:/var/lib/docker/vfs/dir/0a86bd1085468f04feaeb47cc32cfdb0c05fd10e5c7b470790042107d9c02b70
]
 
docker inspect --format '{{.Volumes}}' volumes-from-demo
map[
 /var/lib/pulp:/var/lib/docker/vfs/dir/3a11750bd3c31a8025f0cba8b825e568dafff39638fa1a45a17487df545b0f6a
 /var/www:/var/lib/docker/vfs/dir/0a86bd1085468f04feaeb47cc32cfdb0c05fd10e5c7b470790042107d9c02b70
]
```

These two containers have the same filesystems mounted on their declared volume mount points.

## Shared Storage and Pulp

The next two images I need to create for a Pulp service are going to require shared storage.  The Pulp worker process places files in `/var/lib/pulp` and symlinks them into `/var/www` to make them available to the web server.  The Apache server needs to be able to read both the web repository in `/var/www` and the Pulp content in `/var/lib/pulp` so that it can resolve the symlinks and serve the content to clients. I can build the images using the VOLUME directive to create the "windows" I need and then use a content image to hold the files.  Both the worker and apache containers will use the `--volumes-from` directive to mount the storage from the content container.

Here's what that will looks like in Docker:

![alt](http://resource.docker.cn/pulp-worker-apache-content-1.png)

The content container will be created first.  The content image uses the pulp-base as its parent so the file structure, ownership and permissions for the volume content will be initialized correctly.  The worker and Apache containers will get their volumes from the content container.

## Summary


In this post I learned what Docker does with a VOLUME directive if no external volume is provided for the container at runtime.  I also learned how to share storage between two or more running containers.

In the next post I'll show [how to mount (persistent) host storage into a container](http://cloud-mechanic.blogspot.com/2014/10/storage-concepts-in-docker-persistent.html).

In the final post before going back to building a Pulp service I'll demonstrate how to create a pod with storage shared between the containers and if there's space, how to mount host storage into the pod as well.

## References

- [Docker](http://www.docker.com/)
	- Dockerfile [VOLUME directive](https://docs.docker.com/reference/builder/#volume)
	- Docker CLI [run command](https://docs.docker.com/reference/commandline/cli/#run) (see --volume and --volumes-from options)
	- Docker CLI [inspect command](https://docs.docker.com/reference/commandline/cli/#inspect)
- [Pulp](http://pulpproject.org/)

---

Original source: [Storage Concepts in Docker: Shared Storage and the VOLUME directive](http://cloud-mechanic.blogspot.jp/2014/10/storage-concepts-in-docker.html)