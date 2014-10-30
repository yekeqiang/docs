# Docker RabbitMQ Cluster

---

##### Author: Biju Kunjummen

---

I have been trying to create a [Docker](http://docker.com/) based [RabbitMQ](https://www.rabbitmq.com/) cluster on and off for sometime and got it working today - fairly basic and flaky but could be a good starting point for others to improve on.

This is how the sample cluster looks on my machine, this is a typical cluster described in the RabbitMQ clustering guide available [here](https://www.rabbitmq.com/clustering.html). As recommended at the site, there are 2 disk based nodes and 1 RAM based node here.


![alt](http://resource.docker.cn/sample-rabbitmq-cluster.png)


To quickly replicate this, you only need to have [fig](http://www.fig.sh/index.html) in your machine, just create a fig.yml file with the following entry:

```
rabbit1:
  image: bijukunjummen/rabbitmq-server
  hostname: rabbit1
  ports:
    - "5672:5672"
    - "15672:15672"

rabbit2:
  image: bijukunjummen/rabbitmq-server
  hostname: rabbit2
  links:
    - rabbit1
  environment:
    - CLUSTERED=true
    - CLUSTER_WITH=rabbit1
    - RAM_NODE=true

rabbit3:
  image: bijukunjummen/rabbitmq-server
  hostname: rabbit3
  links:
    - rabbit1
    - rabbit2
  environment:
    - CLUSTERED=true
    - CLUSTER_WITH=rabbit1
```

and in the folder holding this file, run:

```
fig up
```

That is it!, the entire cluster should come up. If you need more nodes, just modify the fig.yml file.

The docker files for creating the dockerized rabbitmq-server is available at my github repo [here](https://github.com/bijukunjummen/docker-rabbitmq-cluster) and the "rabbitmq-server" image itself is [here](https://registry.hub.docker.com/u/bijukunjummen/rabbitmq-server/) at the docker hub.

## References:


- The base rabbitmq image is somewhat based on cthulhuology's docker rabbitmq image: [https://github.com/cthulhuology/docker-rabbitmq](https://github.com/cthulhuology/docker-rabbitmq)
- Docker file: [https://github.com/bijukunjummen/docker-rabbitmq-cluster](https://github.com/bijukunjummen/docker-rabbitmq-cluster)
- Docker image: [https://registry.hub.docker.com/u/bijukunjummen/rabbitmq-server/](https://registry.hub.docker.com/u/bijukunjummen/rabbitmq-server/)


---

Original source: [Docker RabbitMQ Cluster](http://java.dzone.com/articles/docker-rabbitmq-cluster)