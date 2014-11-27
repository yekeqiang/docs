# Building a Bitnami Tomcat Image using Docker

---

Author: Gilbert Pilz

---

I am a long-time fan of [Bitnami's prepackaged stacks](https://bitnami.com/stacks). If you want to, for example, quickly stand up a new [Drupal](https://www.drupal.org/) instance, Bitnami allows you to do this - using either a machine image with the stack pre-installed or a binary installer that you can run on the appropriate type of OS.

When I first learned about Docker, I thought of Bitnami and how it seemed a natural fit for them to offer Docker image versions of their stacks. It turns out that they are in the process of doing exactly [that](https://bitnami.com/docker). However, at the time of this writing, they don't have these available, so I decided to build my own. What follows is a step by step recipe for taking the Bitnami Tomcat 7 installer and building a Docker image that captures the result of a successful install.


## Step 0 - Create a VM and install Docker

I did this in a single step using Digital Ocean's ability to select OS / application combos - in this case Docker 1.3.1 on Ubuntu 14.04 (64 bit). To keep Bitnami's installer from complaining about memory (in Step 4) you are going to need at least a 2 GB VM. If you want to run multiple stacks side-by-side on the same VM, you are going to need at least 4GB.

## Step 1 - Download the Bitnami Tomcat Installer onto Your VM

The easiest way to do this is use 'wget' on the VM:

```
root@vm:~# mkdir bitnami; cd bitnami
root@vm:~# wget https://bitnami.com/redirect/to/45854/bitnami-tomcatstack-7.0.57-0-linux-x64-installer.run
root@vm:~# chmod +x *.run
```

Note that Bitnami is always updating their downloads so, by the time you read this, the installer above may not be available. Just use the appropriate installer for your OS. Obviously you can also choose to use an earlier or later version of Tomcat.

Also note the I've saved the installer under a new directory (which we will reference in Step 3) and made it executable.

## Step 2 - Download/Pull the Base Docker Image

Working with Docker is like baking sourdough bread; you need a little something to start with. I chose to use Docker's base Ubuntu image because (a) I really don't care which OS I'm running Tomcat on, and (b) I've used Bitnami's Tomcat stack on Ubuntu before and never had any problems.

```
root@vm:~# docker pull ubuntu
```

You should see a brief flurry of activity ending with:

```
Status: Downloaded newer image for ubuntu:latest
```

## Step 3 - Start a Container

First I'll show you the command, then I'll explain the options:

```
root@vm:~# docker run --cap-add=ALL -i -p 80:80 -t -v /root/bitnami:/bitnami ubuntu /bin/bash
```

`--cap-add=ALL`: When it starts, Tomcat tries to set some [capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html) (i.e. establish the privilege to do one or more "superuser like" things). By default Docker does not allow processes within a container to do this. This option allows processes within the container to set any capability they want. This is a sloppy and dangerous thing to do. I should dig into the Tomcat code and figure out exactly which capabilities it is requesting and grant only those capabilities (see the "[principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)").

`-v /root/bitnami:/bitnami`: This option bind mounts "/root/bitnami" on the VM to "/bitnami" in the container. This will allow us to access the installer file from inside the container.

`-p 80:80`: By default the Apache web server listens on port 80. This option maps port 80 of the container to port 80 on our VM. Obviously you can map the container port to any free port on your VM (e.g 8080 using "-p 8080:80").

`-i, -t`: These two options connect you to the shell running inside the container.

`ubuntu`: This option specifies the image to run in the container. In this case it is the default Ubuntu image that we pulled in Step 2.

`/bin/bash`: This option tells Docker to run a bash shell inside the container.

At this point you should find yourself at a container-level prompt like:

```
root@d10f70897ce3:/# 
```

## Step 4 - Run the Bitnami Installer

Next we want to run the Tomcat installer to install Apache, Tomcat, and MySQL into our container:

```
/bitnami/bitnami-tomcatstack-7.0.57-0-linux-x64-installer.run --mode unattended
```

This command will take a couple of minutes to complete, so be patient. If all goes well you should return to the container-level prompt where you can poke around a bit to check things out. A "`ps -ef`" should show you the Apache, MySQL, and Tomcat processes, there should be an "`/opt/tomcatstack-7.0.57-0 directory`", etc. You can test whether apache is up and accessible by browsing to "`http://<your VM address>/`". You should see the welcome page for the Bitnami Tomcat stack.

Note that the way in which we installed Apache, MySQL, and Tomcat is extremely unsafe. For example, there is no password for the Tomcat manager application. Under this configuration it should only be a matter of minutes before someone installs something unpleasant onto Tomcat. The Bitnami installer supports a number of command-line options for setting the MySQL password, the Tomcat manager password, etc. You can play around with these to get the configuration you want. This is where Docker shines; you can quickly re-run Steps 3 and 4 to experiment with different configurations. One thing to be aware of is that Docker saves containers after you exit them so, to avoid confusion, you should probably "`docker rm <container-id>`" on any containers you are no longer interested in.

## Step 5 - Snapshot the Container

Now that you have a container running a configuration of the Tomcat stack that you are happy with, it is time to snapshot that container and create a Docker image. Since we started the Apache, MySQL, and Tomcat processes from the bash shell that we launched on container startup, exiting the shell will cause these processes to terminate. I confess to being somewhat superstitious, however, so I prefer to shut down these processes in the "proper" manner:

```
root@d10f70897ce3:/# /opt/tomcatstack-7.0.57-0/ctlscript.sh stop
```

After this completes you can simply exit the bash shell to exit the container and return to your VM-level shell. At this point we can snapshot the container and create a new image using the "docker commit" command like so:

```
root@vm:~# docker commit -m="Some pithy comment." d10f70897ce3 mybitnami/tomcat:v1
```

The resulting image should be viewable through the "docker images" command.

## Step 6 - Launching the Image

Launching our newly created image is simply a matter of starting a container using that image:

```
root@vm:~# docker run --cap-add=ALL -d -p 80:80 mybitnami/tomcat:v1 /bin/sh -c "/opt/tomcatstack-7.0.57-0/ctlscript.sh start; tail -F /opt/tomcatstack-7.0.57-0/apache-tomcat/logs/catalina-daemon.out"
```

This looks a little intimidating, so let's break it down. The "--cap-add=ALL" option was covered in Step 3. We still need this because Tomcat still sets the same capabilities. The "-d" option simply tells Docker to run the container in the background. We've eliminated the "-i" and "-t" options because we don't need to interact directly with the container. The "-p 80:80" options specifies the same port mapping and we've eliminated the "-v" option because we no longer need to access any host files from the container. What makes this step look complicated is the in-line shell script at the end. What we are telling Docker to do is run the following commands in a shell:

```
/opt/tomcatstack-7.0.57-0/ctlscript.sh start
tail -F /opt/tomcatstack-7.0.57-0/apache-tomcat/logs/catalina-daemon.out
```

Docker will run a shell that executes "ctlscript.sh start" thus starting Apache, MySQL, and Tomcat. It will then run the "tail" command on the main Tomcat log file, blocking on additional writes to this file. What this means is that the shell process that is the parent or grandparent of all the Apache, MySQL, and Tomcat processes will continue to run, thus keeping the whole tree of processes alive.

There are a number of ways we can monitor our container at this point. We can view a top-like display of the processes in the container via:

```
root@vm:~# docker top <container ID>
```

We can look at containers STDOUT and STDERR using:

```
root@vm:~# docker logs <container ID>
```

## Step 7 - Stopping the Container

To stop the container running our tomcat stack we can send the SIGTERM signal to the root process of the container (our shell running "tail") via:

```
root@vm:~# docker stop <container ID>
```

This should cause all of the server processes to shut down cleanly. As I mentioned, I'm a bit superstitious about these things so I would prefer a mechanism that invoked "`ctlscript.sh stop`" before exiting the container. I've spent enough time investigating to determine that this is a subject for another post.

---

## Some Questions

### Why Not Use an Existing Tomcat Image?

If you are familiar with Docker you are probably aware that there are plenty of [existing images](https://registry.hub.docker.com/search?q=tomcat) that run tomcat. Why not simply use one of these? Firstly, none of these images (that I am aware of) include an integrated Apache or, more importantly, MySQL. Secondly, I am working with an application that I built using the Bitnami stack and I'm comfortable dinking with this stack. It is less work for me to build an image of my existing system than it is to switch to a new system.

## Why Not Use "docker build"?

Steps 3-5 could have been replaced using the "docker build" command and a Docker file. However, at the time of this writing, the containers used during the "docker build" command do not allow their processes to request capabilities. A

```
RUN bitnami-tomcatstack-7.0.56-0-linux-x64-installer.run
```

command will fail with the following error:

```
set_caps: failed to set capabilities
check that your kernel supports capabilities
set_caps(CAPS) failed for user 'tomcat'

Service exit with a return value of 4
```

when Tomcat tries to run for the first time. This issue is being tracked by Docker here: [https://github.com/docker/docker/issues/1916](https://github.com/docker/docker/issues/1916).

### Why Use Docker At All?

At the beginning of this post I pointed out that Bitnami stacks exist in machine image form for most popular systems. I can go to AWS and, in less time and less effort, create a new VM that is functionally equivalent to the docker container that I have created here. Some points:

- My Bitnami Tomcat stack Docker image is a just a building block. Next I'm going to install a webapp on Tomcat, a database on MySQL, etc. Then I'm going to snapshot that. Again, I could do the same with AWS, but I can't run an AMI anywhere besides AWS. I can take my Docker images and run them on anything with a compatible kernel.
- When saved as a TAR file my docker image is approximately 800 Mb. Most VM images are far larger than this. Lighter is faster.
Bitnami does a great job with integration but nothing is ever quite exactly the way you want it. The dink-->test-->dink-some-more cycle in Steps 3 and 4 is much faster using containers on an individual VM than using multiples VMs.
- If, for whatever reason, I wanted to run multiple instances/versions of my stack it would probably be much cheaper to run them side-by-side in separate containers on the same (larger) VM than it would be to run them each in their own (smaller) VMs. This cost difference is even greater if I decide that I need to make my stacks available at a static IP address and/or given DNS name.

---

Original source: [Building a Bitnami Tomcat Image using Docker](http://recursivedigressions.blogspot.fr/2014/11/building-bitnami-tomcat-image-using.html)