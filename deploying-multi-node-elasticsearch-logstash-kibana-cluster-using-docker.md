# Deploying and migrating a multi-node ElasticSearch-Logstash-Kibana cluster using Docker

---


##### Author: Luke Marsden

---

ElasticSearch has exploded in popularity recently and it seems like almost everyone is using it for something, [big](http://www.elasticsearch.org/blog/using-elasticsearch-and-logstash-to-serve-billions-of-searchable-events-for-customers/) or [small](http://brudtkuhl.com/install-elasticsearch-in-5-minutes/).  Running ElasticSearch in production, however, does not have a reputation of being easy.  First, most people don’t just run ElasticSearch, they run the full ELK stack which is ElasticSearch, Logstash and Kibana.  Each of these services plays a role, and for real workloads, that probably means that you want to deploy each service to its own server which introduces complexities around multi-node deployment as well as networking.  Furthermore, once your ELK stack is out there doing its thing, you are going to probably need to upgrade your ElasticSearch box to a larger node at some point because ElasticSearch is notoriously memory hungry.

While [Docker](https://registry.hub.docker.com/) itself is a great way to package up the ElasticSearch, Logstash and Kibana images, deployment, networking and migrations are ops tasks that native Docker doesn’t support.

In this post, we’ll show you how to use [Flocker, an open-source data volume and container manager](https://github.com/ClusterHQ/flocker) to deploy an ELK stack to multiple nodes, and then perform a near seamless database and data volume migration of ElasticSearch from one node to another.

## Setting up ELK

First, a quick overview of the various ELK components and the roles that they play.

- Logstash receives logged messages and relays them to ElasticSearch
- ElasticSearch stores the logged messages in a database
- Kibana connects to ElasticSearch to retrieve the logged messages and presents them in a web interface.

The first thing that we need to do is package up our three applications along with their dependencies into three separate Docker images.  We’ve done this for you and put the result on DockerHub:

- [ElasticSearch](https://registry.hub.docker.com/u/clusterhq/elasticsearch/)
- [Logstash](https://registry.hub.docker.com/u/clusterhq/logstash/)
- [Kibana](https://registry.hub.docker.com/u/clusterhq/kibana/)

There are great tutorials out there for creating Docker images so we won’t go into that here.

## Deploying ELK to a multi-node cluster

Now that we have our Docker images, we are ready to deploy our stack to multiple nodes.

Working with Flocker takes two configuration files: [application configuration](http://docs.clusterhq.com/en/0.2.1/advanced/configuration.html#application-configuration) and [deployment configuration](http://docs.clusterhq.com/en/0.2.1/advanced/configuration.html#deployment-configuration). Let’s look at the application configuration first.

The application configuration is simply a yml file that describes how your application made up of multiple Docker containers talk to each other.  For this reason, we often refer to it as application.yml.  If you are familiar with the Fig tool from Docker, you will instantly recognize many similarities with Flocker’s application yml.

Below is the application.yml that is needed to start all three containers, as well as the port mapping to let them speak with each other, and to create a Flocker-managed Docker data volume in the ElasticSearch container.

```
"version": 1
"applications":
  "elasticsearch":
    "image": "clusterhq/elasticsearch"
    "ports":
    - "internal": 9200
      "external": 9200
    "volume":
      "mountpoint": "/var/lib/elasticsearch/"
  "logstash":
    "image": "clusterhq/logstash"
    "ports":
    - "internal": 5000
      "external": 5000
    "links":
    - "local_port": 9200
      "remote_port": 9200
      "alias": "es"
  "kibana":
    "image": "clusterhq/kibana"
    "ports":
    - "internal": 8080
      "external": 80
```

Let’s take particular note of a few things:

- The ElasticSearch application has a volume and mount point specified, in this case /var/lib/elasticsearch. **One of the major benefits of Flocker is its ability to migrate data volumes between hosts as we will see later**.
- *links* allow containers to communicate even when located on different hosts
- *ports* proxy a port (“*external*”) on the Docker host (accessible to the outside world) to port (“internal”) in the container.

## Deploying ElasticSearch

Now that we have our ELK stack images ready to go and our application.yml defined, we are ready to deploy these containers to multiple hosts.  We specify where we want our containers deployed in the second configuration file mentioned above: the deployment configuration.

In this example, we will deploy each of the services to its own virtual machine (VM).  If you want to follow along, you can use virtually any host and the steps work equally well on VMs, bare metal servers, or any combination thereof. For instance, you might want to run ElasticSearch on bare metal for performance reasons, but run Logstash and Kibana on VMs to keep costs down.  It’s up to you and Flocker is agnostic to the underlying host.

The deployment config is also just a yml file.  deployment.yml tells Flocker where to deploy each container by listing one or more IP addresses and application aliases defined in the application.yml.

In this case, we are going to deploy each of our containers to a different VM.

```
"version": 1
"nodes":
  "172.16.255.250": ["elasticsearch"]
  "172.16.255.251": ["logstash"]
  "172.16.255.252": ["kibana"]
```

When we run the command flocker-deploy using the CLI tool provided by Flocker, the containers will be automatically deployed, networked, and started up on the servers that were defined in the deployment configuration.

```
alice@mercury:~/flocker-tutorial$ flocker-deploy deployment.yml application.yml
alice@mercury:~/flocker-tutorial$
```

## Migrating ElasticSearch and its data from one server to another

Now that ElasticSearch has been deployed to multiple nodes in a cluster, life is good.  Oh wait, one of your ElasticSearch queries is consuming 90% of available RAM on your m3.large EC2 instance and after a few minutes looking you can’t figure out why.  You can’t really afford sluggish performance of your ES cluster till you figure out the root cause of the poor performance so you want to move ElasticSearch to a bigger box, a m3.xlarge with 15 GB of RAM.

With Flocker, this is easy.  Just update your deployment.yml with the IP address of your new, beefy box and re-run flocker-deploy.  Your ElasticSearch container and its data volume will be automatically moved to the new node and connections that would have formally gone to the original node will be automatically routed to the new one.

### OLD:

```
"version": 1
"nodes":
  "172.16.255.250": ["elasticsearch"]
  "172.16.255.251": ["logstash"]
  "172.16.255.252": ["kibana"]
```

### NEW:

```
"version": 1
"nodes":
  "172.16.255.250": []
  "172.16.255.251": ["logstash"]
  "172.16.255.252": ["kibana"]
  "172.16.255.253": ["elasticsearch"]
```

Here’s what happens when you re-run flocker-deploy to migrate ElasticSearch from node1 to node2:

- Flocker checks to see if you’ve changed your deployment config
- Since it sees that you’ve moved ElasticSearch from 172.16.255.250 to 172.16.255.253 , it initiates a migration.
- The migration starts by pushing the entire contents of the the data volume of node1 to node2.  During this time, node1 is still accepting connections so your users and/or other processes dependent on that data don’t experience any connectivity issues.
- Once all the data have been copied over, only then is the application running on node1 shut down.
- Any changes to the data volume after the data was copied over will then be replicated.  Depending on how busy your database it, this may just be a few hundred kilobytes of changes.
- Once these last few changes have been copied over, Flocker hands off ownership of the volume to node2
- ElasticSearch starts up on node2

We call this operation two-phase push, because the data is migrated in two phases.  During phase one, the longest phase, while the data volume is being copied over, the database continues to serve connections.  It is only during phase two when the application experiences downtime. Importantly, removing this downtime completely is one of the goals of the Flocker project.  We are actively working towards a world when applications running in containers and their data can be seamlessly moved around between machines, even whole data centers with the same ease possible in a VM-based world.

Till then, we hope that this brief overview of deploying and migrating ELK has been useful.  For more information, [follow along with our step-by-step Getting Started guide](https://docs.clusterhq.com/en/0.2.1/gettingstarted/index.html) for installing and using Flocker.  In addition to ElasticSearch, we have examples for deploying and managing [MongoDB](https://docs.clusterhq.com/en/0.2.1/gettingstarted/tutorial/index.html), [PostgreSQL](https://docs.clusterhq.com/en/0.2.1/gettingstarted/examples/postgres.html) and [MySQL](https://docs.clusterhq.com/en/0.2.1/gettingstarted/examples/environment.html).

---

Original source: [Deploying and migrating a multi-node ElasticSearch-Logstash-Kibana cluster using Docker](https://clusterhq.com/blog/deploying-multi-node-elasticsearch-logstash-kibana-cluster-using-docker/)