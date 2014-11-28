# Create your own Docker image (Tech Tip #57)

---

Author: Arun Gupta

---

[Docker](https://www.docker.com/) simplifies software delivery by making it easy to build and share images that contain your application’s entire environment, i.e. operating system, JDK, database, WAR file, specific tuning required for your application, etc.

There are three main components of Docker:

- Docker images are “build component” – a read-only template of application operating system.
- Containers are “run component” – a runtime representation created from images.
- Registry are “distribution component” – a place to store and distribute images.

Several JBoss projects are available as Docker images at [www.jboss.org/docker](http://www.jboss.org/docker/). [Tech Tip #39](http://blog.arungupta.me/2014/07/getting-started-with-docker/) explained how to get started with Docker on Mac. It also explained how to start the [official WildFly Docker image](https://github.com/jboss-dockerfiles/wildfly).

Docker image is made up of multiple layers where each layer provides some functionality, and a higher layer can add functionality on top of it. For example, Docker mounts the root filesystem as read-only layer and then adds a read-write layer on top of it. All these layers are combined together using [Union Mount](v) to provide application operating environment.

The complete history of how the WildFly image was built can be seen as:


```
docker history --no-trunc=true jboss/wildfly
IMAGE                                                              CREATED             CREATED BY                                                                                                                                                                   SIZE
365390553f925f96f8c00f79525ad101847de7781bb4fec23b1188f25fe99a6a   3 weeks ago         /bin/sh -c #(nop) CMD [/opt/jboss/wildfly/bin/standalone.sh -b 0.0.0.0]                                                                                                      0 B
d7fccab36b8f5a324ef8dd017c8964f2de7839379503021f15502ba3f0b908bd   3 weeks ago         /bin/sh -c #(nop) EXPOSE map[8080/tcp:{} 9990/tcp:{}]                                                                                                                        0 B
184d6d02f340455d33d226d6467484027d0763964a586ae9ada1782262299a74   3 weeks ago         /bin/sh -c #(nop) ENV JBOSS_HOME=/opt/jboss/wildfly                                                                                                                          0 B
57ada25ecdd03191355ec1c8f5f1a4e05b3e152709c9b603d5ed5fb0c4d53853   3 weeks ago         /bin/sh -c cd $HOME && curl http://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.tar.gz | tar zx && mv $HOME/wildfly-$WILDFLY_VERSION $HOME/wildfly   135 MB
59ec65b61fba2b6ada099abe3a9a30d05ffb71370b43dfbd0208fd4f5a34c005   3 weeks ago         /bin/sh -c #(nop) ENV WILDFLY_VERSION=8.1.0.Final                                                                                                                            0 B
90832e1f0bb9e9f98ecd42f6df6b124c1e6768babaddc23d646cd75c7b2fddec   5 weeks ago         /bin/sh -c #(nop) ENV JAVA_HOME=/usr/lib/jvm/java                                                                                                                            0 B
72d585299bb5c5c1c326422cfffadc93d8bb4020f35bf072b2d91d287967807a   5 weeks ago         /bin/sh -c #(nop) USER jboss                                                                                                                                                 0 B
e02bdb6c4ed5436da02c958d302af5f06c1ebb1821791f60d45e190ebb55130f   5 weeks ago         /bin/sh -c yum -y install java-1.7.0-openjdk-devel && yum clean all                                                                                                          217.2 MB
b17a20d6f5f8e7ed0a1dba277acd3f854c531b0476b03d63a8f0df4caf78c763   5 weeks ago         /bin/sh -c #(nop) USER root                                                                                                                                                  0 B
7759146eab1a3aa5ba5ed12483d03e64a6bf1061a383d5713a5e21fc40554457   5 weeks ago         /bin/sh -c #(nop) MAINTAINER Marek Goldmann <mgoldman@redhat.com>                                                                                                            0 B
2ea8562cac7c25a308b4565b66d4f7e11a1d2137a599ef2b32ed23c78f0a0378   5 weeks ago         /bin/sh -c #(nop) USER jboss                                                                                                                                                 0 B
4d37cbbfc67dd508e682a5431a99d8c1feba1bd8352ffd3ea794463d9cfa81cc   5 weeks ago         /bin/sh -c #(nop) WORKDIR /opt/jboss                                                                                                                                         0 B
379edb00ab0764276787ea777243990da697f2f93acb5d9166ff73ad01511a87   5 weeks ago         /bin/sh -c groupadd -r jboss -g 1000 && useradd -u 1000 -r -g jboss -m -d /opt/jboss -s /sbin/nologin -c "JBoss user" jboss                                                  295 kB
cd5bb934bb6755e910d19ac3ae4cfd09221aa2f98c3fbb51a7486991364dc1ae   5 weeks ago         /bin/sh -c yum -y install xmlstarlet saxon augeas bsdtar unzip && yum clean all                                                                                              21.35 MB
20a1abe1d9bfb9b1e46d5411abd5a38b6104a323b7c4fb5c0f1f161b8f7278c2   5 weeks ago         /bin/sh -c yum -y update && yum clean all                                                                                                                                    200.7 MB
1ef0a50fe8b1394d3626a7624a58b58cff9560ddb503743099a56bbe95ab481a   5 weeks ago         /bin/sh -c #(nop) MAINTAINER Marek Goldmann <mgoldman@redhat.com>                                                                                                            0 B
7d3f07f8de5fb3a20c6cb1e4447773a5741e3641c1aa093366eaa0fc690c6417   7 weeks ago         /bin/sh -c #(nop) ADD file:285fdeab65d637727f6b79392a309135494d2e6046c6cc2fbd2f23e43eaac69c in /                                                                             374.1 MB
782cf93a8f16d3016dae352188cd5cfedb6a15c37d4dbd704399f02d1bb89dab   7 weeks ago         /bin/sh -c #(nop) MAINTAINER Lokesh Mandvekar <lsm5@fedoraproject.org> - ./buildcontainers.sh                                                                                0 B
511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158   17 months ago                                                                                                                                                                                    0 B
```

The exact command issued at each layer is listed in this output. If you scroll to the far right then you can see the total space consumed by each layer as well. For example, Fedora is used as the base image and consumes ~574 MB of the total image, Open JDK 7 is taking 217.5 MB and WildFly is 135 MB.

Docker images are built by reading the instructions from Dockerfile. This is a text file that contains all the commands, in order, needed to build a given image. It adheres to a specific format and use a specific set of instructions. The [vocabulary of commands](https://docs.docker.com/reference/builder/) is rather limited but serves the purpose well. The image can be built by giving the command `docker build`. [Docker Tutorial](https://docs.docker.com/userguide/dockerimages/) provides complete instructions on how to create your own custom image.

The [official WildFly Docker image](http://www.jboss.org/docker/) is built using Fedora 20 as the base operating system. The Dockerfile can be seen at [github.com/jboss-dockerfiles/wildfly/blob/master/Dockerfile](https://github.com/jboss-dockerfiles/wildfly/blob/master/Dockerfile). It uses [jboss/base-jdk:7](https://registry.hub.docker.com/u/jboss/base-jdk/) as the base image, which uses [jboss/base](https://registry.hub.docker.com/u/jboss/base/) as the base image. [Dockerfile of jboss/base](https://registry.hub.docker.com/u/jboss/base/dockerfile/) shows Fedora 20 is used as the base image.

An alternative is to build this image using CentOS or Ubuntu as a base image. Dockerfiles for these images are available at [github.com/arun-gupta/docker-images/](https://github.com/arun-gupta/docker-images/).

Starting [boot2docker](http://boot2docker.io/) shows the output as:


```
bash
unset DYLD_LIBRARY_PATH ; unset LD_LIBRARY_PATH
mkdir -p ~/.boot2docker
if [ ! -f ~/.boot2docker/boot2docker.iso ]; then cp /usr/local/share/boot2docker/boot2docker.iso ~/.boot2docker/ ; fi
/usr/local/bin/boot2docker init 
/usr/local/bin/boot2docker up 
$(/usr/local/bin/boot2docker shellinit)
docker version
Last login: Mon Nov 24 11:03:33 on ttys006
hello arungupta
~> bash
~> unset DYLD_LIBRARY_PATH ; unset LD_LIBRARY_PATH
~> mkdir -p ~/.boot2docker
~> if [ ! -f ~/.boot2docker/boot2docker.iso ]; then cp /usr/local/share/boot2docker/boot2docker.iso ~/.boot2docker/ ; fi
~> /usr/local/bin/boot2docker init 
Virtual machine boot2docker-vm already exists
~> /usr/local/bin/boot2docker up 
Waiting for VM and Docker daemon to start...
.....................ooooooooooooooooo
Started.
Writing /Users/arungupta/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/arungupta/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/arungupta/.boot2docker/certs/boot2docker-vm/key.pem
 
To connect the Docker client to the Docker daemon, please set:
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/arungupta/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
 
~> $(/usr/local/bin/boot2docker shellinit)
Writing /Users/arungupta/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/arungupta/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/arungupta/.boot2docker/certs/boot2docker-vm/key.pem
~> docker version
Client version: 1.3.1
Client API version: 1.15
Go version (client): go1.3.3
Git commit (client): 4e9bbfa
OS/Arch (client): darwin/amd64
Server version: 1.3.1
Server API version: 1.15
Go version (server): go1.3.3
Git commit (server): 4e9bbfa
```

And then you can build the CentOS-based WildFly Docker image as shown below. Note this command is given from the “wildfly-centos” directory of [github.com/arun-gupta/docker-images/](https://github.com/arun-gupta/docker-images/). And so the Dockerfile is at github.com/arun-gupta/docker-images/blob/master/wildfly-centos/Dockerfile.


```
wildfly-centos&gt; docker build -t wildfly-centos .
Sending build context to Docker daemon 4.096 kB
Sending build context to Docker daemon 
Step 0 : FROM centos
centos:latest: The image you are pulling has been verified
511136ea3c5a: Pull complete 
5b12ef8fd570: Pull complete 
ae0c2d0bdc10: Pull complete 
Status: Downloaded newer image for centos:latest
 ---&gt; ae0c2d0bdc10
Step 1 : MAINTAINER Arun Gupta &lt;arungupta@redhat.com&gt;
 ---&gt; Running in 7fc52653d381
 ---&gt; e490dfcb3685
Removing intermediate container 7fc52653d381
Step 2 : RUN yum -y update &amp;&amp; yum clean all
 ---&gt; Running in 90de23c9dde7
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.cc.columbia.edu
 * extras: centos.mirror.ndchost.com
 * updates: centos-distro.cavecreek.net
Resolving Dependencies
--&gt; Running transaction check
---&gt; Package tzdata.noarch 0:2014h-1.el7 will be updated
---&gt; Package tzdata.noarch 0:2014j-1.el7_0 will be an update
--&gt; Finished Dependency Resolution
 
Dependencies Resolved
 
================================================================================
 Package         Arch            Version                 Repository        Size
================================================================================
Updating:
 tzdata          noarch          2014j-1.el7_0           updates          434 k
 
Transaction Summary
================================================================================
Upgrade  1 Package
 
Total download size: 434 k
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
warning: /var/cache/yum/x86_64/7/updates/packages/tzdata-2014j-1.el7_0.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for tzdata-2014j-1.el7_0.noarch.rpm is not installed
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) &lt;security@centos.org&gt;"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-0.1406.el7.centos.2.5.x86_64 (@Updates/$releasever)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : tzdata-2014j-1.el7_0.noarch                                  1/2 
  Cleanup    : tzdata-2014h-1.el7.noarch                                    2/2 
  Verifying  : tzdata-2014j-1.el7_0.noarch                                  1/2 
  Verifying  : tzdata-2014h-1.el7.noarch                                    2/2 
 
Updated:
  tzdata.noarch 0:2014j-1.el7_0                                                 
 
Complete!
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
Cleaning up everything
Cleaning up list of fastest mirrors
 ---&gt; f212cb9dbcf5
Removing intermediate container 90de23c9dde7
Step 3 : RUN yum -y install xmlstarlet saxon augeas bsdtar unzip &amp;&amp; yum clean all
 ---&gt; Running in d4bd822933c8
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror-centos.hostingswift.com
 * extras: mirror.keystealth.org
 * updates: mirrors.advancedhosters.com
No package xmlstarlet available.
Resolving Dependencies
--&gt; Running transaction check
---&gt; Package augeas.x86_64 0:1.1.0-12.el7 will be installed
--&gt; Processing Dependency: augeas-libs = 1.1.0-12.el7 for package: augeas-1.1.0-12.el7.x86_64
 
. . .
 
--&gt; Processing Dependency: python-lxml for package: python-javapackages-3.4.1-6.el7_0.noarch
--&gt; Running transaction check
---&gt; Package python-lxml.x86_64 0:3.2.1-4.el7 will be installed
--&gt; Finished Dependency Resolution
 
Dependencies Resolved
 
================================================================================
 Package                  Arch        Version                Repository    Size
================================================================================
Installing:
 augeas                   x86_64      1.1.0-12.el7           base          35 k
 bsdtar                   x86_64      3.1.2-7.el7            base          55 k
 
. . .
 
 python-javapackages      noarch      3.4.1-6.el7_0          updates       31 k
 python-lxml              x86_64      3.2.1-4.el7            base         758 k
 
Transaction Summary
================================================================================
Install  4 Packages (+9 Dependent packages)
 
Total download size: 4.2 M
Installed size: 8.0 M
Downloading packages:
--------------------------------------------------------------------------------
Total                                              188 kB/s | 4.2 MB  00:22     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : lzo-2.06-6.el7_0.2.x86_64                                   1/13 
  Installing : libxslt-1.1.28-5.el7.x86_64                                 
 
. . .
 
  Installing : unzip-6.0-13.el7.x86_64                                    13/13 
  Verifying  : augeas-1.1.0-12.el7.x86_64                                  1/13 
 
. . .
 
  Verifying  : javapackages-tools-3.4.1-6.el7_0.noarch                    13/13 
 
Installed:
  augeas.x86_64 0:1.1.0-12.el7            bsdtar.x86_64 0:3.1.2-7.el7          
  saxon.noarch 0:9.3.0.4-11.el7           unzip.x86_64 0:6.0-13.el7            
 
Dependency Installed:
  augeas-libs.x86_64 0:1.1.0-12.el7  bea-stax.noarch 0:1.2.0-9.el7              
  bea-stax-api.noarch 0:1.2.0-9.el7  javapackages-tools.noarch 0:3.4.1-6.el7_0  
  libarchive.x86_64 0:3.1.2-7.el7    libxslt.x86_64 0:1.1.28-5.el7              
  lzo.x86_64 0:2.06-6.el7_0.2        python-javapackages.noarch 0:3.4.1-6.el7_0 
  python-lxml.x86_64 0:3.2.1-4.el7  
 
Complete!
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
Cleaning up everything
Cleaning up list of fastest mirrors
 ---&gt; 28b11e6151f0
Removing intermediate container d4bd822933c8
Step 4 : RUN groupadd -r jboss -g 1000 &amp;&amp; useradd -u 1000 -r -g jboss -m -d /opt/jboss -s /sbin/nologin -c "JBoss user" jboss
 ---&gt; Running in 943c20ba5a51
 ---&gt; 73603eab89b7
Removing intermediate container 943c20ba5a51
Step 5 : WORKDIR /opt/jboss
 ---&gt; Running in 29c865c3109c
 ---&gt; 9a661ae4341b
Removing intermediate container 29c865c3109c
Step 6 : USER jboss
 ---&gt; Running in 7dfd8416ae2c
 ---&gt; 6265153611c7
Removing intermediate container 7dfd8416ae2c
Step 7 : USER root
 ---&gt; Running in a72588fba840
 ---&gt; 12ed28a7acb7
Removing intermediate container a72588fba840
Step 8 : RUN yum -y install java-1.7.0-openjdk-devel &amp;&amp; yum clean all
 ---&gt; Running in 4efb3e17eb38
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.trouble-free.net
 * extras: centos.mirror.ndchost.com
 * updates: centos-distro.cavecreek.net
Resolving Dependencies
--&gt; Running transaction check
---&gt; Package java-1.7.0-openjdk-devel.x86_64 1:1.7.0.71-2.5.3.1.el7_0 will be installed
 
. . .
 
---&gt; Package hwdata.noarch 0:0.252-7.3.el7 will be installed
--&gt; Finished Dependency Resolution
 
Dependencies Resolved
 
================================================================================
 Package                      Arch    Version                    Repository
                                                                           Size
================================================================================
Installing:
 java-1.7.0-openjdk-devel     x86_64  1:1.7.0.71-2.5.3.1.el7_0   updates  9.2 M
Installing for dependencies:
 alsa-lib                     x86_64  1.0.27.2-3.el7             base     389 k
 
. . .
 
144 k
 xorg-x11-font-utils          x86_64  1:7.5-18.1.el7             base      87 k
 xorg-x11-fonts-Type1         noarch  7.5-9.el7                  base     521 k
 
Transaction Summary
================================================================================
Install  1 Package (+73 Dependent packages)
 
Total download size: 49 M
Installed size: 181 M
Downloading packages:
--------------------------------------------------------------------------------
Total                                              1.7 MB/s |  49 MB  00:29     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : freetype-2.4.11-9.el7.x86_64                                1/74 
  Installing : libjpeg-turbo-1.2.90-5.el7.x86_64                           2/74 
 
. . .
 
73/74 
  Installing : 1:java-1.7.0-openjdk-devel-1.7.0.71-2.5.3.1.el7_0.x86_64   74/74 
  Verifying  : libsndfile-1.0.25-9.el7.x86_64                              1/74 
  Verifying  : libXfont-1.4.7-2.el7_0.x86_64                               2/74 
  Verifying  : kmod-14-9.el7.x86_64                                        
 
. . .
 
73/74 
  Verifying  : gdk-pixbuf2-2.28.2-4.el7.x86_64                            74/74 
 
Installed:
  java-1.7.0-openjdk-devel.x86_64 1:1.7.0.71-2.5.3.1.el7_0                      
 
Dependency Installed:
  alsa-lib.x86_64 0:1.0.27.2-3.el7                                              
  atk.x86_64 0:2.8.0-4.el7                                                      
 
. . .
                                          
  xorg-x11-font-utils.x86_64 1:7.5-18.1.el7                                     
  xorg-x11-fonts-Type1.noarch 0:7.5-9.el7                                       
 
Complete!
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
Cleaning up everything
Cleaning up list of fastest mirrors
 ---&gt; 44c4bb92fa11
Removing intermediate container 4efb3e17eb38
Step 9 : USER jboss
 ---&gt; Running in 824d62c49182
 ---&gt; 930cb2a860f7
Removing intermediate container 824d62c49182
Step 10 : ENV JAVA_HOME /usr/lib/jvm/java
 ---&gt; Running in f19681365fe5
 ---&gt; fff2c21b0a71
Removing intermediate container f19681365fe5
Step 11 : ENV WILDFLY_VERSION 8.2.0.Final
 ---&gt; Running in cc9d42ece5c1
 ---&gt; b7b7ca7a9172
Removing intermediate container cc9d42ece5c1
Step 12 : RUN cd $HOME &amp;&amp; curl -O http://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.zip &amp;&amp; unzip wildfly-$WILDFLY_VERSION.zip &amp;&amp; mv $HOME/wildfly-$WILDFLY_VERSION $HOME/wildfly &amp;&amp; rm wildfly-$WILDFLY_VERSION.zip
 ---&gt; Running in 28e92a1b304f
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  126M  100  126M    0     0   813k      0  0:02:38  0:02:38 --:--:--  644k
Archive:  wildfly-8.2.0.Final.zip
   creating: wildfly-8.2.0.Final/
   creating: wildfly-8.2.0.Final/.installation/
   creating: wildfly-8.2.0.Final/appclient/
   creating: wildfly-8.2.0.Final/appclient/configuration/
   creating: wildfly-8.2.0.Final/bin/
 
. . .
 
  inflating: wildfly-8.2.0.Final/domain/configuration/application-users.properties  
  inflating: wildfly-8.2.0.Final/domain/configuration/mgmt-users.properties  
  inflating: wildfly-8.2.0.Final/standalone/configuration/application-users.properties  
  inflating: wildfly-8.2.0.Final/standalone/configuration/mgmt-users.properties  
   creating: wildfly-8.2.0.Final/domain/tmp/auth/
   creating: wildfly-8.2.0.Final/standalone/tmp/auth/
 ---&gt; a1bc79a43c77
Removing intermediate container 28e92a1b304f
Step 13 : ENV JBOSS_HOME /opt/jboss/wildfly
 ---&gt; Running in e3c995170046
 ---&gt; d46fdd618d55
Removing intermediate container e3c995170046
Step 14 : EXPOSE 8080 9990
 ---&gt; Running in d55d6a6f43cf
 ---&gt; 6c17e2cefecf
Removing intermediate container d55d6a6f43cf
Step 15 : CMD /opt/jboss/wildfly/bin/standalone.sh -b 0.0.0.0
 ---&gt; Running in 76e2630d16f5
 ---&gt; 97c8780a7d6a
Removing intermediate container 76e2630d16f5
Successfully built 97c8780a7d6a
```

The list of Docker images can now be seen as:

```
wildfly-centos&gt; docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
wildfly-centos      latest              97c8780a7d6a        58 seconds ago      619.6 MB
centos              latest              ae0c2d0bdc10        2 weeks ago         224 MB
```

The total image size is 619.6 MB. The official WildFly Docker image can be installed as shown:

```
wildfly-centos&gt; docker pull jboss/wildfly
Pulling repository jboss/wildfly
365390553f92: Download complete 
511136ea3c5a: Download complete 
782cf93a8f16: Download complete 
7d3f07f8de5f: Download complete 
1ef0a50fe8b1: Download complete 
20a1abe1d9bf: Download complete 
cd5bb934bb67: Download complete 
379edb00ab07: Download complete 
4d37cbbfc67d: Download complete 
2ea8562cac7c: Download complete 
7759146eab1a: Download complete 
b17a20d6f5f8: Download complete 
e02bdb6c4ed5: Download complete 
72d585299bb5: Download complete 
90832e1f0bb9: Download complete 
59ec65b61fba: Download complete 
57ada25ecdd0: Download complete 
184d6d02f340: Download complete 
d7fccab36b8f: Download complete 
Status: Downloaded newer image for jboss/wildfly:latest
```

And the complete list of Docker images can again be seen as:

```
wildfly-centos> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
wildfly-centos      latest              97c8780a7d6a        12 minutes ago      619.6 MB
centos              latest              ae0c2d0bdc10        2 weeks ago         224 MB
jboss/wildfly       latest              365390553f92        4 weeks ago         948.7 MB
```

The image size in this case is 948.7 MB. A detailed understanding of this image is created was explained earlier in this blog.

Ubuntu-based WildFly image can be built and installed as shown below. Note this command is given from the “wildfly-ubuntu” directory of [github.com/arun-gupta/docker-images/](https://github.com/arun-gupta/docker-images/). And so the Dockerfile is at [github.com/arun-gupta/docker-images/blob/master/wildfly-ubuntu/Dockerfile](github.com/arun-gupta/docker-images/blob/master/wildfly-ubuntu/Dockerfile).


```
wildfly-ubuntu> docker build -t wildfly-ubuntu .
Sending build context to Docker daemon 4.096 kB
Sending build context to Docker daemon 
Step 0 : FROM ubuntu
ubuntu:latest: The image you are pulling has been verified
d497ad3926c8: Pull complete 
ccb62158e970: Pull complete 
e791be0477f2: Pull complete 
3680052c0f5c: Pull complete 
22093c35d77b: Pull complete 
5506de2b643b: Pull complete 
511136ea3c5a: Already exists 
Status: Downloaded newer image for ubuntu:latest
 ---> 5506de2b643b
Step 1 : MAINTAINER Arun Gupta <arungupta@redhat.com>
 ---> Running in cc444b436b26
 ---> 9df66d33d676
Removing intermediate container cc444b436b26
Step 2 : RUN apt-get update
 ---> Running in 3dc63c0f708a
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
Ign http://archive.ubuntu.com trusty-security InRelease
Ign http://archive.ubuntu.com trusty-proposed InRelease
Get:1 http://archive.ubuntu.com trusty Release.gpg [933 B]
Get:2 http://archive.ubuntu.com trusty-updates Release.gpg [933 B]
Get:3 http://archive.ubuntu.com trusty-security Release.gpg [933 B]
Get:4 http://archive.ubuntu.com trusty-proposed Release.gpg [933 B]
Get:5 http://archive.ubuntu.com trusty Release [58.5 kB]
Get:6 http://archive.ubuntu.com trusty-updates Release [62.0 kB]
Get:7 http://archive.ubuntu.com trusty-security Release [62.0 kB]
Get:8 http://archive.ubuntu.com trusty-proposed Release [209 kB]
Get:9 http://archive.ubuntu.com trusty/main Sources [1335 kB]
Get:10 http://archive.ubuntu.com trusty/restricted Sources [5335 B]
Get:11 http://archive.ubuntu.com trusty/universe Sources [7926 kB]
Get:12 http://archive.ubuntu.com trusty/main amd64 Packages [1743 kB]
Get:13 http://archive.ubuntu.com trusty/restricted amd64 Packages [16.0 kB]
Get:14 http://archive.ubuntu.com trusty/universe amd64 Packages [7589 kB]
Get:15 http://archive.ubuntu.com trusty-updates/main Sources [179 kB]
Get:16 http://archive.ubuntu.com trusty-updates/restricted Sources [1250 B]
Get:17 http://archive.ubuntu.com trusty-updates/universe Sources [114 kB]
Get:18 http://archive.ubuntu.com trusty-updates/main amd64 Packages [465 kB]
Get:19 http://archive.ubuntu.com trusty-updates/restricted amd64 Packages [6341 B]
Get:20 http://archive.ubuntu.com trusty-updates/universe amd64 Packages [286 kB]
Get:21 http://archive.ubuntu.com trusty-security/main Sources [62.7 kB]
Get:22 http://archive.ubuntu.com trusty-security/restricted Sources [40 B]
Get:23 http://archive.ubuntu.com trusty-security/universe Sources [19.1 kB]
Get:24 http://archive.ubuntu.com trusty-security/main amd64 Packages [205 kB]
Get:25 http://archive.ubuntu.com trusty-security/restricted amd64 Packages [40 B]
Get:26 http://archive.ubuntu.com trusty-security/universe amd64 Packages [93.4 kB]
Get:27 http://archive.ubuntu.com trusty-proposed/main amd64 Packages [193 kB]
Get:28 http://archive.ubuntu.com trusty-proposed/restricted amd64 Packages [40 B]
Fetched 20.6 MB in 43s (469 kB/s)
Reading package lists...
 ---> 7fcaaf17b124
Removing intermediate container 3dc63c0f708a
Step 3 : RUN apt-get -y install xmlstarlet bsdtar unzip curl
 ---> Running in 81613b0e3b71
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:
  ca-certificates krb5-locales libarchive13 libasn1-8-heimdal libcurl3
  libgssapi-krb5-2 libgssapi3-heimdal libhcrypto4-heimdal libheimbase1-heimdal
 
. . .
 
Updating certificates in /etc/ssl/certs... 164 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....done.
Processing triggers for sgml-base (1.26+nmu4ubuntu1) ...
 ---> 0d897d545547
Removing intermediate container 81613b0e3b71
Step 4 : RUN groupadd -r jboss -g 1000 && useradd -u 1000 -r -g jboss -m -d /opt/jboss -s /sbin/nologin -c "JBoss user" jboss
 ---> Running in bf31e0e1f448
 ---> 7494e71043c0
Removing intermediate container bf31e0e1f448
Step 5 : WORKDIR /opt/jboss
 ---> Running in 1ac355c74395
 ---> abbbb6aa6b71
Removing intermediate container 1ac355c74395
Step 6 : USER jboss
 ---> Running in a656007f6e6b
 ---> 657671b82d0f
Removing intermediate container a656007f6e6b
Step 7 : USER root
 ---> Running in f53138927d04
 ---> d3bd53152a51
Removing intermediate container f53138927d04
Step 8 : RUN apt-get -y install openjdk-7-jdk
 ---> Running in 1076b257c6f6
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:
  acl at-spi2-core ca-certificates ca-certificates-java colord cpp cpp-4.8
  dbus dbus-x11 dconf-gsettings-backend dconf-service desktop-file-utils
 
. . .
 
Adding debian:Wells_Fargo_Root_CA.pem
Adding debian:XRamp_Global_CA_Root.pem
Adding debian:certSIGN_ROOT_CA.pem
Adding debian:ePKI_Root_Certification_Authority.pem
Adding debian:thawte_Primary_Root_CA.pem
Adding debian:thawte_Primary_Root_CA_-_G2.pem
Adding debian:thawte_Primary_Root_CA_-_G3.pem
Adding debian:spi-cacert-2008.pem
done.
done.
 ---> 558f23c93a8c
Removing intermediate container 1076b257c6f6
Step 9 : USER jboss
 ---> Running in 9d6955bcd19f
 ---> 7f16c426ccaf
Removing intermediate container 9d6955bcd19f
Step 10 : ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
 ---> Running in 66f22bd0de95
 ---> 6304f386c946
Removing intermediate container 66f22bd0de95
Step 11 : ENV WILDFLY_VERSION 8.2.0.Final
 ---> Running in 784e1533fa09
 ---> 1a5e1aeadc85
Removing intermediate container 784e1533fa09
Step 12 : RUN cd $HOME && curl -O http://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.zip && unzip wildfly-$WILDFLY_VERSION.zip && mv $HOME/wildfly-$WILDFLY_VERSION $HOME/wildfly && rm wildfly-$WILDFLY_VERSION.zip
 ---> Running in c7d808526af6
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  126M  100  126M    0     0   993k      0  0:02:09  0:02:09 --:--:--  870k
Archive:  wildfly-8.2.0.Final.zip
   creating: wildfly-8.2.0.Final/
 
. . .
 
  inflating: wildfly-8.2.0.Final/standalone/configuration/mgmt-users.properties  
   creating: wildfly-8.2.0.Final/domain/tmp/auth/
   creating: wildfly-8.2.0.Final/standalone/tmp/auth/
 ---> 551e8b1db275
Removing intermediate container c7d808526af6
Step 13 : ENV JBOSS_HOME /opt/jboss/wildfly
 ---> Running in 7709c6f468ef
 ---> 0cec4b9ccd6b
Removing intermediate container 7709c6f468ef
Step 14 : EXPOSE 8080 9990
 ---> Running in dd053271b09e
 ---> 0281986b0ed8
Removing intermediate container dd053271b09e
Step 15 : CMD /opt/jboss/wildfly/bin/standalone.sh -b 0.0.0.0
 ---> Running in fb29091de599
 ---> 6a1c4acf3f78
Removing intermediate container fb29091de599
Successfully built 6a1c4acf3f78
```

The list of Docker images can once again be seen as:

```
wildfly-ubuntu> docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
wildfly-ubuntu      latest              6a1c4acf3f78        5 minutes ago       749.5 MB
<none>              <none>              1a5e1aeadc85        13 minutes ago      607.7 MB
wildfly-centos      latest              97c8780a7d6a        About an hour ago   619.6 MB
centos              latest              ae0c2d0bdc10        2 weeks ago         224 MB
jboss/wildfly       latest              365390553f92        4 weeks ago         948.7 MB
ubuntu              latest              5506de2b643b        4 weeks ago         199.3 MB
```

Docker image can run with docker run command. Some other related commands are:

- `docker ps`: Lists containers
- `docker stop <id>`: Stops the container with the given <id>

Run CentOS image as shown below. Specifying -i option will make it interactive and -t option allocates a pseudo-TTY. And port 8080 from the container is made accessible on port 80 of the container.

```
~> docker run -i -t -p 80:8080 wildfly-centos
=========================================================================
 
  JBoss Bootstrap Environment
 
  JBOSS_HOME: /opt/jboss/wildfly
 
  JAVA: /usr/lib/jvm/java/bin/java
 
  JAVA_OPTS:  -server -Xms64m -Xmx512m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true
 
=========================================================================
 
22:20:52,769 INFO  [org.jboss.modules] (main) JBoss Modules version 1.3.3.Final
22:20:53,038 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.2.Final
22:20:53,120 INFO  [org.jboss.as] (MSC service thread 1-7) JBAS015899: WildFly 8.2.0.Final "Tweek" starting
22:20:54,176 INFO  [org.jboss.as.server] (Controller Boot Thread) JBAS015888: Creating http management service using socket-binding (management-http)
22:20:54,197 INFO  [org.xnio] (MSC service thread 1-10) XNIO version 3.3.0.Final
22:20:54,206 INFO  [org.xnio.nio] (MSC service thread 1-10) XNIO NIO Implementation Version 3.3.0.Final
22:20:54,239 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 31) WFLYIO001: Worker 'default' has auto-configured to 16 core threads with 128 task threads based on your 8 available processors
22:20:54,265 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 32) JBAS010280: Activating Infinispan subsystem.
22:20:54,302 INFO  [org.jboss.as.security] (ServerService Thread Pool -- 45) JBAS013171: Activating Security Subsystem
22:20:54,314 WARN  [org.jboss.as.txn] (ServerService Thread Pool -- 46) JBAS010153: Node identifier property is set to the default value. Please make sure it is unique.
22:20:54,345 INFO  [org.jboss.as.jsf] (ServerService Thread Pool -- 38) JBAS012615: Activated the following JSF Implementations: [main]
22:20:54,363 INFO  [org.jboss.as.naming] (ServerService Thread Pool -- 40) JBAS011800: Activating Naming Subsystem
22:20:54,374 INFO  [org.jboss.as.security] (MSC service thread 1-8) JBAS013170: Current PicketBox version=4.0.21.Final
22:20:54,419 INFO  [org.jboss.as.webservices] (ServerService Thread Pool -- 48) JBAS015537: Activating WebServices Extension
22:20:54,486 INFO  [org.jboss.as.connector.logging] (MSC service thread 1-7) JBAS010408: Starting JCA Subsystem (IronJacamar 1.1.9.Final)
22:20:54,573 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-7) JBAS017502: Undertow 1.1.0.Final starting
22:20:54,579 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 47) JBAS017502: Undertow 1.1.0.Final starting
22:20:54,586 INFO  [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool -- 27) JBAS010403: Deploying JDBC-compliant driver class org.h2.Driver (version 1.3)
22:20:54,644 INFO  [org.jboss.remoting] (MSC service thread 1-10) JBoss Remoting version 4.0.6.Final
22:20:54,658 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-1) JBAS010417: Started Driver service with driver-name = h2
22:20:54,763 INFO  [org.jboss.as.naming] (MSC service thread 1-1) JBAS011802: Starting Naming Service
22:20:54,761 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-16) JBAS015400: Bound mail session [java:jboss/mail/Default]
22:20:56,387 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 47) JBAS017527: Creating file handler for path /opt/jboss/wildfly/welcome-content
22:20:56,422 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-10) JBAS017525: Started server default-server.
22:20:56,544 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) JBAS017531: Host default-host starting
22:20:56,712 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-10) JBAS017519: Undertow HTTP listener default listening on /0.0.0.0:8080
22:20:56,975 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-14) JBAS010400: Bound data source [java:jboss/datasources/ExampleDS]
22:20:56,976 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-15) JBAS015012: Started FileSystemDeploymentService for directory /opt/jboss/wildfly/standalone/deployments
22:20:57,172 INFO  [org.jboss.ws.common.management] (MSC service thread 1-10) JBWS022052: Starting JBoss Web Services - Stack CXF Server 4.3.2.Final
22:20:57,239 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
22:20:57,240 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
22:20:57,241 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.2.0.Final "Tweek" started in 4836ms - Started 184 of 234 services (82 services are lazy, passive or on-demand)
```

In a different shell, get the container’s IP address as:


```
~> boot2docker ip
 
The VM's Host only interface IP address is: 192.168.59.103
```

And then access WildFly at [http://192.168.59.103](http://192.168.59.103/).

Similarly, running the WildFly Ubuntu image shows:


```
wildfly-ubuntu> docker run -i -t -p 80:8080 wildfly-ubuntu
=========================================================================
 
  JBoss Bootstrap Environment
 
  JBOSS_HOME: /opt/jboss/wildfly
 
  JAVA: /usr/lib/jvm/java-7-openjdk-amd64/bin/java
 
  JAVA_OPTS:  -server -Xms64m -Xmx512m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true
 
=========================================================================
 
22:34:07,612 INFO  [org.jboss.modules] (main) JBoss Modules version 1.3.3.Final
22:34:07,911 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.2.Final
22:34:07,996 INFO  [org.jboss.as] (MSC service thread 1-7) JBAS015899: WildFly 8.2.0.Final "Tweek" starting
22:34:09,076 INFO  [org.jboss.as.server] (Controller Boot Thread) JBAS015888: Creating http management service using socket-binding (management-http)
22:34:09,097 INFO  [org.xnio] (MSC service thread 1-9) XNIO version 3.3.0.Final
22:34:09,106 INFO  [org.xnio.nio] (MSC service thread 1-9) XNIO NIO Implementation Version 3.3.0.Final
22:34:09,136 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 32) JBAS010280: Activating Infinispan subsystem.
22:34:09,170 INFO  [org.jboss.as.security] (ServerService Thread Pool -- 45) JBAS013171: Activating Security Subsystem
22:34:09,180 WARN  [org.jboss.as.txn] (ServerService Thread Pool -- 46) JBAS010153: Node identifier property is set to the default value. Please make sure it is unique.
22:34:09,191 INFO  [org.jboss.as.jsf] (ServerService Thread Pool -- 38) JBAS012615: Activated the following JSF Implementations: [main]
22:34:09,207 INFO  [org.jboss.as.naming] (ServerService Thread Pool -- 40) JBAS011800: Activating Naming Subsystem
22:34:09,212 INFO  [org.jboss.as.security] (MSC service thread 1-2) JBAS013170: Current PicketBox version=4.0.21.Final
22:34:09,230 INFO  [org.jboss.as.webservices] (ServerService Thread Pool -- 48) JBAS015537: Activating WebServices Extension
22:34:09,317 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-6) JBAS017502: Undertow 1.1.0.Final starting
22:34:09,317 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 47) JBAS017502: Undertow 1.1.0.Final starting
22:34:09,323 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 31) WFLYIO001: Worker 'default' has auto-configured to 16 core threads with 128 task threads based on your 8 available processors
22:34:09,413 INFO  [org.jboss.as.connector.logging] (MSC service thread 1-15) JBAS010408: Starting JCA Subsystem (IronJacamar 1.1.9.Final)
22:34:09,617 INFO  [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool -- 27) JBAS010403: Deploying JDBC-compliant driver class org.h2.Driver (version 1.3)
22:34:09,658 INFO  [org.jboss.as.naming] (MSC service thread 1-10) JBAS011802: Starting Naming Service
22:34:09,674 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-1) JBAS015400: Bound mail session [java:jboss/mail/Default]
22:34:09,692 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-12) JBAS010417: Started Driver service with driver-name = h2
22:34:09,693 INFO  [org.jboss.remoting] (MSC service thread 1-9) JBoss Remoting version 4.0.6.Final
22:34:11,117 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 47) JBAS017527: Creating file handler for path /opt/jboss/wildfly/welcome-content
22:34:11,134 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-5) JBAS017525: Started server default-server.
22:34:11,206 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-14) JBAS017531: Host default-host starting
22:34:11,377 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-5) JBAS017519: Undertow HTTP listener default listening on /0.0.0.0:8080
22:34:11,576 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-2) JBAS015012: Started FileSystemDeploymentService for directory /opt/jboss/wildfly/standalone/deployments
22:34:11,627 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-8) JBAS010400: Bound data source [java:jboss/datasources/ExampleDS]
22:34:11,847 INFO  [org.jboss.ws.common.management] (MSC service thread 1-6) JBWS022052: Starting JBoss Web Services - Stack CXF Server 4.3.2.Final
22:34:11,911 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
22:34:11,912 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
22:34:11,912 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.2.0.Final "Tweek" started in 4762ms - Started 184 of 234 services (82 services are lazy, passive or on-demand)
```

You can login to the host VM as shown:

``
boot2docker ssh
```

Different layers of the image are stored in /var/lib/docker directory as shown:

```
docker-images> boot2docker ssh
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
Boot2Docker version 1.3.2, build master : 495c19a - Mon Nov 24 20:40:58 UTC 2014
Docker version 1.3.2, build 39fa2fa
docker@boot2docker:~$ cd /var/lib/docker/
docker@boot2docker:/mnt/sda1/var/lib/docker$ ls -la
total 64
drwxr-xr-x   10 root     root          4096 Nov 24 23:33 ./
drwxr-xr-x    4 root     root          4096 Nov 24 19:15 ../
drwxr-xr-x    5 root     root          4096 Nov 24 19:16 aufs/
drwx------   17 root     root          4096 Nov 24 23:33 containers/
drwx------    3 root     root          4096 Nov 24 19:16 execdriver/
drwx------   84 root     root         12288 Nov 24 23:28 graph/
drwx------    2 root     root          4096 Nov 25 14:50 init/
-rw-r--r--    1 root     root         11264 Nov 24 23:33 linkgraph.db
-rw-------    1 root     root           565 Nov 24 23:28 repositories-aufs
drwx------    2 root     root          4096 Nov 24 23:28 tmp/
drwx------    2 root     root          4096 Nov 24 19:21 trust/
drwx------    2 root     root          4096 Nov 24 19:16 volumes/
```

VM image on Mac OSX is stored in `~/VirtualBox VMs/boot2docker-vm` directory. This directory can grow up rather quickly if the intermediate containers are not removed. boot2docker-vm.vmdk on my machine is ~5GB for these different images.

You can reset it by running the following commands (WARNING: This will destroy all images you’ve downloaded and built so far):

```
boot2docker down
boot2docker destroy
boot2docker init
boot2docker up
```

Containers, as you can imagine, have a memory foot print.

More Docker goodness is coming in subsequent blogs!

---

Original source: [Create your own Docker image (Tech Tip #57)](http://blog.arungupta.me/2014/11/create-own-docker-image-techtip57/)