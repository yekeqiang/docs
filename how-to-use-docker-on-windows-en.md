# How to Use Docker on Windows

---

##### Author: Maxime Heckel

---

![alt](http://resource.docker.cn/how-to-use-docker-windows.jpg)

## Prerequisites

To follow this tutorial, you need a working installation of Windows 7 or 8 on your computer.

## How Docker works and how to make it work on windows

Before jumping into the installation process, let’s see how Docker works in order to understand what we’ll need to do to make it work on Windows.
We must first view Docker as both a server and a client.

The server lets us build, download, start and stop images or containers.
The client is just a command line tool that will allow us to communicate with the API of the server.

Here is what it looks like on Linux :

![alt](http://resource.docker.cn/diagram-linux.png)

The purpose of this tutorial is to use Windows to run both our Docker client and server; therefore using it as a Docker host. To do that we are going to use [Boot2docker](http://boot2docker.io/)

### Boot2docker

Since we aren’t using Linux, it is not possible ([yet](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/)) to use Docker natively. We are going to need some sort of lightweight VM that emulates a Docker Host.

This is what Boot2docker is for.

> Boot2docker is a lightweight Linux distribution based on Tiny Core Linux made specifically to run Docker containers. It runs completely from RAM, weighs ~27MB and boots in ~5s

*Source : [boot2docker.io](http://boot2docker.io/)*

Basically, Boot2Docker will encapsulate our Docker server into a virtual machine and let us access it through the Windows Docker client.

![alt](http://resource.docker.cn/diagram-win.png)

## Installation

- First, download and run the latest version of Boot2docker [here](https://github.com/boot2docker/windows-installer/releases/tag/v1.3.0). The Windows version of Boot2docker contains the Docker for Windows installer. It will install a set of tools (VirtualBox, Boot2docker ISO, MSYS-git UNIX tools) to help us run Docker.

- We now have a new shortcut on the Desktop called **Boot2Docker Start**. This shortcut is a shell script that will initiate the Boot2docker virtual machine and also the docker client. It will ask you for a ssh passphrase; you can either type one or just hit enter.

	Boot2docker then starts initializing the VM and the Docker client. However, unlike Docker on OSX, we can’t use the Docker client directly from Windows but only from the Boot2docker VM.
	
	We are now logged into the boot2docker VM and ready to use Docker.

## Running Docker containers

Now that the install process is finally finished, we have the ability to launch containers, pull images, etc. as if we were using Docker on Linux.

First, type :

```
docker
```

to ensure that everything is working.

Boot2docker comes with a sample image named **hello-world** that lets new users quickly launch a container. Let’s give it a try :

```
docker run hello-world
```

We should see a “hello world” message printed on the screen.

### Port Redirection

Redirecting ports with Boot2docker is really easy. The command is the same as on Linux, i.e. by using the -p flag like this :

```
docker run -p PUBLIC_PORT:PRIVATE_CONTAINER_PORT IMAGE CMD
```

For instance, if we wanted to run the redis image on port 6379, we would run :

```
docker run -p 6379:6379 redis
```

The only thing that differs from running this command on Windows instead of Linux is that we will need the Boot2docker IP address in order to access that redis server.

In order to get it we can run the following command :

```
boot2docker ip
```

### Sharing a Windows folder

This part will explain how to properly set up a shared folder between Windows and Boot2docker. However, as this feature is not officially included we will need to install additional tools.

First, we need to get cifs-utils on the Boot2docker VM

```
wget http://distro.ibiblio.org/tinycorelinux/5.x/x86/tcz/cifs-utils.tcz
 
tce-load -i cifs-utils.tcz
```

Then, we create a folder

```
mkdir /mnt/sharefolder
```

This will be used as a mount point for our shared folder.

We will then need to create and share a new folder on Windows. To keep it simple, let’s just share the folder with ourselves.

Finally, we mount the shared folder on the Boot2docker VM

```
sudo mount -t cifs //WINDOWS_IP/shared /mnt/sharefolder -o username=WINDOWS_USERNAME
```

We will be asked to enter our Windows password and the folder will be mounted.

This will allow us to create volume containers (instructions [here](https://github.com/boot2docker/boot2docker#folder-sharing)) or share Dockerfiles between Windows and the VM for instance.

## Further details on Boot2docker

### User and password

We might need the username and the password of the Boot2docker user. By default the username is docker and the password is tcuser.

### Upgrade Boot2docker

In order to upgrade Boot2docker:

- Download the latest version of Docker for Windows and run the installer, after which the Boot2docker management tool is updated.

- Upgrade the VM

```
boot2docker stop
 
boot2docker download
 
boot2docker start
```

## Conclusion

The main goal of this tutorial was to explain how to install a working Docker environment on Windows using Boot2docker. However, with the recent announcements regarding Docker Windows Support on Azure and Windows Server, a cleaner way to run Docker without VMs may arise.

---

Original source: [How to Use Docker on Windows](http://blog.tutum.co/2014/11/05/how-to-use-docker-on-windows/)