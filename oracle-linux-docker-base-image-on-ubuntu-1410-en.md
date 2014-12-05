# Oracle Linux Docker Base Image on Ubuntu 14.10

---

Author: Bruno.Borges, Nov 28, 2014

---

Oracle Linux team has been working hard to provide Docker support. You will be glad to see that they, since August, are [releasing Docker binaries in Oracle Linux YUM repositories](https://blogs.oracle.com/linux/entry/ahoy_cast_off_with_docker). Now recently they are also publishing the [Oracle Linux Docker Base Images for OL6 and OL7](https://blogs.oracle.com/linux/entry/oracle_linux_images_for_docker)! 

*The documentation of Oracle Linux ([OL6](https://docs.oracle.com/cd/E37670_01/E37355/html/ol_docker.html) and [OL7](https://docs.oracle.com/cd/E52668_01/E54669/html/ol7-docker.html)) and Docker is also well advanced and explanatory. Keep track of everything about Docker and Oracle Linux from [this yum page](http://public-yum.oracle.com/docker-images/).*

And why is this important you may ask? Well because it's definitely a required step if at some point the Fusion Middleware product teams move forward in certifying their products, such as WebLogic, on Docker as well. So if you want to work with Oracle products on Docker right now, although yet not certified nor supported, make sure you use the OL base images.

As a developer, I like to use Ubuntu Linux on my laptop. To use the OL7 Docker Base Image for example, I had to follow the following very short and easy steps:

1. Make sure you have [Docker installed on your Ubuntu](https://docs.docker.com/installation/ubuntulinux/) environment

2. Download the [OL7 Base Image](http://public-yum.oracle.com/docker-images/OracleLinux/OL7/).

3. Uncompress the file first with xz. You may find an issue where 
`docker` can't find the xz binary.

```
$ sudo docker load -i oraclelinux-7.0.tar.xz
2014/11/28 18:36:08 Error: Untar exit status 1 exec: "xz": executable file not found in $PATH

$ unxz oraclelinux-7.0.tar.xz
```

4. Load the image in your local Docker repository, as `root`, after extracting with xz (but keeping `tar`)

```
$ sudo docker load -i oraclelinux-7.0.tar
```

5. Check installation

```
$ sudo docker images
REPOSITORY     TAG  IMAGE ID       CREATED      VIRTUAL SIZE
oraclelinux    7.0  5f1be1559ccf   2 weeks ago    265.2 MB
```

6. Create and run a container based on this image

```
$ sudo docker run -t -i oraclelinux:7.0 bash
```

Now have fun with Docker and Oracle Linux, and let me know if you create something cool with Fusion Middleware products!


---

Original source: [Oracle Linux Docker Base Image on Ubuntu 14.10](https://blogs.oracle.com/brunoborges/entry/oracle_linux_docker_base_image)