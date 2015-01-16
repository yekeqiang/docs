# Understanding Volumes in Docker

---

Author: Adrian Mouat

---

It’s clear from looking at the questions asked on the docker IRC channel (#docker on Freenode) and [stackoverflow](https://stackoverflow.com/questions/tagged/docker) that there’s a lot of confusion over how volumes work in Docker. In this post I’ll try to explain how volumes work and present some best practices. Whilst this post is primarily aimed at Docker users with little to no knowledge of volumes, even experienced users are likely to learn something as there are several subtleties regarding volumes that many people aren’t aware of.


In order to understand what a Docker volume is, we first need to be clear about how the filesystem normally works in Docker. Docker images are stored as series of read-only layers. When we start a container, Docker takes the read-only image and adds a read-write layer on top. If the running container modifies an existing file, the file is copied out of the underlying read-only layer and into the top-most read-write layer where the changes are applied. The version in the read-write layer hides the underlying file, but does not destroy it — it still exists in the underlying image. When a Docker container is deleted, relaunching the image will start a fresh container without any of the changes made in the previously running container — those changes are lost. Docker calls this combination of read-only layers with a read-write layer on top a [Union File System](https://docs.docker.com/terms/layer/#union-file-system).

In order to be able to save (persist) data and share data between containers, Docker came up with the concept of volumes. Quite simply, volumes are directories (or files) that are outside of the default Union File System and exist as normal directories and files on the host filesystem.

There are two ways to initialise volumes, with some subtle differences that are important to understand. We can declare a volume at run-time with the -v flag:


```
$ docker run -it --name container-test -h CONTAINER -v /data debian /bin/bash
root@CONTAINER:/# ls /data
root@CONTAINER:/# 
```

This will make the directory /data inside the container live outside the Union File System and directly accessible on the host. Any files that the image held inside the /data directory will be copied into the volume. We can find out where the volume lives on the host by using the docker inspect command on the host (open a new terminal and leave the previous container running if you’re following along):


```
$ docker inspect -f {{.Volumes}} container-test
```

And you should see something like:

```
map[/data:/var/lib/docker/vfs/dir/cde167197ccc3e138a14f1a4f7c0d65b32cecbada822b0db4cc92e79059437a9] 
```

Telling us that Docker has mounted /data inside the container as a directory somewhere under /var/lib/docker. Let’s add a file to the directory from the host:

```
$ sudo touch /var/lib/docker/vfs/dir/cde167197ccc3e138a14f1a4f7c0d65b32cecbada822b0db4cc92e79059437a9/test-file
```

Then switch back to our container and have a look:

```
$ root@CONTAINER:/# ls /data
test-file
```

Changes are reflected immediately as the directory on the container is simply a mount of the directory on the host. We can achieve exactly the same effect by using VOLUME instruction in a Dockerfile:

```
FROM debian:wheezy
VOLUME /data
```

But there’s one more thing the -v argument to docker run can do and that can’t be done from a Dockerfile, and that’s mount a specific directory on the host to the container. For example:

```
$ docker run -v /home/adrian/data:/data debian ls /data
```

Will mount the directory `/home/adrian/data` on the host as /data inside the container. Any files already existing in the `/home/adrian/data` directory will be available inside the container. This is very useful for sharing files between the host and the container, for example mounting source code to be compiled. The host directory for a volume cannot be specified from a Dockerfile, in order to preserve portability (the host directory may not be available on all systems). When this form of the -v argument is used any files in the image under the directory are not copied into the volume.

## Sharing Data

To give another container access to a container’s volumes, we can simply give the `–volumes-from` argument to docker run. For example:


```
$ docker run -it -h NEWCONTAINER --volumes-from container-test debian /bin/bash
root@NEWCONTAINER:/# ls /data
test-file
root@NEWCONTAINER:/#
```

It’s important to note that this works whether container-test is running or not. A volume will never be deleted as long as a container is linked to it.

## Data Containers

It’s common practice to use a data-only container for storing persistent databases, configuration files, data files etc. The [Docker website has some good documentation](https://docs.docker.com/userguide/dockervolumes/) on this. For example:

```
$ docker run --name dbdata postgres echo "Data-only container for postgres"
```

This command will create a postgres image, including the volume defined in the Dockerfile, run the echo command and exit. The echo command is useful in so far as it helps us identify the purpose of the image when looking at docker ps. We can use this volume from other containers with the `–volumes-from` argument e.g:

```
$ docker run -d --volumes-from dbdata --name db1 postgres
```

There are two important points using running data containers:

- Don’t leave the data-container running; it’s a pointless waste of resources
- Don’t use a “minimal image” such as busybox or scratch for the data-container. Just use the database image itself. You already have the image, so it isn’t taking up any additional space and it also allows the volume to be seeded with data from image.

## Backups

If you’re using a data-container, it’s pretty trivial to do a backup:

```
$ docker run --rm --volumes-from dbdata -v $(pwd):/backup debian tar cvf /backup/backup.tar /var/lib/postgresql/data
```

Should create a tarball of everything in the volume (the official postgres Dockerfile defines a volume at `/var/lib/postgresql/data`).

## Permissions and Ownership

Often you will need to set the permissions and ownership on a volume or initialise the volume with some default data or configuration files. The key point to be aware of here is that anything after the VOLUME instruction in a Dockerfile will not be able to make changes to that volume e.g:


```
FROM debian:wheezy
RUN useradd foo
VOLUME /data
RUN touch /data/x
RUN chown -R foo:foo /data
```

Will not work as expected. We want the touch command to run in the image’s filesystem but it is actually running in the volume of a temporary container. The following will work:

```
FROM debian:wheezy
RUN useradd foo
RUN mkdir /data && touch /data/x
RUN chown -R foo:foo /data
VOLUME /data
```

Docker is clever enough to copy any files that exist in the image under the volume mount into the volume and set the ownership correctly. This won’t happen if you specify a host directory for the volume (so that host files aren’t accidentally overwritten).

If you can’t set permissions and ownership in a RUN command, you will have to do so using a CMD or ENTRYPOINT script that runs after container creation.

## Deleting Volumes

This is a bit more subtle than most people realise. Chances are, if you’ve been using docker rm to delete your containers, you probably have lots of orphan volumes lying about taking up space.

Volumes are only deleted under the following circumstances:

- the container was deleted with `docker rm -v` and no other containers were linked to the volume (and no host directory was specified for the volume). Note that the `-v` is essential.
- the `–rm` flag was provided to `docker run`

Unless you’ve been very careful about always running your containers like this, you’re going to have zombie files and directories under /var/lib/docker/vfs/dir and no easy way of telling what they represent.

## Further Reading

The following resources explain resources in more depth and were essential in writing this blog:

- [Data-only Container Madness](http://container42.com/2014/11/18/data-only-container-madness/)
- [Docker In-Depth: Volumes](http://container42.com/2014/11/03/docker-indepth-volumes/)
- [Managing Data in Containers](https://docs.docker.com/userguide/dockervolumes/)

Also, it looks like we can expect to get some more tools for dealing with volumes soon:

- [Proposal #8484](https://github.com/docker/docker/pull/8484)

---

Original source: [Understanding Volumes in Docker](http://container-solutions.com/2014/12/understanding-volumes-docker/)