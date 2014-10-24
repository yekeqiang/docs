# SDN and Dockerfiles for Clocker, the Docker Cloud Maker

---

##### Author: Andrew Donald Kennedy

---

> **Editor’s Note**: [Cloudsoft](http://cloudsoft.io/), a sponsor of The New Stack, posted an [update](http://www.cloudsoftcorp.com/blog/2014/10/clocker-update-adding-sdn-support-weave-new-goodies/) to the [Clocker project](http://brooklyncentral.github.io/clocker/) on their blog yesterday. It discusses new support for Dockerfiles and [Weave](http://www.infoq.com/news/2014/09/zettio_releases_weave), a new software defined networking capability and open source project led by the group at [Zettio](http://www.zett.io/).  The New Stack will take a closer look at Weave later this month. For background, Clocker lets you create one or more Docker clouds,  underpinned by a dynamic cluster of Docker hosts deployed on your preferred infrastructure. Instead of VMs, Clocker serves up containers on demand. Orchestration is with [Brooklyn](https://brooklyn.incubator.apache.org/), an Apache incubator project, which serves as the central nervous system, monitoring and managing the whole Clocker stack including any apps that are deployed in it using Brooklyn blueprints. Clocker is in many respects a way to bootstrap Docker clouds across any number of platforms that are managed with Brooklyn.

This post describes some of the new features available in both the stable and pre-release versions, as well as the current roadmap for future versions of Apache licensed open source project Clocker.

## Introduction

The latest pre-release version of [Clocker](http://clocker.io/) is now available, as version [0.7.0-SNAPSHOT](https://github.com/brooklyncentral/clocker), and as a stable release version [0.6.2](http://github.com/brooklyncentral/clocker/releases/v0.6.2/). Clocker is an [Apache Brooklyn](http://brooklyn.io/) application that simplifies deployment, management and orchestration of application blueprints on [Docker](http://docker.io/) in the Cloud. To read an introduction to Clocker, see the [Clocker – Creating a Docker Cloud with Apache Brooklyn](http://www.cloudsoftcorp.com/blog/2014/06/clocker-creating-a-docker-cloud-with-apache-brooklyn/) article on the Cloudsoft blog.


## New Features

Version 0.7.0 builds on the previous stable 0.6.2 release, which contained many bugfixes and improvements based on feedback from users, as well as incorporating a few new features such as an affinity strategy and better control of the underlying Docker Engine and containers. In particular 0.6.2 made full use of the local Docker image repository, in order to take advantage of the fast start-up time that is one of the large benefits of container technology over virtual machines. The useful thing about Clocker is that this feature is available automatically, for any Brooklyn entity, rather than having to go to the effort of creating new images by hand for each service or application component.

## Autogenerating Docker Images

Clocker creates the image in the background while it is deploying an entity. The normal process in Brooklyn entities is to install them from a .tgz archive that is downloaded, or run some APT or Yum commands to set up the environment, which is common to all instances of that type of entity, then customise that for a particular location and launch the service. With Clocker, after the install phase we execute the docker commit command and save the image data for re-use. This means containers for the second and further entities of dome type on a particular Docker host will use that image and customise it, saving the network traffic and install time penalty. At the moment this is restricted to being shared within a host only, but Clocker 0.8.x will also include the ability to docker push an image out to a remote repository, as well asdocker pull from the same or via jFrog’s [Artifactory](http://www.jfrog.com/confluence/display/RTF/Docker+Repositories), [Docker Hub](http://hub.docker.com/), [quay.io](http://quay.io/) and others. If you have an image pre-configured (such as an AMI) with a set of Docker images, you could also use their IDs when starting an entity, to similar effect.

## Dockerfile Support

Clocker 0.7.0 also allows the deployment of applications defined solely by Dockerfiles, without requiring the use of specific Brooklyn entities which may not be available. This is achieved by packaging the Dockerfile and associated resources into an archive that can be deployed to a Docker host and used to build an image. Usefully, this archive can be automatically generated from the files found in a checkout from the [Docker Registry](http://github.com/docker-library/) on GitHub. Samples of application blueprints using this technique will be available in the 0.7.0 release.

## SDN Support

Probably the most interesting new feature that is present in 0.7.0 is the addition of software defined networking or [SDN](https://en.wikipedia.org/wiki/Software-defined_networking) capabilities. We are using the [Weave](http://github.com/zettio/weave/) open source project from the guys at [Zettio](http://zettio.com/) which is explicitly designed to work with Docker.

> ##### Clocker keeps track of each container deployed and attaches them to the Weave network, controlling the allocation of IP addresses and configuring the deployed entities so that they can use this information.

Since Weave is acting as a software-based network switch, connecting all Docker hosts via TCP and UDP userspace networking, the benefits of SDN are available even in cloud VMs where minimal control over networking is available. Weave allows the services on containers in different hosts to communicate seamlessly, even if there are intermediate proxying entities like the Erlang portmapper daemon, which is difficult to port-forward through Docker normally. Because Clocker is responsible for assigning the Weave network IP addresses, it can ensure that a container’s address stays the same, even after rebooting or moving to a different host, making applications much mroe resilient in the face of failures. Clocker also uses the [DNSmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) service to provide name resolution in a private namespace that makes access to servers through Docker’s port forwarding more effective.

## Policy Based Management

The Clocker environment also makes more use of the underlying Brooklyn platform’s capabilities, and 0.7.0 will include policies to handle restarting failed containers and their associated services, as well as the ability to scale the Docker host cluster to provide a continual amount of headroom for new containers. At present, in version 0.6.2 when a new Docker container requires a new Docker host to be created, the deployment must block until a suitable virtual machine can be provisioned by the underlying cloud, a process that takes minutes rather than seconds.

## Brooklyn YAML Support

Another new mechanism provided by Brooklyn is the ability to specify and configure arbitrary objects in the CAMP blueprint YAML file. This is described in another [blog post](http://blog.abstractvisitorpattern.co.uk/2014/10/new-brooklyn-blueprint%20features.html), and is the basis of the provisioning and placement strategy configuration used in 0.7.0. Previous versions of Clocker permitted only single placement strategies to be defined, either per entity or on the infrastructure as a whole.

The use of multiple strategies allows a much more nuanced approach to be taken for the supply-side of the container provisioning equation. As an example, the infrastructure can be configured with strategies that define the maximum containers per host, and prevent deployment to hosts with too high a CPU load. Individual entities can then augment this strategy by specifying their CPU core and RAM requirements or affinity and geographic constraints. Placement strategies are joined by provisioning strategies, which modify the flags used to create the virtual machines the Docker Engine runs on, when no appropriate host can be found for a container.

## Clocker Evolution

The next release of Clocker will extend the Docker Hub and repository integration to allow access to private repositories, which will be provided by extensions to the existing jclouds-labs driver, rather that the CLI. The jclouds driver is an essential part of Clocker, allowing containers to be exposed to Brooklyn in an identical way to virtual machines.

Another feature that is being developed in conjunction with improvements in the core Brooklyn software is the ability to self-host a Clocker instance. This will enable starting a Clocker instance that uses an already provisioned Docker Engine running on localhost (or in the cloud) and optionally running the Brooklyn control-plane itself inside a new container on that Docker instance. One of the many requests we get from users is to make the initial experience easier, and we hope this will be a step in the right direction.

![alt](http://resource.docker.cn/alice-in-wonderland-clock.jpg)

In a [presentation](http://speakerdeck.com/grkvlt/clocker-evolution) I gave at a recent Silicon Valley DevOps meet-up, I asked several questions which are relevant to the future development of Clocker. Weave integration is a big step, but it is not the only SDN solution for Docker. Fig and Kubernetes show that Clocker is not alone in the orchestration for containers space, and there is probably room for more services fulfilling various niches here. However, as with Dockerfiles being used to define a single container, a standard for describing a multi-host set of containers would be a useful thing. For Clocker and Brooklyn, the answer is [Oasis CAMP](https://www.oasis-open.org/committees/camp/) but Google may have other ideas.

Finally, the users of software are not the same people as the engineers who design and build it, and there will always be missing features in both Clocker and Docker that would improve the experience, so help us out by telling us what they are:

- Where is Docker networking headed?
- What is the future of Docker orchestration?
- Are there any other missing Docker features that would improve usability?

Keep those ideas coming and reach out to me on [@grkvlt](http://twitter.com/grkvlt).

## Learn More

More detailed documentation and further information on Brooklyn can be found at the main [Apache Brooklyn](http://brooklyn.incubator.apache.org/) site. Clocker is hosted on GitHub at [brooklyncentral/clocker](http://github.com/brooklyncentral/clocker/) and there is a quick start guide at [clocker.io](http://clocker.io/). You can provide feedback with any comments or problems on the [incubator-brooklyn-dev](https://mail-archives.apache.org/mod_mbox/incubator-brooklyn-dev/) mailing list, or add [issues](http://github.com/brooklyncentral/clocker/issues/) to the Clocker GitHub repository. If you have an idea for an improvement or a new feature, just [fork](http://github.com/brooklyncentral/clocker/fork) the code and issue a pull request!

*Andrew is a Senior Software Engineer at Cloudsoft and a contributor to Open Source projects including jclouds and Apache Qpid and is a founder member of the Apache Brooklyn project. Areas of interest include distributed systems, virtualization, messaging, information security and LOLcats. Prior to joining Cloudsoft, Andrew worked for various investment banks as a software engineer and security consultant and has over twenty years experience in the IT industry.*

---

Original source: [SDN and Dockerfiles for Clocker, the Docker Cloud Maker](http://thenewstack.io/sdn-and-dockerfiles-for-clocker-the-docker-cloud-maker/)

