# THOUGHTS AT THE START OF DOCKERCON EU

---

##### Author: Ben Golub, December 4, 2014

---

![alt](http://resource.docker.cn/nemo-science-center.jpg)
 
*The DockerCon EU Venue: NEMO in Amsterdam*

 

As I write this blog post, we are just a few hours from the start of [DockerCon EU](http://europe.dockercon.com/). So, it feels like a good time to reflect on both where Docker has come from in the ~19 months since Solomon and team started the project and where we are going in the future. 

## WHERE WE’VE COME FROM

The progress over the past 19 months (and especially in the five months since we launched Docker 1.0 at [DockerCon 14](http://www.dockercon.com/) in San Francisco) has been incredible. Docker is truly blessed to have such an amazing community and, through the combined efforts of users, contributors, and vendors,  Docker has transformed from a great idea to a great technology to a great ecosystem to, frankly, an amazing movement. This movement  will help usher in a new era in the way [distributed applications](http://blog.docker.com/2014/12/announcing-docker-machine-swarm-and-compose-for-orchestrating-distributed-apps/) are built, shipped, and run.

One way to look at the progress is with some key numbers. As you can see, the change in the last six months, across a whole variety of measures, has been really spectacular.


![alt](http://resource.docker.cn/metrics-graphic-3.jpg)


But, the numbers really only show part of the progress. Here’s a snapshot of how the project has grown:

![alt](http://resource.docker.cn/changes41.jpg)

* DGAB: Docker Governance Advisory Board

 

### Platform

A year ago, Docker was a great way to package up any Linux app into a single container and run it on any server. However, “any server” pretty much meant any server running a modern version of Ubuntu. By June of this year, we had expanded to all the major Linux distros, OpenStack, major PaaS platforms, and some significant public clouds. Now, we’ve got major integrations in all the major virtualization platforms ([VMWare](http://blog.docker.com/2014/08/docker-vmware-1-1-3/)) and have seen Docker services launched at Softlayer, Rackspace, [AWS](http://blog.docker.com/2014/11/docker-based-multi-container-applications-run-on-aws-cloud-with-introduction-of-the-ec2-container-service/), [Azure](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/), and [Google Compute](http://googlecloudplatform.blogspot.nl/2014/11/unleashing-containers-and-kubernetes-with-google-compute-engine.html).

### Users

A year ago, we were thrilled by the activity that was happening with the really early adopters and the small web companies. By DockerCon in June, big web companies (e.g., Gilt, Groupon, EBay , Baidu, and Google) were talking about their use of Docker in production. After shipping Docker 1.0 in June, we have seen a profound uptick in the use of Docker in traditional enterprises. Virtually every major bank has a Docker program or pilot underway, and there is adoption across media, manufacturing, health-care, and even government. We are pleased to have folks like ING, the Joint Genome Institute, Société Générale, and BBC talking about their use of Docker—and their vision of distributed applications—at DockerCon EU.

 

### Governance

When we started Docker, we knew that we wanted to be radically open, embracing the most permissive license (Apache), and actively encouraging broad contribution. By June 14, it was clear that we were on to something, with over 95% of contributors (and roughly half of all PRs), coming from outside Docker, Inc. Community members are core maintainers (e.g.,[Tianon](https://github.com/tianon)), and we have implemented a fully open design process. But, it also became clear that we needed to scale up to become even more transparent and effective, balancing open governance with effective governance. You can learn more [here](http://blog.docker.com/2014/11/docker-governance-advisory-board-output-of-first-meeting/), but we are happy to have a functioning Governance  Advisory Board, proactive reporting on metrics, escalation procedures, and a new team at Docker (team Meta) that is fire-walled from the commercial side of the house, and dedicated solely to making Docker awesome for contributors and community alike.

 

### Functionality

When we started Docker, our mission was to build “the button” that would enable any application to build, ship, and run on any server, anywhere. We started out focusing on single container applications, and that’s still where the bulk of our effort lies. But, as people looked to build out more complex, multi-container distributed applications in more complicated, multi-organization and multi-infrastructure environments, it was clear that our functionality needed to evolve. At DockerCon in June, we unveiled Docker Hub. And, at this DockerCon, we will be making some significant announcements around both open source orchestration and Docker, Inc. commercial management tools.

 

## SO, WHAT ARE DISTRIBUTED APPS ANYWAY?

Scott Johnston has written an excellent [blog post on distributed applications](http://blog.docker.com/2014/12/announcing-docker-machine-swarm-and-compose-for-orchestrating-distributed-apps/), but to quickly summarize:

We are entering an era of distributed applications. And, whether you call them distributed apps, cloud native apps, 12factor apps, or microservices, these applications represent the next generation of application architectures, approaches, and processes.

![alt](http://resource.docker.cn/distrograph.jpg)

Docker believes the best way to construct a powerful, robust distributed applications is to have each of its components/microservices/etc. built as an interoperable and standard container. We also believe it is critical to have a standard—yet flexible and open— set of orchestration capabilities that let the entire application behave as a logical whole, even as the individual Docker containers are run across different servers, clusters, and, ultimately, even data centers.

In keeping with Docker’s overall philosophy on being open and layered, we will ship these new capabilities with a “batteries included, but removable” philosophy. If you want to use Docker as a single container format, that’s cool with us. We continue to invest hugely in this area (we’ve more than tripled the Docker, Inc. folk working on this, and the community contributions have similarly grown). On the other hand, If you want to use our APIs in conjunction with orchestration tools like Mesos, Kubernetes, Clocker, and others—great. And, if you want a fully integrated, end-to-end solution from us, we’ll ship Docker with batteries included, ready to run.

 

## WHERE WE’RE GOING

![alt](http://resource.docker.cn/5-steps.jpg)

To some extent, you can think about all this as a five-step journey. Docker was fortunate to build upon the pioneering work of the folk who worked on things like zones, cGroups, namespaces, etc. The first fifteen months of our life were about taking the low level container standards and making them easy to use and portable, as well as building an amazing ecosystem of users, partners, content, and contributors.  The next year will be about extending that to make complex, multi-container, distributed applications a reality, with both open source orchestration capabilities and some exciting commercial products from Docker, Inc. and our partners.

Docker’s priorities for the next year, therefore,  are pretty clear:

1. Keep the entire ecosystem strong, open, healthy, and growing
2. Build the foundations for distributed applications the right way
3. Prove that this new model provides both open and effective governance
4. Make sure that Docker is truly production worthy
5. As a company, make sure we have a revenue model that supports our enormous investment in (and responsibility to) the community

## THANKS

Finally, I can’t conclude without some thanks to those who have gotten us here. Heartfelt thanks to the Amsterdam Docker community for organizing DockerCon EU, to the over 700 contributors to Docker,  and to the amazing ecosystem that has written over 18k  Docker tools and 60k Dockerized languages, frameworks, and apps. Thanks to our partners, thanks to the awesome Docker, Inc. team, and to the huge community of people out doing meetups and evangelizing Docker. And, of course, thanks to [Solomon Hykes](https://github.com/shykes), without his leadership and tireless efforts, none of this would be possible.

---

Original source: [THOUGHTS AT THE START OF DOCKERCON EU](https://blog.docker.com/2014/12/thoughts-at-the-start-of-dockercon-eu/)