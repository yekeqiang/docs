# MODULUS AND DOCKER

---

Author: Brandon Cannaday

---

![alt](http://resource.docker.cn/modulus-docker.png)

A couple of weeks ago we briefly talked about how [Docker has changed](http://blog.modulus.io/docker-and-the-future-of-paas) the way we deploy our internal software for both the public cloud and enterprise installs. Today we’re excited to announce that we’ve also implemented Docker as the default containerization technology for all customer applications.

In this article I’m going to provide a peek into how things work at Modulus and how we’ve made Docker work in a PaaS environment. What makes PaaS such a unique use-case for [Docker](https://www.docker.com/) is its dynamic nature. We manage thousands of applications that are constantly scaling, deploying, and moving around within our various infrastructure providers and regions. This dynamic nature leads to many architectural and development challenges that we’ve overcome to deliver a robust, high performance, and fault-tolerant system for Docker container management.

## The High-Level

Modulus is made up of many microservices all communicating to each other via queues and JSON APIs. The provisioning system specifically is made up of three main pieces, which we’ve internally named Congress, Medusa, and Mal. Congress is our main API service, Medusa contains the provisioning intelligence, and Mal runs on every physical server.

![alt](http://resource.docker.cn/modulus-high-level.png)

The separation of concerns is very important when building an easy-to-maintain system. Each service that makes up Modulus can be revved independently, and as long as the APIs remain unchanged, the rest of the system is unaffected.

## The Low-Level

Let’s dig in a little and see what happens during a provision. Provisioning occurs when a project requires a new servo. This happens when scaling up or deploying a newly created project.

### Medusa

Each application server maintains their own state, which includes things like what projects are running, how much memory is available to them, and how much capacity is remaining. When Medusa needs a new servo for a project, the first thing it does is request the state from every Mal in the entire system. Doing this has one major advantage - if a Mal doesn’t respond to a state request it gets removed from eligibility for this provisioning round. This allows us to be fault tolerant to minor network blips, failed servers, or entire region outages.

Once Medusa has the system-wide state information it then runs through our intelligent provisioning logic to properly place the new servo on the best possible physical application server.

## Mal

Once Medusa has determined where a servo should be located it then makes a request to Mal to create it. Mal creates and destroys Docker containers on-demand when applications are provisioned and deprovisioned. If Mal fails to provision a Docker container, which can happen for a lot of different reasons, an error will be returned to Medusa, which then invokes a re-provision with this particular application host excluded.

The first thing Mal does when provisioning is to create the local data volume to hold your application data. It then creates and runs our Docker image with the proper memory and CPU limits as well as exposing the correct ports for our load balancer. With Docker’s ability to launch quickly, this is done within a matter of seconds.

![alt](http://resource.docker.cn/modulus-low-level.png)

We chose to create data volumes externally to each servo because we didn’t want to lose application data on server reboots. By default any changes made to a Docker image that aren’t committed are lost when the container shuts down. For all data except customer application data, this is the behavior we want. On a reboot, Mal simply restarts every container and the customer’s data is automatically remounted and the project restarted.

If you’ve ever used public infrastructure providers, you know how important it is to be reboot-tolerant. This is also why Medusa contains intelligence on where to place servos. We try very hard to make sure servos are placed on different physical servers. This way if one is not responding, the load balancer will automatically redirect traffic to another.

## Security, Performance, and Stability

Docker has a few known vulnerabilities that we take all precautions to avoid. The most well known is the ability to break out of containers when you have root privileges. Since we completely control the image that customer apps run under, we have a great deal of control over the environment in which they run. All customer apps run under a separate user account that has very explicitly defined access rights to the files, folders, and systems within the container.

Compared to our previous containerization technology, based on LXC, Docker has an improved networking stack for greater throughput and lower latency. We’ve also improved our underlying disk performance by changing how we create and mount volumes to each container.

The biggest gains come in the form of greatly improved recovery mechanisms when initially deploying and scaling applications. We’ve also drastically changed the way scaling works to more properly support massive outages at our infrastructure providers. If an entire provider or region goes down, you can now seamlessly scale your application into another region and continuing running with very little interruption.

## How do I switch my app to Docker?

Right now we run both containerization technologies together. Existing applications haven’t changed and are still on LXC. Any operations made to those projects will remain on LXC. We silently launched the new technology a couple of weeks ago – surprise! All new projects are currently being deployed to the new system.

If your project is more than a couple of weeks old and you’d like to switch to our latest and greatest, all you have to do is completely stop your project and start it again. When you start a stopped project it invokes a provision, which is described above. Since you have no existing servos using LXC, the provision will take place on the new Docker infrastructure.

## The Future

We’ve very carefully architected our solution to prepare us for some very exciting and powerful features coming to the Modulus platform. I can’t go into too many details right now, but what we're building is by far some of the most interesting and cutting edge deployment technology we've ever created.

---

Original source: [MODULUS AND DOCKER](http://blog.modulus.io/modulus-and-docker)