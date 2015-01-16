# Pushing Docker images to Registry (Tech Tip #58)

---

Author: Arun Gupta

---

[Tech Tip #57](http://blog.arungupta.me/2014/11/create-own-docker-image-techtip57/) explained how to create your own Docker images. That particular blog specifically showed how to build your own WildFly Docker images on CentOS and Ubuntu. Now you are ready to share your images with rest of the world. That’s where [Docker Hub](https://hub.docker.com/) comes in handy.

Docker Hub is the “distribution component” of Docker, or a place to store and search images. From the [Getting Started with Docker Hub](https://docs.docker.com/userguide/dockerhub/) docs …

*The Docker Hub is a centralized resource for working with Docker and its components. Docker Hub helps you collaborate with colleagues and get the most out of Docker*.

Starting and pushing images to with Docker Hub is pretty straight forward.

- #### Pushing images to Docker Hub require an account. It can be created as explained here. Or rather easily by using docker login command.

```
wildfly-centos> docker login
Username: arungupta
Password: 
Email: arun.gupta@gmail.com
Login Succeeded
``` 

Searching on WildFly shows there are 72 images:

```
wildfly-centos> docker search wildfly
NAME                                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
jboss/wildfly                            WildFly application server image                42                   [OK]
sewatech/wildfly                         Debian + WildFly 8.1.0.Final with OpenJDK ...   1                    [OK]
kamcord/wildfly                                                                          1                    
openshift/wildfly-8-centos                                                               1                    [OK]
abstractj/wildfly                        AeroGear Wildfly Docker image                   1                    
jsightler/wildfly_nightly                Nightly build from wildfly's github master...   1                    
centos/wildfly                           CentOS based WildFly Docker image               1                    
aerogear/unifiedpush-wildfly                                                             1                    [OK]
t0nyhays/wildfly                                                                         1                    [OK]
tsuckow/wildfly-propeller                Dockerization of my application *Propeller...   0                    [OK]
n3ziniuka5/wildfly                                                                       0                    [OK]
snasello/wildfly                                                                         0                    [OK]
jboss/keycloak-adapter-wildfly                                                           0                    [OK]
emsouza/wildfly                                                                          0                    [OK]
sillenttroll/wildfly-java-8              WildFly container with java 8                   0                    [OK]
jboss/switchyard-wildfly                                                                 0                    [OK]
n3ziniuka5/wildfly-jrebel                                                                0                    [OK]
dfranssen/docker-wildfly                                                                 0                    [OK]
wildflyext/wildfly-camel                 WildFly with Camel Subsystem                    0                    
ianblenke/wildfly                                                                        0                    [OK]
arcamael/docker-wildfly                                                                  0                    [OK]
dmartin/wildfly                                                                          0                    [OK]
pires/wildfly-cluster-backend                                                            0                    [OK]
aerogear/push-quickstarts-wildfly-dev                                                    0                    [OK]
faga/wildfly                             Wildfly application server with ubuntu.         0                    
abstractj/unifiedpush-wildfly            AeroGear Wildfly Docker image                   0                    
murad/wildfly                            - oficial centos image - java JDK "1.8.0_0...   0                    
aerogear/unifiedpush-wildfly-dev                                                         0                    [OK]
ianblenke/wildfly-cluster                                                                0                    [OK]
blackhm/wildfly                                                                          0                    
khipu/wildfly8                                                                           0                    [OK]
rowanto/docker-wheezy-wildfly-java8                                                      0                    [OK]
ordercloud/wildfly                                                                       0                    
lavaliere/je-wildfly                     A Jenkins Enterprise demo master with a Wi...   0                    
adorsys/wildfly                          Ubuntu - Wildfly - Base Image                   0                    
akalliya/wildfly                                                                         0                    
lavaliere/joc-wildfly                    Jenkins Operations Center master with an a...   0                    
tdiesler/wildfly                                                                         0                    
apiman/on-wildfly8                                                                       0                    [OK]
rowanto/docker-wheezy-wildfly-java8-ex                                                   0                    [OK]
arcamael/blog-wildfly                                                                    0                    
lavaliere/wildfly                                                                        0                    
jfaerman/wildfly                                                                         0                    
yntelectual/wildfly                                                                      0                    
svenvintges/wildfly                                                                      0                    
dbrotsky/wildfly                                                                         0                    
luksa/wildfly                                                                            0                    
tdiesler/wildfly-camel                                                                   0                    
blackhm/wildfly-junixsocket                                                              0                    
abstractj/unifiedpush-wildfly-dev        AeroGear UnifiedPush server developer envi...   0                    
abstractj/push-quickstarts-wildfly-dev   AeroGear UnifiedPush Quickstarts developer...   0                    
bn3t/wildfly-wicket-examples             An image to run the wicket-examples on wil...   0                    
lavaliere/wildfly-1                                                                      0                    
munchee13/wildfly-node                                                                   0                    
munchee13/wildfly-manager                                                                0                    
munchee13/wildfly-dandd                                                                  0                    
munchee13/wildfly-admin                                                                  0                    
bparees/wildfly-8-centos                                                                 0                    
lecoz/wildflysiolapie                    fedora latest, jdk1.8.0_25, wildfly-8.1.0....   0                    
lecoz/wildflysshsiolapie                 wildfly 8.1.0.Final, jdk1.8.0_25, sshd, fe...   0                    
wildflyext/example-camel-rest                                                            0                    
pepedigital/wildfly                                                                      0                    [OK]
tsuckow/wildfly                          JBoss Wildfly 8.1.0.Final standalone mode ...   0                    [OK]
mihahribar/wildfly                       Dockerfile for Wildfly running on Ubuntu 1...   0                    [OK]
hpehl/wildfly-domain                     Dockerfiles based on "jboss/wildfly" to se...   0                    [OK]
raynera/wildfly                                                                          0                    [OK]
hpehl/wildfly-standalone                 Dockerfile based on jboss/wildfly to setup...   0                    [OK]
aerogear/wildfly                                                                         0                    [OK]
piegsaj/wildfly                                                                          0                    [OK]
wildflyext/wildfly                       Tagged versions JBoss WildFly                   0
``` 

Official images are tagged jboss/wildfly.

- #### In order to push your own image, it needs to be built as a named image otherwise you’ll get an error as shown:

```
2014/11/26 09:59:37 You cannot push a "root" repository. Please rename your repository in <user>/<repo> (ex: arungupta/wildfly-centos)
``` 

This can be easily done as shown:

```
wildfly-centos> docker build -t="arungupta/wildfly-centos" .
Sending build context to Docker daemon 4.096 kB
Sending build context to Docker daemon 
Step 0 : FROM centos
 ---> ae0c2d0bdc10
Step 1 : MAINTAINER Arun Gupta <arungupta@redhat.com>
 ---> Using cache
 ---> e490dfcb3685
Step 2 : RUN yum -y update && yum clean all
 ---> Using cache
 ---> f212cb9dbcf5
Step 3 : RUN yum -y install xmlstarlet saxon augeas bsdtar unzip && yum clean all
 ---> Using cache
 ---> 28b11e6151f0
Step 4 : RUN groupadd -r jboss -g 1000 && useradd -u 1000 -r -g jboss -m -d /opt/jboss -s /sbin/nologin -c "JBoss user" jboss
 ---> Using cache
 ---> 73603eab89b7
Step 5 : WORKDIR /opt/jboss
 ---> Using cache
 ---> 9a661ae4341b
Step 6 : USER jboss
 ---> Using cache
 ---> 6265153611c7
Step 7 : USER root
 ---> Using cache
 ---> 12ed28a7acb7
Step 8 : RUN yum -y install java-1.7.0-openjdk-devel && yum clean all
 ---> Using cache
 ---> 44c4bb92fa11
Step 9 : USER jboss
 ---> Using cache
 ---> 930cb2a860f7
Step 10 : ENV JAVA_HOME /usr/lib/jvm/java
 ---> Using cache
 ---> fff2c21b0a71
Step 11 : ENV WILDFLY_VERSION 8.2.0.Final
 ---> Using cache
 ---> b7b7ca7a9172
Step 12 : RUN cd $HOME && curl -O http://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.zip && unzip wildfly-$WILDFLY_VERSION.zip && mv $HOME/wildfly-$WILDFLY_VERSION $HOME/wildfly && rm wildfly-$WILDFLY_VERSION.zip
 ---> Using cache
 ---> a1bc79a43c77
Step 13 : ENV JBOSS_HOME /opt/jboss/wildfly
 ---> Using cache
 ---> d46fdd618d55
Step 14 : EXPOSE 8080 9990
 ---> Running in 9c2c2a5ef41c
 ---> 8988c8cbc051
Removing intermediate container 9c2c2a5ef41c
Step 15 : CMD /opt/jboss/wildfly/bin/standalone.sh -b 0.0.0.0
 ---> Running in 9e28c3449ec1
 ---> d989008d1f84
Removing intermediate container 9e28c3449ec1
Successfully built d989008d1f84
``` 

`docker build` command builds the image, `-t` specifies the repository name to be applied to the resulting image.

- #### Once the image is built, it can be verified as:

```
wildfly-centos> docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
arungupta/wildfly-centos   latest              d989008d1f84        14 hours ago        619.6 MB
wildfly-ubuntu             latest              a2e96e76eb10        43 hours ago        749.5 MB
<none>                     <none>              0281986b0ed8        44 hours ago        749.5 MB
<none>                     <none>              1a5e1aeadc85        44 hours ago        607.7 MB
wildfly-centos             latest              97c8780a7d6a        45 hours ago        619.6 MB
registry                   latest              7e2db37c6564        13 days ago         411.6 MB
centos                     latest              ae0c2d0bdc10        3 weeks ago         224 MB
jboss/wildfly              latest              365390553f92        4 weeks ago         948.7 MB
ubuntu                     latest              5506de2b643b        4 weeks ago         199.3 MB
``` 

Notice the first line shows the named image `arungupta/wildfly-centos`.

- #### This image can then be pushed to Docker Hub as:

``` 
wildfly-centos> docker push arungupta/wildfly-centos
The push refers to a repository [arungupta/wildfly-centos] (len: 1)
Sending image list
Pushing repository arungupta/wildfly-centos (1 tags)
511136ea3c5a: Image already pushed, skipping 
5b12ef8fd570: Image already pushed, skipping 
ae0c2d0bdc10: Image already pushed, skipping 
e490dfcb3685: Image successfully pushed 
f212cb9dbcf5: Image successfully pushed 
28b11e6151f0: Image successfully pushed 
73603eab89b7: Image successfully pushed 
9a661ae4341b: Image successfully pushed 
6265153611c7: Image successfully pushed 
12ed28a7acb7: Image successfully pushed 
44c4bb92fa11: Image successfully pushed 
930cb2a860f7: Image successfully pushed 
fff2c21b0a71: Image successfully pushed 
b7b7ca7a9172: Image successfully pushed 
a1bc79a43c77: Image successfully pushed 
d46fdd618d55: Image successfully pushed 
8988c8cbc051: Image successfully pushed 
d989008d1f84: Image successfully pushed 
Pushing tag for rev [d989008d1f84] on {https://cdn-registry-1.docker.io/v1/repositories/arungupta/wildfly-centos/tags/latest}
```
 
- #### And you can verify this by pulling the image:

```
wildfly-centos> docker pull arungupta/wildfly-centos
Pulling repository arungupta/wildfly-centos
d989008d1f84: Download complete 
511136ea3c5a: Download complete 
5b12ef8fd570: Download complete 
ae0c2d0bdc10: Download complete 
e490dfcb3685: Download complete 
f212cb9dbcf5: Download complete 
28b11e6151f0: Download complete 
73603eab89b7: Download complete 
9a661ae4341b: Download complete 
6265153611c7: Download complete 
12ed28a7acb7: Download complete 
44c4bb92fa11: Download complete 
930cb2a860f7: Download complete 
fff2c21b0a71: Download complete 
b7b7ca7a9172: Download complete 
a1bc79a43c77: Download complete 
d46fdd618d55: Download complete 
8988c8cbc051: Download complete 
Status: Image is up to date for arungupta/wildfly-centos:latest
```
 
Enjoy!

---

Original source: [Pushing Docker images to Registry (Tech Tip #58)](http://blog.arungupta.me/2014/11/pushing-docker-images-registry-techtip58/)