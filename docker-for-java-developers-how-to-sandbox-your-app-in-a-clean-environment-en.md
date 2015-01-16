# Docker for Java Developers: How to sandbox your app in a clean environment


---

Author: Oleg Shelajev

---

I’ve got a confession: I’ve been putting off looking into [Docker](http://www.docker.com/) for some time, mainly due to lack of time and the fact that ZeroTurnaround, our sponsor and parent, recently decided to get out of the Ops business altogether and [focus on developer tools](http://zeroturnaround.com/rebellabs/5-reasons-the-jvm-development-tools-market-rocks/). As an engineer on our former release automation tool, I tend to get a slightly bad taste when I look at virtualization or giant cluster management tools.

So I’m coming to the Docker game a bit late perhaps, but today I’m going to talk about how Java developers can cut through the jungle that is the Docker phenomenon and more easily grasp the benefits of using this hot new tool for simplifying your development process.

In this post, we’ll discuss what Docker is, what you can do with it, what you can (but probably don’t want to) do with it, and how to think about the soup that is Docker.

## What is Docker?

Docker is, in its own words, “an open platform for developers and sysadmins to build, ship, and run distributed applications”. In other, simpler words, Docker is a container manager. Note that it’s not a virtualization solution that abstracts away the underlying OS or hardware, like the well-known [VirtualBox](https://www.virtualbox.org/), but an engine to package an individual application system to run in its own particular environment.

It means that Docker can be used as a [virtualenv](http://virtualenv.readthedocs.org/) in Python, sandboxing the application dependencies so as not to let them to get into dependency management hell.

It’s pretty clear how Docker can be interesting for operations people to deploy apps to myriads of servers at once. Or for cloud computing enablers, like [Google Cloud Computing](https://cloud.google.com/compute/docs/containers), but let’s get our hands dirty and see how we, as ordinary Java developers, can benefit from containers in development.

## Installation and getting started

The instructions to install Docker are available from, surprise-surprise, its [website](https://docs.docker.com/installation/mac/#installation). I don’t want to go into the exact list of activities required to obtain it, so in order to keep this post more high level than that, this chapter will be very brief.

If you’re lucky to run a operation system that supports Docker out of the box (read: some Linux distro), you’re all good to go. Install Docker using your favorite package manager and verify this commmand works:

```
$ sudo docker run ubuntu:14.04 /bin/echo 'Hello world'
Hello world
```

If you’re not that lucky and belong to a category of people running Windows or OSX, worry not. There is [boot2docker](http://boot2docker.io/) project that will start a tiny lightweight Linux virtual machine for you so you can enjoy using Docker nonetheless.

Installing boot2docker comes easy, there are OS specific installers prepared for us. Running it is also trivial, and here are the [instructions for Mac](https://docs.docker.com/installation/mac/#installation) people.

Verify that it works using the command above, ssh into boot2docker and run it:

```
$ boot2docker ssh
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.3.1, build master : 9a31a68 - Fri Oct 31 03:14:34 UTC 2014
Docker version 1.3.1, build 4e9bbfa
 
docker@boot2docker:~$ sudo docker run ubuntu:14.04 /bin/echo 'Hello world'
Hello world
```

That’s it, we have run our first Docker app. It does nothing fancy, just echoes a message, but it is a crucial step in setting up the infrastructure for something useful.

## Let’s get groovy with those magnificent containers

Although we like the language, we’re not exactly a [Groovy](http://groovy.codehaus.org/) shop so we’ll focus on getting Java running in the container.

The good news is that even here, everything is done for us. There’s an official repo with Docker images for Java: [dockerfile/java](https://github.com/dockerfile/java). Well, to be very precise, these are not images, but dockerfiles that tell Docker how to create an actual image.

If you look at the [Docker file for Java 8](https://github.com/dockerfile/java/blob/master/oracle-java8/Dockerfile), you’ll immediately notice that the Java 8 image is based on the stock Ubuntu image and installs Java as you would do to the usual Ubuntu machine.

Additionally, it defines the JAVA_HOME and the default Docker command to execute:

```
# Define working directory.
WORKDIR /data
# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
# Define default command.
CMD ["bash"]
```

Now, our Docker container is equipped with Java and is able to run Java commands:

```
docker run -it --rm dockerfile/java:oracle-java8 java -version
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
```

We can run random Java commands to be executed in our Docker environment. And it doesn’t use a proper virtual machine for that, so no VM overhead is thrown upon you.

How easy and portable is that?!

## Awesome, let me personalise it even more

I know what you’re thinking: this is pretty cool, but now I have to make a dockerfile to download and install Maven, clone my project repository, then build it, downloading half of the internet in the process. Fear not, Docker can easily access your host filesystem, so you don’t have to provide much besides the access to Java.


Indeed, you can have it, cat. Linux obviously has the advantage of being the native OS of Docker, so the mapping is as seamless as it can be. A Docker command just takes a parameter, -v, which directories to map to something inside the container.

For example the command below will share my ~/repo dir with the container.

```
docker run -v /Users/shelajev/repo:/opt/repo ubuntu ls -la /opt/repo/zt-zip
```

OSX users, you’re covered too, since boot2docker 1.3.1 your home directory is shared by default with the boot2docker virtual machine, so the -v will work too:

![alt](http://resource.docker.cn/image00-640x349.png)

Windows people, don’t worry, there’s just an incantation for you [to make it work too](http://www.incrediblemolk.com/sharing-a-windows-folder-with-the-boot2docker-vm/). In short, you’ll have to install VirtualBox Guest Additions and manually map a folder between your host and the boot2docker VM. This is pretty straightforward when you understand what you’re doing or follow a how-to, like the one linked above.

Now it’s pretty clear how to run your actual Java applications in a Docker container. Your source code sits on your host machine, and you edit it there as well. You even build it on the host machine, because operations activities don’t start with the source code, they start with a prebuilt artifact. From there it’s as simple as putting it into the correct place and using the correct command to start it.

Here’s an example of a Java project, which I’ll use [Spring Boot](http://projects.spring.io/spring-boot/) for getting the application started. We wrote a bit about this in last week’s post, [What you can build for free in 2 hours with Spring Boot, Twitter and Facebook](http://zeroturnaround.com/rebellabs/what-you-can-build-for-free-in-2-hours-with-spring-boot-twitter-and-facebook/).

So usually, on my machine I’d run it like:

```
shelajev@shrimp  ~/repo/tmp/gs-spring-boot/complete  ‹master›
$ java -jar target/gs-spring-boot-0.1.0.jar
```

And verify that the application is running on the port 8080. To run it in the Docker container, I just do the same, but provide a port mapping so I can access that from the host machine:

```
docker run -v /Users/shelajev/repo:/opt/repo -p 8080:8080 --rm dockerfile/java:oracle-java8 java -jar /opt/repo/tmp/gs-spring-boot/complete/target/gs-spring-boot-0.1.0.jar
```

The output of this command is very predictable, it starts the Spring Boot app and prints all the initialization things it has to. For brevity, it’s not included here, but it looks exactly like this [pastebin](http://pastebin.com/B60rLz9d).

![alt](http://resource.docker.cn/image03-640x242.png)

E voilà! I can access it from my host machine browser! The exact IP where boot2docker offers an interface that is mapped to docker is easily obtainable with a:

```
open http://$(boot2docker ip 2>/dev/null)/
```

## Conclusion: Docker is as Dev-friendly as an Ops tool can be

I’m comfortable in asserting that Docker is a well-thought-out tool where almost everything is accessible from the command line.

In this post we looked at how to get started with Docker containers to enable a portable development environment for Java projects. This might reduce the install complexity for a new project without sacrificing the goods or performance that your usual environment provides you: your favorite IDE, automatic tests, tools like [JRebel](http://zeroturnaround.com/software/jrebel/) (shameless plug), and others.

For Java developers, Docker helps isolate our apps in a clean environment, so the unpredictability of “works on my machine” is a little less irritating. Isolation is important because it reduces the complexity of the software environment we’re using. Plus, to benefit from using Docker you don’t have to get into the world of containers and start deploying your application to thousands nodes. Just the isolation from your developer’s machine is a big plus.

Docker, with its intuitive user interface reduces this complexity even further, becoming a great tool for any developer. Additional benefits of using it is that it’ll make operations people life easier, and we all know how important is that. Free karma, right?

---

Original source: [Docker for Java Developers: How to sandbox your app in a clean environment](http://zeroturnaround.com/rebellabs/docker-for-java-developers-how-to-sandbox-your-app-in-a-clean-environment/)