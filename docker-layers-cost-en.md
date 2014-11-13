# Docker layers cost

---

##### Author: Manuel Vacelet

---

After having run docker in production since spring for [mytuleap.com](http://mytuleap.com/), we are converting all [Tuleap](https://tuleap.org/) CI/CD to docker since September. As Dylan [already wrote about](https://www.tuleap.org/tuleap-and-continuous-integration), Jenkins/Gerrit is a very central piece in our delivery process and we have to be very careful with changes.

One of the critical part is the build time for each testing job.

Tuleap project benefits from [Rackspace Open Source program](https://www.tuleap.org/how-tuleap-project-uses-rackspace) but we try to be thrifty with those resources: we only provision servers when there is a build to run thanks to Jenkins and JClouds.

This means that we spawn new slaves 10 times a day. That’s great from an engineering point of view but the direct consequence is that we have to pull docker build images 10 times a day and this takes time.

## Reduce the size of downloaded images

One of the key features of Docker is the layered filesystem for images. By default, the various operations made in a container that implies file access are persisted on a special, copy on write, filesystem (think of git with all your files).

This is especially true when you are using a Dockerfile. Each statement (RUN, COPY, etc) will result in a new layer.

This is handy for development and layer re-use as, once a command is done, it’s cached and might be used in another build. The most common usage is, when you are building a new image, you don’t want to re-run all the command from scratch (and re-download the whole internet every once in a while).

This is what we used to do at [Enalean](https://enalean.com/) when building our images. Our Dockerfiles usually ended to look like:

```
FROM centos:centos6
# Update to last version
RUN yum -y update; yum clean all
RUN yum -y install php && yum clean all
RUN yum -y install php-soap && yum clean all
RUN yum -y install php-mysql && yum clean all
RUN yum -y install php-gd && yum clean all
RUN yum -y install php-process && yum clean all
RUN yum -y install php-xml && yum clean all
RUN yum -y install php-mbstring && yum clean all
RUN yum -y install mysql-server && yum clean all
# EPEL
RUN rpm -i http://mir01.syntis.net/epel/6/i386/epel-release-6-8.noarch.rpm
RUN yum -y install php-pecl-xdebug && yum clean all
# Uncomment when EPEL start shitting bricks (like 404)
# RUN sed -i ‘s/#baseurl/baseurl/’ /etc/yum.repos.d/epel.repo
# RUN sed -i ‘s/mirrorlist/#mirrorlist/’ /etc/yum.repos.d/epel.repo
# Repoforge
RUN rpm —import http://apt.sw.be/RPM-GPG-KEY.dag.txt
RUN rpm -i http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
ADD rpmforge.repo /etc/yum.repos.d/
RUN yum -y install —enablerepo=rpmforge-extras git && yum clean all
```

Last week, [@christianbayle](https://twitter.com/christianbayle) announced on tuleap-devel that [he managed to greatly reduce the size](https://tuleap.net/plugins/forumml/message.php?group_id=101&topic=32016&list=1) of one of your docker image, [tuleap-aio](https://registry.hub.docker.com/u/enalean/tuleap-aio/), by building a fresh new base OS, smaller than default centos:centos6.

I was surprised of those results as the new base OS was only 50MB smaller than official image but Christian’s tuleap-aio was 600MB smaller than ours.

After digging a bit, the other difference between the 2 approaches was that Christan’s used to group all yum install commands. And this was the trick.

We quickly adapted it to our CI images and now with the grouped layer approach, [dockerfile](https://github.com/Enalean/docker-tuleap-test-simpletest/blob/c6-php53/Dockerfile) looks like:

```
FROM centos:centos6
RUN rpm —import http://apt.sw.be/RPM-GPG-KEY.dag.txt && \
 rpm -i http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm && \
 rpm -i http://mir01.syntis.net/epel/6/i386/epel-release-6-8.noarch.rpm
COPY *.repo /etc/yum.repos.d/
RUN yum -y —enablerepo=rpmforge-extras install php \
 php-pecl-xdebug \
 php-soap \
 php-mysql \
 php-gd \
 php-process \
 php-xml \
 php-mbstring \
 mysql-server \
 php-zendframework \
 htmlpurifier \
 jpgraph-tuleap \
 php-pear-Mail-mimeDecode \
 rcs \
 cvs \
 php-guzzle \
 php-password-compat \
 unzip \
 tar \
 subversion \
 bzip2 \
 php-pecl-xdebug \
 git \
 && yum clean all
```

Resulting images size evolution:

- [enalean/tuleap-simpletest:c5-php51](https://registry.hub.docker.com/u/enalean/tuleap-simpletest/) (1.473 GB -> 991.2 MB)
- [enalean/tuleap-simpletest:c6-php53](https://registry.hub.docker.com/u/enalean/tuleap-simpletest/) (909.6 MB -> 612 MB)
- [enalean/tuleap-simpletest:c6-php54](https://registry.hub.docker.com/u/enalean/tuleap-simpletest/) (1.025 GB -> 697.7 MB)
- [enalean/tuleap-simpletest:c6-php55](https://registry.hub.docker.com/u/enalean/tuleap-simpletest/) (1.002 GB -> 706 MB)
- [enalean/tuleap-rest-test:latest](https://registry.hub.docker.com/u/enalean/tuleap-test-rest/) (1.178 GB -> 409.9 MB)

As you can see, the gain is quite impressive.

## Takeaway

So as conclusion I would recommend to:

- One run statement per command while you are under image development (so you leverage on cache)
- When everything is ready, group as much as possible in one layer so save space and kittens.

---

##### Original source: [Docker layers cost](https://medium.com/@vaceletm/docker-layers-cost-b28cb13cb627)