# Copying Data From One Docker Container to Another

---

Author: Sudhi Seshachala 

---

Docker containers can easily be compared to a directory. A Docker container holds everything that is required for an application to run including its dependencies and can be started, stopped, moved and deleted at will. When containers are activated they use Linux kernel namespaces. Each container operates in isolation from the server apart from networking and storage. Each container is created from a Docker image and one can have multiple containers running, depending upon the server hardware capacity. Docker containers are the run component of Docker.

## What are Docker images?

Each Docker container is created from a Docker image, which can be built either by executing commands **manually** or automatically through **Dockerfiles**. Images can be basic either consisting of the operating-system fundamentals or a pre-built stack ready for launch. They are read only templates and consist of series of layers, so any new action or update leads to the addition of new layer on top of the previous one rather than replacing whole image entirely. This is how a new container is made.

## Copying data from one Docker Container to another:

Copying of data from one container to another is essential and most important feature of Docker containers. Copying data to another container means having a backup that could be used if any kind of disaster occurs out of any situation in real time scenario.

## Using Data Volume Containers:

Data volume is a directory available to a container but it is located outside of its root file system. Sharing data between the containers allows volume to bypass image layering. Any changes in the data are made directly, containers have dedicated directories in the `/var/lib/docker` directory and volumes are stored separately `in/var/lib/docker/volumes/` by default. `–v` option of docker run can be used to copy volume data between the two containers.

## How to copy content from one container to another `-An` Illustration

In this example we assume that Docker is running two instances of data volume container image `mymod/dvc: v1`. Following are the commands that are used to start the container

```
# docker run –d –name dvc1 mymod/dvc:v1
```

```
# docker run –d –name dvc2 mymod/dvc:v1
```

Now to copy data to the host and to even mount the volume from another container, use `cp` command to copy data

```
[root@host ~]# docker run –rm –v /var/tmp:/host:rw  \ –volumes- from dvc1 cp –r/var/www/html/host/dvc1_files
```

The container mounts the host directory `/var/tmp` read-writable as `/host`, mounts all the volumes, including `/var/www/html`, that `dvc1` exports, and copies the file hierarchy under `/var/www/html` to `/host/dvc1_files`, which corresponds to `/var/tmp/dvc1_files` on the host.

Now to copy data from dvc1 host to another data volume container we use following command

```
[root@host ~]# docker run –rm –v /var/tmp:/host:ro –volumes-from dvc2 \ cp –a –T /host/dvc1_files /var/www/html
```

The container mounts the host directory `/var/tmp read-only` as `/host`, mounts the volumes exported by dvc2, and copies the file hierarchy under `/host/dvc1_files` (`/var/tmp/dvc1_files` on the host) to `/var/www/html`, which corresponds to a volume that dvc2 exports.

Another method with which one can copy data is `ADD` command, a Dockerfile command. Source and destination form the two arguments for `ADD` command to run successfully. It copies the files or data from the source container into the destination container with its own filesystem.

## Conclusion

Docker will certainly expose nicer ways to grow container service to support wider deployment platforms and the community is surely expected to build service layers around. Development of core capabilities, cross service management and messaging between the containers are the paths set by Docker. We’ve tried to demonstrate via this article that data can be copied and replicated in another Docker container thus can be used in any case of disaster as well as recovery. Docker provides the ability to easily differ containers. It improves debugging, fast sharing of the environment and quick deployment. Docker has an ability to put the containers directly from Dev or QA onto AWS. User can share data through volume containers, mount host directories to containers or even can export data from containers to archive files thus very flexible.

---

Original source: [Copying Data From One Docker Container to Another](http://devops.com/blogs/devops-toolbox/copying-data-one-docker-container-another/)