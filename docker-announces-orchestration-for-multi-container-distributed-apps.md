# DOCKER ANNOUNCES ORCHESTRATION FOR MULTI-CONTAINER DISTRIBUTED APPS

---

Author: David Messina, December 4, 2014

---

*New services accelerate development-to-deployment of applications atop any infrastructure, from on-premise virtualized data centers to public clouds*

**San Francisco, Calif.  and Amsterdam, NL.– December 4, 2014** – [Docker, Inc.](www.docker.com), the company behind the open platform for distributed applications, today announced new, comprehensive platform services for orchestrating multi-container distributed applications. These comprehensive orchestration capabilities are designed to empower developers and sysadmins to create and manage a new generation of portable distributed applications that are rapidly composed of discrete interoperable Docker containers, have a dynamic lifecycle, and can scale to run in concert anywhere from the developer’s laptop to hundreds of hosts in the cloud.

“As we evolve from applications created from a small number of Docker containers on a handful of hosts to large, multi-Docker container applications spread across clusters and diverse infrastructures, it is important that users don’t lose the qualities that have made Docker so successful,” said Solomon Hykes, CTO and founder of Docker and the Chief Maintainer of the Docker open source project. “This includes native and open interfaces, the ability to be portable across all environments, and through a common UI the power to leverage a broad ecosystem of 18,000 tools and 60,000 Dockerized apps.”

