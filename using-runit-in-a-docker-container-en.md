# Using Runit in a Docker Container

---

Author: Maximilian Güntner

---

While the [phusion image](https://phusion.github.io/baseimage-docker/) is great for building Docker containers, it already comes with a lot of stuff that you don’t need in every container you start (like sshd or cron). The phusion image is also based on Ubuntu 14.04 which may not have all packages in the repositories that you need for your container.

I decided to use Debian Jessie for my Docker container and rewritting/porting the phusion image was out of the question (although there are “ports” for [CentOS](https://github.com/pokle/centos-baseimage) and [Debian Wheezy](https://github.com/mota/baseimage-debian)). However, I liked the idea of using runit as the `ENTRYPOINT` for my container.

This article describes how to use [runit](http://smarden.org/runit/) inside a Docker container for service startup and supervision with a small memory footprint (compared to alternatives like [supervisord](http://supervisord.org/)). All instructions have been tested with the `debian:jessie` image but should work with any other distribution.

## ENTRYPOINT [“Runit”]

The problem of runit is that is has been designed to be an init process (PID 1) and therefore does not honor environment variables that are being passed to it. This becomes a problem if you need to access these in a run script like in this example where `your_app` needs to connect to a linked PostgreSQL database.


- #### /etc/service/your_app/run

```
#!/bin/bash
exec 2>&1
exec your_app --db=$DB_PORT_5432_TCP_ADDR:$DB_PORT_5432_TCP_PORT
```

If you just define `ENTRYPOINT` to be `/usr/sbin/runsvdir-start` you will end up with `$DB_PORT_5432_TCP_{ADDR,PORT}` not being defined in the run script and `your_app` will not be able to connect to the database.

## There! I fixed it!

An easy hack is to create a bootstrapper which makes the environment variables available to the container.

First we need to create a script that dumps the environment variables that are being passed by docker to a file. After this we simply `exec` the runit process.

- #### /usr/sbin/runit_bootstrap

```
#!/bin/bash
export > /etc/envvars
exec /usr/sbin/runsvdir-start
```

We can then source the file `/etc/envvars` in the run script from above:

- #### /etc/service/your_app/run

```
#!/bin/bash
exec 2>&1
source /etc/envvars
exec your_app --db=$DB_PORT_5432_TCP_ADDR:$DB_PORT_5432_TCP_PORT
```

For reference, this is how the Dockerfile would look like:

- #### Dockerfile

```
# DOCKER-VERSION 1.3.1
# VERSION 0.1
FROM debian:jessie
MAINTAINER Maximilian Güntner <maximilian.guentner@gmail.com>

# runit depends on /etc/inittab which is not present in debian:jessie
RUN touch /etc/inittab
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y -q runit

RUN mkdir -p /etc/service/your_app/
COPY ./your_app/run /etc/service/your_app/run

COPY runit_bootstrap /usr/sbin/runit_bootstrap
RUN chmod 755 /usr/sbin/runit_bootstrap
ENTRYPOINT ["/usr/sbin/runit_bootstrap"]
```

Have fun with (easy) service supervison in Docker!

You can also check out [a Docker image where I used this method](https://github.com/mguentner/docker-renderd-osm).

---

Original source: [Using Runit in a Docker Container](http://www.sourcediver.org/blog/2014/11/17/using-runit-in-a-docker-container/)
