# Local Development with Docker and Fig


---

Author: Nic Grayson

---

This setup lets you route through a running fig project to an app you start on your machine outside of docker

## Motivation

When developing locally you frequently need to run dependencies of your application locally. This can be problematic if you are running a different operating system than your hosting environment. Frequently database versions or dependencies on other installed and/or running applications can cause the ‘It works in my machine!’ problem. This project allows developers to run a copy of production while running a local copy of any work in progess applications. Running a fig project like this allows for a very fast development feedback cycle.

## Required Tools to Install

[docker](http://docker.com/), [fig](http://fig.sh/), and [boot2docker](http://boot2docker.io/)

### Docker

We use docker to run all of our databases, applications, and wiring. Installation documents [here](https://docs.docker.com/installation/#installation).

### Fig

Fig allows you to start up docker containers and cordinate how they run. Installation documents [here](http://www.fig.sh/install.html).

### boot2docker

Boot2docker enables non-linux operating systems (Mac and Windows) to run docker as if it was native. To install boot2docker follow the docs [here](http://boot2docker.io/). Note that you need have version 1.3.x installed for the volume mounting to work correctly.

## Fig Project Setup

We will need to setup a dns server to ensure that our docker containers and local system resolve our development urls correctly. This will make all of our docker containers run with the –dns flag and our local system use the dns docker container to resolve specific domain names. (e.i. dev.nicgrayson.com and local.nicgrayson.com) We will also setup a load balancer using [haproxy](http://www.haproxy.org/) to route traffic through our docker containers.

### DNS Server

Using the [nicgrayson/boot2docker-dns](https://registry.hub.docker.com/u/nicgrayson/boot2docker-dns) docker image we will add our needed local development domains via docker volumes. You will need to create two zone files like this:

```
$TTL 14400
dev.nicgrayson.com.	14400	IN	SOA	ns2.nicgrayson.com.	ops.nicgrayson.com.	(
						1 ;Serial Number
						14400 ;refresh
						7200 ;retry
						1209600 ;expire
						86400 ;minimum
)
dev.nicgrayson.com.	 14400	IN	NS	ns2.nicgrayson.com.
dev.nicgrayson.com.	 14400	IN	NS	ns1.nicgrayson.com.
dev.nicgrayson.com.	 14400	IN	A	192.168.59.103
*.dev.nicgrayson.com. 14400	IN	A	192.168.59.103
```

```
$TTL 14400
local.nicgrayson.com.	14400	IN	SOA	ns2.nicgrayson.com.	ops.nicgrayson.com.	(
						1 ;Serial Number
						14400 ;refresh
						7200 ;retry
						1209600 ;expire
						86400 ;minimum
)
local.nicgrayson.com.	 14400	IN	NS	ns2.nicgrayson.com.
local.nicgrayson.com.	 14400	IN	NS	ns1.nicgrayson.com.
local.nicgrayson.com.	 14400	IN	A	192.168.59.3
```

Then you will need a file named `customzones` to load these zones:

```
zone "dev.nicgrayson.com" {
  type master;
  file "/data/db.dev.nicgrayson.com";
};

zone "local.nicgrayson.com" {
  type master;
  file "/data/db.local.nicgrayson.com";
};
```

Note that the ip `192.168.59.103` is the boot2docker ip address and `192.168.59.3` is the ip of your local machine from a boot2docker container. This is how the traffic is routed to and from running docker containers.

### Configure Your Computer’s DNS

On a Mac you will need to create a file for each domain you wish to use the docker dns for.

```
sudo sh -c 'echo "nameserver 192.168.59.103" > /etc/resolver/dev.nicgrayson.com'
```

### Load Balancer

This haproxy container is the magic that makes all of this work so well. It will check if a local app is running. If so it will direct traffic to this app, otherwise it will send traffic to the running docker container. To set this up we will use the [dockerfile/haproxy](https://registry.hub.docker.com/u/dockerfile/haproxy/) image and a volume mount. This volume needs to contain an haproxy.cfg file. Here is an example:

```
global
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
  chroot /var/lib/haproxy
  user haproxy
  group haproxy
  stats socket /tmp/haproxy-stats

defaults
  log global
  mode http
  option httplog
  option dontlognull
  option forwardfor
  option httpclose
  timeout connect 5000ms
  timeout client 50s
  timeout server 50s

frontend http
  bind *:80

  acl api hdr_beg(host) -i api.
  acl haproxy hdr_beg(host) -i haproxy.

  use_backend srvs-api if api

  use_backend haproxy-stats if haproxy

backend srvs-api
  balance roundrobin
  server api-local local.nicgrayson.com:4567 check
  server api-docker dev.nicgrayson.com:4567 backup

backend haproxy-stats
  stats enable
  stats uri /
```

This config will check for a running api sinatra app and route to the local system or the docker container.

### Fig.yml File to Put it All Together

```
dns:
  image: nicgrayson/boot2docker-dns:latest
  volumes:
    - dns/:
  ports:
    - "53:53"
    - "53:53/udp"

lb:
  image: dockerfile/haproxy:latest
  dns: 172.17.42.1
  volumes:
    - haproxy/:/haproxy-override
  ports:
    - "80:80"
  links:
    - dns

api:
  image: nicgrayson/fig-api:latest
  ports:
    - "4567:4567"
  links:
    - lb
```

## Example Project Output

`fig up` without the local app started:

```
Creating dockerlocaldevelopment_dns_1...
Creating dockerlocaldevelopment_lb_1...
Creating dockerlocaldevelopment_api_1...
Attaching to dockerlocaldevelopment_dns_1, dockerlocaldevelopment_lb_1, dockerlocaldevelopment_api_1
lb_1  | [WARNING] 312/041801 (10) : Server srvs-api/api-local is DOWN, reason: Layer4 connection problem, info: "Connection refused", check duration: 1ms. 0 active and 1 backup servers left. Running on backup. 0 sessions active, 0 requeued, 0 remaining in queue.
api_1 | [2014-11-09 04:18:02] INFO  WEBrick 1.3.1
api_1 | [2014-11-09 04:18:02] INFO  ruby 2.1.3 (2014-09-19) [x86_64-linux]
api_1 | == Sinatra/1.4.5 has taken the stage on 4567 for development with backup from WEBrick
api_1 | [2014-11-09 04:18:02] INFO  WEBrick::HTTPServer#start: pid=7 port=4567
```

log when app starts up:

```
lb_1  | [WARNING] 312/042020 (10) : Server srvs-api/api-local is UP, reason: Layer4 check passed, check duration: 2ms. 1 active and 1 backup servers online. 0 sessions requeued, 0 total in queue.
```

You can see in the output above that the local copy of the app reported down then the load balancer detected that the app started. This causes all traffic info api.dev.nicgrayson.com to instead route to the local, outside of docker, instance of the application. This project can be found on my [Github](https://github.com/nicgrayson/docker-local-development).

## Take Aways

Running a local copy of your production services should be easy and fast. Swaping out one piece of your environment to work on locally is fantastic. Sharing a fig project with your team ensures that everyone is using the same dependencies to develop.

---

Original source: [Local Development with Docker and Fig](http://nicgrayson.com/docker-local-dev/)