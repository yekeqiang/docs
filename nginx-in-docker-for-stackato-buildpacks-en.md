# Nginx in Docker for Stackato buildpacks

---

##### Author: Daniel Watrous

---

I’m about to write a few articles about creating buildpacks for Stackato, which is a derivative of CloudFoundry and the technology behind [Helion Development Platform](http://www8.hp.com/us/en/cloud/helion-devplatform-overview.html). The approach for [deploying nginx in docker](http://software.danielwatrous.com/use-docker-to-build-a-lemp-stack-buildfile/) as part of a buildpack differs from the approach I published previously. There are a few reasons for this:

- Stackato discourages root access to the docker image
- All services will run as the stackato user
- The PORT to which services must bind is assigned and in the free range (no root required)

## Get a host setup to run docker

The easiest way to follow along with this tutorial is to [deploy stackato somewhere like hpcloud.com](http://software.danielwatrous.com/install-stackato-cloudfoundry-on-hpcloud/). The resulting host will have the docker image you need to follow along below.

### Manually prepare a VM to run docker containers

You can also use Vagrant to spin up a virtual host where you can run these commands.

```
vagrant init ubuntu/trusty64
```

Modify the Vagrantfile to contain this line

```
  config.vm.provision :shell, path: "bootstrap.sh"
```

Then create a the bootstrap.sh file based on the details below.

**bootstrap.sh**

```
#!/usr/bin/env bash
 
# set proxy variables
#export http_proxy=http://proxy.example.com:8080
#export https_proxy=https://proxy.example.com:8080
 
# bootstrap ansible for convenience on the control box
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
apt-get update
apt-get -y install lxc-docker
 
# need to add proxy specifically to docker config since it doesn't pick them up from the environment
#sed -i '$a export http_proxy=http://proxy.example.com:8080' /etc/default/docker
#sed -i '$a export https_proxy=https://proxy.example.com:8080' /etc/default/docker
 
# enable non-root use by vagrant user
groupadd docker
gpasswd -a vagrant docker
 
# restart to enable proxy
service docker restart
# give docker a few seconds to start up
sleep 2s
 
# load in the stackato/stack-alsek image (can take a while)
docker load < /vagrant/stack-alsek.tar
```

### Get the staccato docker image

Before the last line in the above bootstrap.sh script will work, it’s necessary to **place the docker image for Stackato in the same directory as the Vagrantfile**. Unfortunately the Stackato docker image is not published independently, which makes it more complicated to get. One way to do this is to [deploy stackato locally](http://software.danielwatrous.com/explore-cloudfoundry-using-stackato-and-virtualbox/) and grab a copy of the image with this command.

```
docker save stackato/stack-alsek > stack-alsek.tar
```

You might want to save a copy of stack-alsek.tar to save time in the future. I’m not sure if it can be published (legally), but you’ll want to update this image with each release of Stackato anyway.

### Launch a new docker container using the stackato image

You should now have three files in the directory where you did ‘vagrant init’.

```
-rw-rw-rw-   1 user     group          4998 Oct  9 14:01 Vagrantfile
-rw-rw-rw-   1 user     group           971 Oct  9 14:23 bootstrap.sh
-rw-rw-rw-   1 user     group    1757431808 Oct  9 14:02 stack-alsek.tar
```

At this point you should be ready to create a new VM and spin up a docker instance. First tell Vagrant to build the virtual server.

```
vagrant up
```

Next, log in to your server and create the docker container.

```
vagrant@vagrant-ubuntu-trusty-64:~$ docker run -i -t stackato/stack-alsek:latest /bin/bash
root@33ad737d42cf:/#
```

## Build and configure your services

Once you have a system setup and can create docker instances based on the Stackato image, you’re ready to craft your buildpack compile script. One of the first things I do is install the w3m browser so I can test my setup. In this example, I’m just going to build and test nginx. The same process could be used to build any number of steps into a compile script. It may be necessary to manage http_proxy, https_proxy and no_proxy environment variables for both root and stackato users while completing the steps below.

```
apt-get -y install w3m
```

Since everything in a stackato DEA is expected to run as the stackato user, we’ll switch to that user and move into the home directory

```
root@33ad737d42cf:/# su stackato
stackato@33ad737d42cf:/$ cd
stackato@33ad737d42cf:~$ pwd
/home/stackato/
```

Next I’m going to grab the source for nginx and configure and make.

```
wget -e use_proxy=yes http://nginx.org/download/nginx-1.6.2.tar.gz
tar xzf nginx-1.6.2.tar.gz
cd nginx-1.6.2
./configure
make
```

By this point nginx has been built successfully and we’re in the nginx source directory. Next I want to update the configuration file to use a non-privileged port. For now I’ll use 8080, but Stackato will assign the actual PORT when deployed.

```
mv conf/nginx.conf conf/nginx.conf.original
sed 's/\(^\s*listen\s*\)80/\1 8080/' conf/nginx.conf.original > conf/nginx.conf
```

I also need to make sure that there is a logs directory available for nginx error logs on startup.

```
mkdir logs
```

It’s time to [test the nginx build](http://wiki.nginx.org/CommandLine), which we can do with the command below. A message should be displayed saying the test was successful.

```
stackato@33ad737d42cf:~/nginx-1.6.2$ ./objs/nginx -t -c conf/nginx.conf -p /home/stackato/nginx-1.6.2
nginx: the configuration file /home/stackato/nginx-1.6.2/conf/nginx.conf syntax is ok
nginx: configuration file /home/stackato/nginx-1.6.2/conf/nginx.conf test is successful
```

With the setup and configuration validated, it’s time to start nginx and verify that it works.

```
./objs/nginx -c conf/nginx.conf -p /home/stackato/nginx-1.6.2
```

At this point it should be possible to load the nginx welcome page using the command below.

```
w3m http://localhost:8080
```
![alt](http://resource.docker.cn/nginx-docker.png)

## Next steps

If an application requires other resources, this same process can be followed to build, integrate and test them within a running docker container based on the stackato image.

---

Original source: [Nginx in Docker for Stackato buildpacks](http://software.danielwatrous.com/nginx-in-docker-for-stackato-buildpacks/)