Docker orchestration services are being previewed at [DockerCon EU](http://europe.dockercon.com/) – the inaugural Docker user conference in Europe – and are backed by an unrivaled partner ecosystem which includes: Cisco, Digital Ocean, HP, IBM, Mesosphere, Microsoft and VMware.

## Docker Platform Enhancements for Multi-Container Distributed Applications

Docker’s orchestration capabilities build on the foundations of the open platform which established Docker containers as the industry standard for building discrete application services into interoperable and iterative components that can run anywhere. Docker’s orchestration services are a reflection of the transformational shift that is occurring as organizations migrate from slow-evolving monolithic applications to Dockerized distributed applications. Orchestrating these distributed applications requires a native multi-container, multi-host approach with a common UI and tooling that works across infrastructures.

## Comprehensive Orchestration Provides a Powerful New Approach to Application Development and Operations

Docker’s orchestration capabilities are delivered through three new platform services that are designed to cover all aspects of the dynamic lifecycle of distributed applications, where new code or a new Dockerized service that changes application functionality can be put into production in minutes instead of months. While Docker’s orchestration services are the most comprehensive on the market, their unique modular structure allows them to be accessible for both users – developers and sysadmins – and ecosystem partners.  For example, there is an explicit service focused on developers to help facilitate their creation of a distributed application stack, while another service is focused on clustering that speaks to the concerns of the operations team.

## The three new orchestration services are:

- **Docker Machine**: This service further expands the portability capabilities of distributed applications by providing the user the flexibility to provision any host with the Docker Engine, whether a laptop, a data center VM, or a cloud node. This saves a developer a significant amount of time in manual setup and custom scripting, resulting in faster iterations and compressing the development-to-deployment cycle.
- **Docker Swarm**: Docker Swarm is a Docker-native clustering service that works with the Docker Engines, provisioned by the new Docker Machine service, and creates a resource pool of the hosts on which the distributed applications run. By automatically scheduling container workloads and allocating resources, Docker Swarm provides users with high-performance and availability while eliminating inefficient and error-prone manual resource management. Docker Swarm is unique in the industry as it is expressly designed for a continuous lifecycle from development to operations. It is structured in a way that a developer can test his clustered application across a few on-premise machines, while an operations team can use the same tooling to scale that same application across hundreds of hosts across diverse infrastructure. The Docker Swarm API supports pluggable clustering implementations, giving customers the ability to choose highly-scalable solutions like Mesosphere to orchestrate containers across tens thousands of nodes.
- **Docker Compose**: This service provides developers with the ability to assemble applications from discrete, interoperable Docker containers completely independent of any underlying infrastructure, enabling distributed application stacks to be deployed anywhere and moved at any time. Defining a distributed application stack and its dependencies through a simple YAML configuration file converts what was an incredibly complex process into a simple one that can be executed in just a few keystrokes. This powerful capability means that new distributed applications can be spun up in minutes in stark contrast to conventional approaches where an application’s time-to-market was measured in months if not years.

## Open APIs and Open Design Create Opportunity for Broad Ecosystem

Open APIs for Docker’s orchestration services expand collaboration opportunities for Docker’s vibrant ecosystem, which has grown into hundreds of partners over the 19 months of the open source project. Aspects of the orchestration APIs were previously made available to infrastructure partners to help showcase reference implementations as part of Docker’s open design process and are now being made available to the Docker partner ecosystem at DockerCon EU. This new set of APIs will also make it possible for standalone orchestration products to have their feature sets accessible through the Docker UI.

Docker’s open design process is reflected in the three orchestration services, which have been shaped by direct input from the community to cover all aspects of multi-container development to deployment in a continuous application lifecycle.

Furthermore, the community involvement has structured these services to be modular, which provides users of Docker the flexibility to choose which service fits their application’s requirements with the ability to leverage other orchestration tooling from the Docker ecosystem.

## “Batteries Included, but Removable.”  All Orchestration Services are Optional.

Consistent with Docker’s layered, API-driven, open approach, users will have options when using each of these new orchestration services:

1. “Batteries included”: The user would run the default implementation of the service Docker includes in the release.
2. “Batteries removable”: Instead of running the default Docker implementation, some users may elect to use a third-party’s implementation of clustering, scheduling, or any other service written to the orchestration APIs. This will help preserve portability and interoperability, while providing users with the flexibility to choose ecosystem technologies that best suit a particular use case.
3. “Single Container Only”: Even the use of multi-container orchestration APIs is optional. For organizations or vendors that only require a single Docker container approach, they can continue to use the Docker platform without changing the workflows they already have in place today. Even as the Docker project has expanded the set of capabilities around Docker, the community of developers working on single container functionality has tripled and the project will continue to invest resources on this use case.

## Availability

The three Docker orchestration services are now available as Alpha releases.  These new capabilities are being demonstrated at DockerCon EU. APIs are now available for Docker Machine with others being made available in 1H, 2015. These three orchestration services are targeted for general availability in Q2, 2015.

## Ecosystem Partner Quotes

#### *Docker Machine*

“Docker Machine allows us to deliver greater efficiency to our users who are leveraging Docker to build their distributed applications. By making the Docker Engine available natively in our cloud, we’re making it even simpler for developers to deploy their distributed applications.” – Ben Uretsky, CEO, DigitalOcean

“We have strategically partnered with Docker because Microsoft is investing in freedom of choice for developers. We believe that Docker’s orchestration services APIs offer a flexible way to build, deploy and manage highly available, distributed applications built for Linux containers —and when available—Windows Server containers that are freely portable between any host infrastructure. We are excited about the Docker Machine Management API support for Azure as part of the Alpha release.” – John Gossman, architect, Microsoft Azure

“Our focus has always been to provide production-grade infrastructure to run our customers’ mission-critical applications. As DevOps processes gain traction in the enterprise, Docker Machine offer a seamless and DevOps friendly way to help our customers to rapidly migrate applications from a developer laptop running VMware Fusion to their on-premises virtual environments or VMware vCloud Air.” – Kit Colbert, vice president and CTO, Cloud Native Apps, VMware

#### *Docker Swarm*

“The Open APIs for Docker let the community use Docker in combination with Mesosphere to orchestrate containers on our highly-scalable platform.”– Florian Leibert, CEO and co-founder, Mesosphere

#### *Docker Compose*

“As strong proponents of open source, open APIs and heterogeneous computing environments, Cisco is excited by the market transition towards container technology. With new tools like Docker Compose, Docker has an opportunity to capture distributed application policies which we can ultimately extend to the network as well.” – Mike Cohen, director of product management, Cisco

“Docker will play a key role for how our enterprise customers build and run distributed, cloud native applications. HP will collaborate closely with the Docker community on the new Docker Compose capability to provide integrated value with both its enterprise level product, HP Operations Orchestration, as well as with the new open source project, Eclipse Score. Given the HP Helion Development Platform contains built-in Docker technology already, this is a natural and next logical extension of our relationship.” – Jerome Labat, CTO, HP Software

“In collaboration with the open community, Docker is building orchestration services that will help developers rapidly compose distributed applications from containers. IBM is also able to bring these enhancements to enterprise developers as standards-based solutions on the IBM Cloud.” – Angel Diaz, vice president of Open Technology and Cloud Performance Solutions, IBM

## About Docker, Inc.

Docker, Inc. is the company behind the Docker open source platform, and is the chief sponsor of the Docker ecosystem. Docker is an open platform for developers and system administrators to build, ship, run and orchestrate distributed applications. With Docker, IT organizations shrink application delivery from months to minutes, frictionlessly move workloads between data centers and the cloud, and improve infrastructure efficiency by 50 percent or more. Inspired by an active community and by transparent, open source innovation, Docker has been downloaded 67+ million times and is used by thousands of the world’s most innovative organizations, including eBay, Baidu, Yelp, Spotify, Yandex, and Cambridge HealthCare. Docker’s rapid adoption has catalyzed an active ecosystem, resulting in more than 60,000 Dockerized applications and integration partnerships with AWS, Cloud Foundry, Google, IBM, Microsoft, OpenStack, Rackspace, Red Hat and VMware.

Docker, Inc. is venture backed by [AME Cloud Ventures](http://www.amecloudventures.com/) (Yahoo! Founder Jerry Yang), [Benchmark](http://www.benchmark.com/) (Peter Fenton), [Greylock Partners](http://www.greylock.com/) (Jerry Chen), [Insight Venture Partners](http://www.greylock.com/) (Jerry Murdock), [Sequoia Capital](http://www.sequoiacap.com/) (Bill Coughran), [SV Angel](http://svangel.com/) (Ron Conway), [Trinity Ventures](http://www.trinityventures.com/) (Dan Scholnick), [Y Combinator](http://ycombinator.com/).

---

Original source: [DOCKER ANNOUNCES ORCHESTRATION FOR MULTI-CONTAINER DISTRIBUTED APPS](https://blog.docker.com/2014/12/docker-announces-orchestration-for-multi-container-distributed-apps/)