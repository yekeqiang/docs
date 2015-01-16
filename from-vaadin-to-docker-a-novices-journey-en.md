# From Vaadin to Docker, a novice’s journey

---

Author: Nicolas Frankel	

---

I’m a huge [Vaadin](http://vaadin.com/) fan and I’ve created a [Github workshop](https://github.com/nfrankel/vaadin7-workshop/) I can demo at conferences. A common issue with such kind of workshops is that attendees have to prepare their workstations in advance… and there’s always a significant part of them that comes with not everything ready. At this point, two options are available to the speaker: either wait for each of the attendee to finish the preparation – too bad for the people who took the time at home to do that, or start anyway – and lose the not-ready part.

Given the current buzz around Docker, I thought that could be a very good way to make the workshop preparation quicker – only one step, and hasslefree – no problem regarding the quirks of your operation system. The required steps I ask the attendees are the following:

1. Install Git
2. Install Java, Maven and Tomcat
3. Clone the git repo
4. Build the project (to prepare the Maven repository)
5. Deploy the built webapp
6. Start Tomcat

These should directly be automated into Docker. As I wasted much time getting this to work, here’s the tale of my journey in achieving this (be warned, it’s quite long). If you’ve got similar use-cases, I hope it will be useful in you getting things done faster.

## Starting with Docker

The first step was to get to know the basics about Docker. Fortunately, I had the chance to attend a Docker workshop by David Gageot at Duchess Swiss. This included both Docker installation and basics of Dockerfile. I assume readers have likewise a basic understanding of Docker.

For those who don’t, I guess browsing the Docker’s official documentation is a nice idea:

- [Installation](https://docs.docker.com/installation/#installation)
- [Dockerfile reference](http://docs.docker.com/reference/builder/)

## Building my first Dockerfile

The Docker image can be built with the following command ran into the directory of the Dockerfile:

```
$ docker build -t vaadinworkshop .
```

The first issues one can encounter when playing with Docker the first time, is to get the following error message:

```
Get http:///var/run/docker.sock/v1.14/containers/json: dial unix /var/run/docker.sock: no such file or directory
```

The reason is because one didn’t export the required environment variables displayed by the boot2docker information message. If you lost the exact data, no worry, just use the `shellinit` boot2docker parameter:

```
$ boot2docker shellinit
Writing /Users/i303869/.docker/boot2docker-vm/ca.pem:
Writing /Users/i303869/.docker/boot2docker-vm/cert.pem:
Writing /Users/i303869/.docker/boot2docker-vm/key.pem:
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/i303869/.docker/boot2docker-vm
```

Copy-paste the export lines above will solve the issue. These can also be set in one’s .bashrc script as it seems these values seldom change.

Next in line is the following error:

```
Get http://192.168.59.103:2376/v1.14/containers/json: malformed HTTP response "x15x03x01x00x02x02"
```

This error message seems to be because of a mismatch between versions of the client and the server. It seems it is because of a bug on Mac OSX when upgrading. For a long term solution, reinstall Docker from scratch; for a quick fix, use the `--tls` flag with the docker command. As it is quite cumbersome to type it everything, one can alias it:

```
$ alias docker="docker --tls"
```

My last mistake when building the image comes from building the Dockerfile from a not empty directory. Docker sends every file it finds in the directory of the Dockerfile to the Docker container for build:

```
$ docker --tls build -t vaadinworkshop .
Sending build context to Docker daemon Too many kB
```

Fix: do not try this at home and start from a directory container the Dockerfile only.

## Starting from scratch

Dockerfiles describe images – images are built as a layered list of instructions. Docker images are designed around single inheritance: one image has to be set a single parent. An image requiring no parent starts from `scratch`, but Docker provides 4 base official distributions: busybox, debian, ubuntu and centos (operating systems are generally a good start).

Whatever you want to achieve, it is necessary to choose the right parent. Given the requirements I set for myself (Java, Maven, Tomcat and Git), I tried to find the right starting image. Many Dockerfiles are already available online on the [Docker hub](https://registry.hub.docker.com/). The browsing app is quite good, but to be really honest, the search can really be improved.

My intention was to use the image that matched the most of my requirements, then fill the gap. I could find no image providing Git, but I thought the [dgageot/maven Dockerfile](https://registry.hub.docker.com/u/dgageot/maven/dockerfile/) would be a nice starting point. The problem is that the base image is a busybox and provides no installer out-of-the-box (apt-get, yum, whatever). For this reason, David uses a lot of curl to get Java 8 and Maven in his Dockerfiles.
I foolishly thought I could use a [different flavor of busybox](https://registry.hub.docker.com/u/progrium/busybox/) that provides the [opkg](http://wiki.openwrt.org/doc/techref/opkg) installer. After a while, I accumulated many problems, resolving one heading to another. In the end, I finally decided to use the OS I was most comfortable with and to install everything myself:

```
FROM ubuntu:utopic
```

## Scripting Java installation

Installing git, maven and tomcat packages is very straightforward (if you don’t forget to use the non-interactive options) with RUN and apt-get:

```
RUN apt-get update && \
    apt-get install -y --force-yes git maven tomcat8
```

Java doesn’t fall into this nice pattern, as Oracle wants you to accept the license. Nice people did however publish it to a third-party repo. Steps are the following:

1. Add the needed package repository
2. Configure the system to automatically accept the license
3. Configure the system to add un-certified packages
4. Update the list of repositories
5. At last, install the package
6. Also add a package for Java 8 system configuration

```
RUN echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" | tee -a /etc/apt/sources.list && \
    echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
 
RUN apt-get update && \
    apt-get install -y --force-yes oracle-java8-installer oracle-java8-set-default
```

## Building the sources

Getting the workshop’s sources and building them is quite straightforward with the following instructions:

```
RUN git clone  https://github.com/nfrankel/vaadin7-workshop.git
WORKDIR /vaadin7-workshop
RUN mvn package
```

The drawback of this approach is that Maven will start from a fresh repository, and thus download the Internet the first time it is launched. At first, I wanted to mount a volume from the host to the container to share the `~/.m2/repository` folder to avoid this, but I noticed this could only be done at runtime through the `-v `option as the `VOLUME` instruction cannot point to a host directory.

## Starting the image

The simplest command to start the created Docker image is the following:

```
$ docker run -p 8080:8080
```

Do not forget the port forwarding from the container to the host, 8080 for the standard HTTP port. Also, note that it’s not necessary to run the container as a daemon (with the -d option). The added value of that is that the standard output of the CMD (see below) will be redirected to the host. When running as a daemon and wanting to check the logs, one has to execute bash in the container, which requires a sequence of cumbersome manipulations.

## Configuring and launching Tomcat

Tomcat can be launched when starting the container by just adding the following instruction to the Dockerfile:

```
CMD ["catalina.sh", "run"]
```

However, trying to start the container at this point will result in the following error:

```
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/common/classes], exists: [false], isDirectory: [false], canRead: [false]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/common], exists: [false], isDirectory: [false], canRead: [false]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/server/classes], exists: [false], isDirectory: [false], canRead: [false]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/server], exists: [false], isDirectory: [false], canRead: [false]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/shared/classes], exists: [false], isDirectory: [false], canRead: [false]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.ClassLoaderFactory validateFile
WARNING: Problem with directory [/usr/share/tomcat8/shared], exists: [false], isDirectory: [false], canRead: [false]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.Catalina initDirs
SEVERE: Cannot find specified temporary folder at /usr/share/tomcat8/temp
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.Catalina load
WARNING: Unable to load server configuration from [/usr/share/tomcat8/conf/server.xml]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.Catalina initDirs
SEVERE: Cannot find specified temporary folder at /usr/share/tomcat8/temp
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.Catalina load
WARNING: Unable to load server configuration from [/usr/share/tomcat8/conf/server.xml]
Nov 15, 2014 9:24:18 PM org.apache.catalina.startup.Catalina start
SEVERE: Cannot start server. Server instance is not configured.
```

I have no idea why, but it seems Tomcat 8 on Ubuntu is not configured in any meaningful way. Everything is available but we need some symbolic links here and there as well as creating the temp directory. This translates into the following instruction in the Dockerfile:

```
RUN ln -s /var/lib/tomcat8/common $CATALINA_HOME/common && \
    ln -s /var/lib/tomcat8/server $CATALINA_HOME/server && \
    ln -s /var/lib/tomcat8/shared $CATALINA_HOME/shared && \
    ln -s /etc/tomcat8 $CATALINA_HOME/conf && \
    mkdir $CATALINA_HOME/temp
```

The final trick is to connect the exploded webapp folder created by Maven to Tomcat’s webapps folder, which it looks for deployments:

```
RUN mkdir $CATALINA_HOME/webapps && \
    ln -s /vaadin7-workshop/target/workshop-7.2-1.0-SNAPSHOT/ $CATALINA_HOME/webapps/vaadinworkshop
```

At this point, the `Holy Grail` is not far away, you just have to browse the URL… if only we knew what the IP was. Since running on Mac, there’s an additional VM beside the host and the container that’s involved. To get this IP, type:

```
$ boot2docker ip
 
The VM's Host only interface IP address is: 192.168.59.103
```

Now, browsing [http://192.168.59.103:8080/vaadinworkshop/](http://192.168.59.103:8080/vaadinworkshop/) will bring us to the familiar workshop screen:

![alt](http://resource.docker.cn/workshop.png)

## Developing from there

Everything works fine but didn’t we just forget about one important thing, like how workshop attendees are supposed to work on the sources? Easy enough, just mount the volume when starting the container:

```
docker run -v /Users/<login>/vaadin7-workshop:/vaadin7-workshop  -p 8080:8080 vaadinworkshop
```

Note that the host volume must be part of /Users and if on OSX, it must use [boot2docker v. 1.3+](http://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/).

Unfortunately, it seems now is the showstopper, as mounting an empty directory from the host to the container will not make the container’s directory available from the host. On the contrary, it will empty the container’s directory given that the host’s directory doesn’t exist… It seems there’s an [issue](https://github.com/docker/docker/issues/4023) in Docker on Mac. The installation of JHipster runs into the same problem, and proposes to use the [Samba Docker folder sharing project](https://github.com/SvenDowideit/dockerfiles/tree/master/samba).

I’m afraid I was too lazy to go further at this point. However, this taught me much about Docker, its usages and use-cases (as well as OSX integration limitations). For those who are interested, you’ll find below the Docker file. Happy Docker!

```
FROM ubuntu:utopic
 
MAINTAINER Nicolas Frankel <nicolas [at] frankel (dot) ch>
 
# Config to get to install Java 8 w/o interaction
RUN echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu precise main" | tee -a /etc/apt/sources.list && 
echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && 
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
 
RUN apt-get update && 
apt-get install -y --force-yes git oracle-java8-installer oracle-java8-set-default maven tomcat8
 
RUN git clone https://github.com/nfrankel/vaadin7-workshop.git
WORKDIR /vaadin7-workshop
RUN git checkout v7.2-1
RUN mvn package
 
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV CATALINA_HOME /usr/share/tomcat8
ENV PATH $PATH:$CATALINA_HOME/bin
 
# Configure Tomcat 8 directories
RUN ln -s /var/lib/tomcat8/common $CATALINA_HOME/common && 
ln -s /var/lib/tomcat8/server $CATALINA_HOME/server && 
ln -s /var/lib/tomcat8/shared $CATALINA_HOME/shared && 
ln -s /etc/tomcat8 $CATALINA_HOME/conf && 
mkdir $CATALINA_HOME/temp && 
mkdir $CATALINA_HOME/webapps && 
ln -s /vaadin7-workshop/target/workshop-7.2-1.0-SNAPSHOT/ $CATALINA_HOME/webapps/vaadinworkshop
 
VOLUME ["/vaadin7-workshop"]
 
CMD ["catalina.sh", "run"]
 
# docker build -t vaadinworkshop .
# docker run -v ~/vaadin7-workshop training/webapp -p 8080:8080 vaadinworkshop
- See more at: http://blog.frankel.ch/from-vaadin-to-docker-a-novices-journey#sthash.JhVLoYTg.dpuf
```

---

Original source: [From Vaadin to Docker, a novice’s journey](http://blog.frankel.ch/from-vaadin-to-docker-a-novices-journey)