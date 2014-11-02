# Docker Community Updates from Tech Ed and Docker Global Hack Day

---

##### Author: Ross Gardler

---

![alt](http://resource.docker.cn/docker-and-azure-ecosystem.png)

As MS Open Technologies, Inc. recently [announced](http://msopentech.com/blog/2014/10/15/docker-containers-coming-microsoft-linux-server-near/) our intention to build upon an existing [engagement](http://msopentech.com/blog/2014/06/09/docker-on-microsoft-azure/) with the Docker community, Microsoft Corp. selected the TechEd Europe conference in Barcelona this week to demonstrate some of our work on the Docker Client tools. In Mark Russinovich’s section of the opening keynote, he showcased a prototype of the forthcoming Windows Docker client tools (at present the tools are only available on Linux). In addition, Microsoft used this forum to announce CoreOS as the latest Linux distribution endorsed for Microsoft Azure (Mark’s Docker demo used a CoreOS VM on Azure as the Docker host).

## Windows Docker Client Prototype

You can watch [video of the Tech Ed demo](http://www.microsoft.com/eu/whats-next/Live_from_TechEd_Europe.aspx) (the Docker section starts at 1:19:45). In it, you will see:

Mark first shows how he previously created a CoreOS virtual machine on Microsoft Azure using a proposed “docker host” command. Note this command, currently in an early stage of implementation (see below), will significantly improve on the [current solution](http://msopentech.com/blog/2014/09/11/docker_host_in_azure/), which requires the installation of Azure cross-platform CLI tools. By bringing this functionality into the Docker CLI, it will become even easier to manage Docker containers and hosts with the standard Docker tooling.

Then Mark deploys a container with WordPress + Ubuntu to this host. He uses the standard “docker run” command which Docker users are already familiar with. Seconds later, the CoreOS host has a containerized instance of WordPress up and running, and Mark demonstrates that it can be accessed within his browser.

There are two truly significant things to note in this demo:

First, the whole demo (which includes a number of explanatory steps not detailed above) is just seconds over three minutes in length. Think about that for a moment. Having created a CoreOS VM running on Azure you can fire up a container with a fully configured instance of WordPress in just seconds. Think of the advantages this would bring to your own applications.

The second is that this demo used a prototype of the Docker client running on Windows. Today, the client only works on Linux. By bringing the client to Windows, we are making it possible to select your operating system of choice to manage Docker containers. This priority will carry forward as we bring the [Docker Engine to Windows Server](http://msopentech.com/blog/2014/10/15/docker-containers-coming-microsoft-linux-server-near/) in the future, thus allowing applications to be built and deployed as Linux containers or Windows containers - or an optimized mix of both.

### Docker Community Evaluation of Docker Hosts Proposal

The Docker community recently held their second [Global Hack Day](https://blog.docker.com/2014/10/announcing-docker-global-hack-day-2/) with activities in more than 30 cities around the world. One component of this event allowed community members to describe and discuss new proposals for the Docker project. Nathan LeClaire from Docker Inc. proposed the Docker Host feature - the very same feature demoed at TechEd Europe.

Nathan outlined why the Docker host feature is important and described how work done by the community implements his proposed solution, including the Azure implementation discussed above. It was great to see Nathan using our code as an example of how it might work in practice, and we look forward to improving upon this proposed functionality.

You can view Nathan’s demonstration and subsequent review of the proposal in this [video](https://docker.com/community/globalhackday) on the Docker site (the section starts at 16:00). The proposal is being [discussed on GitHub](https://github.com/docker/docker/issues/8681) with the Git branch for the implementation being maintained by [Ben Firshman](https://github.com/bfirsh/docker/tree/host-management), also of Docker Inc.

### Docker Global Hack Day Inspires Azure Storage Driver for Docker

One additional inflection point on container news: MS Open Tech engineering wizard Jeff Mendoza and the equally impressive Ahmet Alp Balkan from the Microsoft Azure team worked collaboratively to enable Docker users to host private Image Repositories on Azure, as part of the Docker Global Hack Day. As with all our work, our focus is on promoting choice by enabling interoperability. For the Seattle event, our team focused creating tools that increase the available choices for Docker users who wish to host their own private registries.

The team implemented a feature that allows private Docker registries to be stored in Azure blob storage. This work is now [merged](https://github.com/docker/docker-registry/pull/664) into the Docker codebase and can be used today (if you are brave enough to use unreleased code). Given that the code has already been merged it is likely that the project maintainers may elect to include it in a future release.

## Why Are Containers so Important?

Containers are an innovative concept, and it appears that people are looking to wrap their heads around how containers apply to them. A colleague of mine over in the Microsoft Azure team, Madhan Arumugam Ramakrishnan, and I felt it might be useful to examine this further. The video below shows some of our discussion and white-boarding on the topic:

Click to watch the [video](https://sec.ch9.ms/ch9/59b1/b4bcbd83-8dab-4c6b-9d6a-ffc78ad859b1/DockerHighLevelRossMadhan_mid.mp4)

## A Great Start

We started our engagement with Docker by ensuring users could run Linux containers on Azure using the tooling they wanted. We made this initial support [available](http://msopentech.com/blog/2014/06/09/docker-on-microsoft-azure/) the same day Docker issued its 1.0 release. With subsequent  [announcements](http://msopentech.com/blog/2014/10/15/docker-containers-coming-microsoft-linux-server-near/), we have committed to cultivating an even closer relationship with Docker and its community. The work described here is the first of many small steps you can expect to see as we deliver on these commitments.

---

Original source: [Docker Community Updates from Tech Ed and Docker Global Hack Day](https://msopentech.com/blog/2014/10/31/docker_community_updates/)