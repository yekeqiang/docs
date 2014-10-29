# Docker Web Services with Consul

---

##### Author: Graham Jenson

---

Do you need a solution to scale your web services both vertically and horizontally, with load balancing and health checking? [Consul](http://www.consul.io/) with [Docker](https://www.docker.com/) and [Nginx](http://nginx.org/) can help!

In this post I will describe how to use [Consul](http://www.consul.io/) (a service registry) and [Nginx](http://nginx.org/) (with [srv-router](https://github.com/vlipco/srv-router)) running in [Docker](https://www.docker.com/) containers to load balance across multiple services.

*Note: For OSX users [boot2docker](http://boot2docker.io/) is required*

## Consul

Consul is a service registry that uses the DNS protocol to return a list of healthy services, in a random order (for load balancing).

Consul can be run as a docker container with:

```
docker run -it -h node \  
 -p 8500:8500 \
 -p 8600:53/udp \
 progrium/consul \
 -server \
 -bootstrap \
 -advertise $DOCKER_IP
```
 
This runs Consul using the [progrium/consul](https://github.com/progrium/docker-consul) image, mapping to the ports 8500 to the web interface, and `8600` to the DNS interface. node is the given hostname, and it is being started as a server in bootstrap mode.

Finally, DOCKER_IP is the address of the docker host and is used for advertising. If you are using boot2docker export DOCKER_IP=`boot2docker ip` will assign it correctly.

A service can be registered with HTTP:

```
curl -XPUT \  
$DOCKER_IP:8500/v1/agent/service/register \
-d '{
 "ID": "simple_instance_1",
 "Name":"simple",
 "Port": 8000, 
 "tags": ["tag"]
}'
```

This service can be discovered using the DNS query tool dig:

```
dig @$DOCKER_IP -p 8600 \  
 tag.simple.service.consul
```

## Nginx with srv-router

To have multiple instances of a service running on a single machine they have to be exposed on different ports. DNS **A records** only contain an IP address, where **[SRV records](http://en.wikipedia.org/wiki/SRV_record)** contain the port of the service.

[srv-router](https://github.com/vlipco/srv-router) uses SRV records to correctly redirect an incoming request to the address and port. srv-router is a Docker container that runs an Nginx server modified with a Lua script to route requests using SRV records.

To run srv-router:

```
docker run -it -p 80:80 \  
 --net host \
 -e "NS_IP=$DOCKER_IP" \
 -e "NS_PORT=8600" \
 -e "TARGET=simple.service.consul" \
 -e "DOMAINS=$DOCKER_IP" \
 vlipco/srv-router
```

This will start the srv-router on port 80. 

`--net` host gives the container the same network interface as the Docker host to give it access to all other containers. The `NS_IP` and `NS_PORT` point towards the Consul server.

The srv-router when called will query Consul for `home.simple.service.consul`, then route to the address and port that is returned. This is the tags namespace in Consul, so each consul service must have the home tag.

## Start to Finish

Using a simple Docker web-service described [here](http://www.maori.geek.nz/post/the_smallest_docker_web_service_that_could), lets set up a "*server*" that uses load-balances across multiple services using Consul and srv-router.

First, lets run two servers on port 8001 and 8002:

```
docker run -it -p 8001:80 python/server  
docker run -it -p 8002:80 python/server
```
  
Now lets start Consul:

```
docker run -it -h node -p 8500:8500 -p 8600:53/udp progrium/consul -server -bootstrap -advertise $DOCKER_IP
```
  
Now srv-router:

```
docker run -it -p 80:80 --net host -e "NS_IP=$DOCKER_IP" -e "NS_PORT=8600" -e "TARGET=simple.service.consul" -e "DOMAINS=$DOCKER_IP" vlipco/srv-router 
```
 
Finally, let register the services:

```
curl -XPUT \  
$DOCKER_IP:8500/v1/agent/service/register \
-d '{
 "ID": "simple_instance_1",
 "Name":"simple",
 "Port": 8001, 
 "tags": ["home"]
}'
```
```
curl -XPUT \  
$DOCKER_IP:8500/v1/agent/service/register \
-d '{
 "ID": "simple_instance_2",
 "Name":"simple",
 "Port": 8002, 
 "tags": ["home"]
}'
```

Lets check Consul has the services with:

```
dig @$DOCKER_IP -p 8600 \  
home.simple.service.consul SRV 
```
 
Finally, lets call the services with `curl $DOCKER_IP`.

What should happen is:

- srv-router gets the request and asks Consul for the SRV records for `home.simple.service.consul`
- Consul returns in a random order the two SRV records for `simple_instance_1` on port `8001` and `simple_instance_2` on port `8002`.
- srv-router then routes the request to one of those services
- then the service handles the request and returns a response

## Future Things and Stuff

In this post I did not describe how Consul can do health checking of the services, or how to streamline registering services with registrator. I will definitely go over these aspects in further posts.

Using Docker to contain and distribute services and Consul with srv-router to load-balance across them could greatly reduce growing pains of services. In conclusion, Docker is awesome.

## Other Resources

[Consul HTTP API](http://www.consul.io/docs/agent/http.html)

[The Docker Book](http://www.amazon.com/gp/product/B00LRROTI4/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B00LRROTI4&linkCode=as2&tag=maor01-20&linkId=2QKZTS7EW7H2VZRM)

[progrium/docker-consul](https://github.com/progrium/docker-consul)

[progrium/registrator](https://github.com/progrium/registrator)

[The Future of Docker](http://www.infoq.com/news/2014/08/the-future-of-docker)

----

Original source: [Docker Web Services with Consul](http://www.maori.geek.nz/post/docker_web_services_with_consul)