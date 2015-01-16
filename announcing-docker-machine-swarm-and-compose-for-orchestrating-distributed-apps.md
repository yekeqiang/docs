# ANNOUNCING DOCKER MACHINE, SWARM, AND COMPOSE FOR ORCHESTRATING DISTRIBUTED APPS

---

Author: Scott Johnston, December 4, 2014

---

As users start exploring Docker and Docker Hub, they typically start by [Dockerizing some apps](http://docs.docker.com/examples/nodejs_web_app/), incorporating Docker into their [build-test pipeline](http://developer-blog.cloudbees.com/2014/07/announcing-dockerhub-jenkins-plugin.html), creating a Docker-based [development environment](https://blog.relateiq.com/a-docker-dev-environment-in-24-hours-part-1-of-2/), or trying out one of the other [half-dozen common use cases](https://news.ycombinator.com/item?id=8470206).  In these use cases and others, we’ve heard from many community members about how Docker has sped up development-to-deploy cycle times and eliminated the friction of moving apps from laptops to data centers, or from one cloud to another.

![alt](http://resource.docker.cn/announcement.png)

## Da capo

After getting their feet wet with simple, single-container apps, we see many users start to employ Docker as a platform for more complex, distributed applications (a.k.a., [cloud native](http://www.slideshare.net/adrianco/netflix-what-changed-gartner-catalyst), [microservices architecture](http://martinfowler.com/articles/microservices.html), [twelve-factor apps](http://12factor.net/)), or for apps composed of multiple containers, typically running across multiple hosts.  Users’ motivations for exploring distributed apps vary, but some common themes are emerging. Users find them appealing because they are:

- **Portable**.  Application owners want an application stack that is independent of the underlying infrastructure so they’ll have the freedom to deploy on the infrastructure of their choosing – and then move or scale without friction as conditions change;
- **Composable**.  Application owners want apps composed of multiple smaller services so they can keep teams small and fast-moving as well as assemble apps from trusted, secure components;
- **Dynamic**.  Development-to-deployment cycles measured in months are no longer acceptable – application owners want cycle times measured in hours or even minutes.  Breaking down monolithic applications into multiple smaller services that can be updated independently reduces change management risk and compresses cycle time.
- **Scalable**.  To simultaneously meet changes in demand while efficiently using infrastructure resources, application owners want to frictionlessly scale-up and -down across hosts, data centers, and even public clouds.

However, as users in the Docker community take the early steps on their path to distributed apps, we often hear questions like these:

- “My singleton Docker containers are 100% portable to any infrastructure; but, how do I ensure my multi-container distributed app is also 100% portable, whether moving from staging to production, or across data centers, or between public clouds?”
- “My organization has standardized on Docker; how do I retain this standard while still taking advantage of the breadth and depth of the tools in the Docker ecosystem?”
- “How should I refactor my app delivery pipeline to support rapid iterations on the containers providing the services?”
- “Breaking up my monolithic apps into microservices means there are many more components to keep track of and manage; what’s available in the Docker ecosystem to help with this?”

To help users along their distributed apps journey, today at DockerCon Europe we’re excited to announce three new Docker orchestration services: **Docker Machine**, **Docker Swarm**, and **Docker Compose**.  Each one covers a different aspect of the lifecycle for distributed apps. Each one is implemented with a “batteries included, but removable” approach which, thanks to our orchestration APIs, means they may be swapped-out for alternative implementations from ecosystem partners designed for particular use cases.

![alt](http://resource.docker.cn/dockercon-blog-orchestra-conductor-300x203.png)

## Docker Machine

Docker Machine takes you from zero-to-Docker in seconds with a single command.

Before Docker Machine, a developer would need to log in to the host and follow installation and configuration instructions specifically for that host and its OS.  With Docker Machine, whether provisioning the Docker daemon on a new laptop, on virtual machines in the data center, or on a public cloud instance, the same, single command …

```
% machine create -d [infrastructure provider] [provider options] [machine name]
``

… gets the target host ready to run Docker containers.  Then, from the same interface, you can manage multiple Docker hosts regardless of their location and run any Docker command on them.

Furthermore, the pluggable backend of Docker Machine allows users to take full advantage of ecosystem partners providing Docker-ready infrastructure, while still accessing everything through the same interface.  This driver API works for provisioning Docker on a local machine, on a virtual machine in the data center, or on a public cloud instance.  In this Alpha release, Docker Machine ships with drivers for provisioning Docker locally with Virtualbox as well as remotely on Digital Ocean instances; more drivers are in the works for AWS, Azure, VMware, and other infrastructure.

Note that Docker Machine is a separate project from the Docker Engine.  To try out the Alpha build and contribute to Docker Machine and its drivers, go to [its repository](http://github.com/docker/machine).

![alt](http://resource.docker.cn/docker-whales-transparent-300x249.png)

## Docker Swarm

Docker Swarm is native clustering for Dockerized distributed apps. It picks-up where Docker Machines leaves off by optimizing host resource utilization and providing failover services.  Specifically, Docker Swarm allows users to create resource pools of hosts running Docker daemons and then schedule Docker containers to run on top, automatically managing workload placement and maintaining cluster state.

For input, the default scheduler uses the resource requirements of the Docker container workloads and the resource availabilities of the hosts in the cluster and then uses [bin pack](http://en.wikipedia.org/wiki/Bin_packing_problem) to automatically optimize placement of workloads.  For example, here is how a user would schedule a redis container requiring 1 gig of memory:

```
% docker run -d -P -m 1g redis
```

To support specific requirements and policy-based scheduling, Docker Swarm provides standard and custom constraints.  For example, say that to ensure good I/O performance you want to run your MySQL container on a host with SSD storage.  You could express this as a constraint when scheduling the MySQL workload as follows:

```
% docker run -d -P -e constraint:storage=ssd mysql
```

In addition to resource optimization, Docker Swarm provides high-availability and failover.  Docker Swarm continuously health-checks the Docker daemon’s hosts and, should one suffer an outage, automatically rebalances by moving and re-starting the Docker containers from the failed host to a new one.

One of the unique aspects of Docker Swarm is that it can scale with the lifecycle of the app.  This means that the developer can start with a “cluster” consisting of a single host and maintain a consistent interface as the app scales from one host to two, 20, or 200 hosts.

Finally, Docker Swarm has a pluggable architecture and ships “batteries included” with a default scheduler.  To this end, we’re excited to announce a partnership with Mesosphere to make it a “first class citizen” in Docker Swam for landing Docker container workloads.  Stay tuned for the public API in the first half of 2015 which will allow swapping-in a scheduler implemented by an ecosystem partner or even your own custom implementation.  Nevertheless, regardless of the underlying scheduler implementation, the interface to the app remains consistent, meaning that the app remains 100% portable.

The above just scratches the surface.  Like Docker Machine, Docker Swarm is a separate project from the Docker Engine.  If you want to learn more about Docker Swarm, including getting your hands on an Alpha build, head on over to [its repo](https://github.com/docker/swarm/).

## Docker Compose

Docker Compose is the last piece of the orchestration puzzle.  After provisioning Docker daemons on any host in any location with Docker Machine and clustering them with Docker Swarm, users can employ Docker Compose to assemble multi-container distributed apps that run on top of these clusters.

The first step to employing Docker Compose is to use a simple YAML file to declaratively define the desired state of the multi-container app:

```
containers:
  web:
     build: .
     command: python app.py
     ports:
     - "5000:5000"
     volumes:
     - .:/code
     links:
     - redis
     environment:
     - PYTHONUNBUFFERED=1
  redis:
     image: redis:latest
     command: redis-server --appendonly yes
```

This example shows how Docker Compose takes advantage of existing containers.  Specifically, in this simple two-container app declaration, the first container is a Python app built each time from the Dockerfile in the current directory.  The second container is built from the redis Official Repo on the Docker Hub Registry.  The links directive declares that the Python app container is dependent on the redis container.

Not that it’s defined, starting your app is as easy as …

```
% docker up
```

With this single command, the Python container is automatically built from its Dockerfile and the redis container is pulled from the Docker Hub Registry.  Then, thanks to the links directive expressing the dependency between the Python and redis containers, the redis container is started *first*, followed by the Python container.

Docker Compose is still a work-in-progress and we want your help to design it. In particular, we want to know whether or not you think this should be a part of the Docker binary or a separate tool. Head over to [the proposal on GitHub](https://github.com/docker/docker/issues/9459) to try out an alpha build and have your say.

## Coda

All this is just the briefest introduction to Docker Machine, Docker Swarm, and Docker Compose.  We hope you’ll take a moment to try them out and give us feedback – these projects are moving quickly and we welcome your input!

We also wish to thank the many community members who have contributed their experience, feedback, and pull requests during the pre-Alpha iterations of these projects.  It’s thanks to you that we were able to make so much progress so quickly, and in the right direction.

Distributed apps offer many benefits to users – portability, scalability, dynamic development-to-deployment acceleration – and we’re excited by the role the Docker platform, community, and ecosystem are playing in making these apps easier to build, ship, and run.  We’ve got a ways to go, but we’re psyched by this start – join us and help us get there faster!

Happy Hacking,

– The Docker Team

---

Learn More

- Machine [source code and Alpha build](http://github.com/docker/machine)
- Swarm [source code and Alpha build](http://github.com/docker/swarm/)
- Compose [proposal, source code and Alpha build](https://github.com/docker/docker/issues/9459)
- Content, workflows, and more – Sign-up for a free [Docker Hub account](http://hub.docker.com/)
- New to Docker?  Try our 10 min [online tutorial](http://docker.com/tryit/)

---

Original source: [ANNOUNCING DOCKER MACHINE, SWARM, AND COMPOSE FOR ORCHESTRATING DISTRIBUTED APPS](https://blog.docker.com/2014/12/announcing-docker-machine-swarm-and-compose-for-orchestrating-distributed-apps/)