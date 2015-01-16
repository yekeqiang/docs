# Docker in 5 minutes on RunAbove with Docker Machine

---

Author: Vincent Giersch

---

Docker is a great tool to easily deploy your applications. Docker Inc. recently introduced a new project, Docker Machine, that allows you to easily setup multiple Docker hosts across multiple cloud providers, and use them with your local Docker client.

In this guide you'll discover how to use Docker Machine on RunAbove. This is an experiment: Docker machine uses a development branch of Docker implementing a TLS encryption and a identity-based authentication and the OpenStack driver is still in development.

This guide will be updated following the evolutions of Docker Machine and its OpenStack driver.

## Install Docker Machine command line
 
We provide you built binaries of Docker Machine with the OpenStack driver at its current development state:

- Linux amd64
- Linux i386
- Linux arm
- Windows i386
- Windows amd64
- Mac OSX i386
- Mac OSX amd64

If you want to build Docker Machine with the OpenStack driver by yourself, the details are available in the README of the project.

## Install Docker client with identity authentication support

Your Docker client will use an identity-based authentication to communicate with the Docker daemon that will be installed on your instance. Builds by Ben Firshman of Docker Inc. are available here:

- Mac OS X
- Linux

If you want to build Docker on this branch, the build instructions are available here and the pull request #8265 here.

## Getting started with Docker Machine on RunAbove

### Get your OpenStack credentials

Set your credentials in your environment using the Open RC file that you can download using the horizon dashboard.

```
$ source XXXXXXX-openrc.sh
```

You need to set the availability zone where you want to deploy your new instance (SBG-1 or BHS-1) using the following environment variable:

```
$ export OS_REGION_NAME=SBG-1
```

### Create your first Docker Machine: a RunAbove Instance

Deploying a new instance with a Docker daemon is now as simple as a simple command:

```
$ machine create \
  -d openstack \
  --openstack-flavor-name="ra.intel.ha.s" \
  --openstack-image-name="Ubuntu 14.04" \
  --openstack-net-name="Ext-Net" \
  --openstack-ssh-user="admin" \
  my-docker-host
```

In the example above, we use a Steadfast Resources S instance as flavor and an Ubuntu 14.04 as image. All the flavor and image available are listed in your Expert dashboard.

Once deployed, you just need to declare to your Docker client that you'll use your fresh installed Docker daemon on your RunAbove instance:

```
$ export DOCKER_HOST=$(machine url) DOCKER_AUTH=identity
```

That's all, you can now use your Docker Machine with your local Docker client:

```
$ docker info
The authenticity of host "1.2.3.4:2376" can't be established.
Remote key ID 6D2O:BGXV:I3AE:WH7W:OIRG:JPVU:EJET:UD7H:TASU:EFAU:CIJG:HEIP
Are you sure you want to continue connecting (yes/no)? yes
Containers: 0
Images: 3
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Dirs: 3
Execution Driver: native-0.2
Kernel Version: 3.13.0-37-generic
Operating System: Ubuntu 14.04.1 LTS
CPUs: 1
Total Memory: 1.955 GiB
Name: vm
ID: 6D2O:BGXV:I3AE:WH7W:OIRG:JPVU:EJET:UD7H:TASU:EFAU:CIJG:HEIP
WARNING: No swap limit support
```

```
$ docker run hello-world
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
```

Feel free to join the discussion about this OpenStack driver on the ongoing pull request, and to post on RunAbove Community if you have any question or comment about using Docker on RunAbove.

---

Original source: [Docker in 5 minutes on RunAbove with Docker Machine](https://community.runabove.com/kb/en/instances/docker-in-5-minutes-on-runabove-with-docker-machine.html) 
