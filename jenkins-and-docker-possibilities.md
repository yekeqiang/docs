# Jenkins and Docker - The Possibilities

---

##### Author: Udaypal Aarkoti

---

![alt](http://resource.docker.cn/jenkins-and-docker.png)

---

At a time when every millisecond delay has an [impact on revenue](https://news.ycombinator.com/item?id=273900), enterprises are constantly looking for ways to automate and optimize their applications and/or services to minimize the impact of a change while also making sure they provide a [steady stream](https://news.ycombinator.com/item?id=2971521) of feature enhancements to stay ahead of their competition. 

In that vein, enterprises, small and big alike, are looking for ways to fully automate their software delivery process. DevOps, continuous delivery, continuous integration, continuous deployment are all terms used to describe these efforts by people and organizations to achieve a higher degree of automation.

Significant number of these enterprises have already [adopted](http://zeroturnaround.com/rebellabs/10-kick-ass-technologies-modern-developers-love/6/) [Jenkins](http://jenkins-ci.org/) to achieve [continuous integration](http://en.wikipedia.org/wiki/Continuous_integration) (CI), a necessary step in achieving [continuous delivery](http://en.wikipedia.org/wiki/Continuous_delivery) (CD).

[Docker](https://www.docker.com/whatisdocker/), on the other hand, is a platform for building and running distributed applications and is revolutionizing the way applications are built and deployed in a complex multi-tiered environments. Docker, while still in its nascent stage, offers [several advantages](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.0_Release_Notes/sect-Red_Hat_Enterprise_Linux-7.0_Release_Notes-Linux_Containers_with_Docker_Format-Advantages_of_Using_Docker.html) over traditional ways of building and deploying applications.

While each of these technologies have their own benefits in software delivery process, together they can provide a more

- **Consistent** - The most important part of building a CD pipeline is how consistently one can build the pipeline so it is easy to create and maintain. Complexity in a pipeline results in unplanned work that is hard to automate and requires human intervention. A good CD pipeline is one that can self heal or provide accurate data so appropriate measures can be taken.
- **Replicable** - Replicate the implementation from one environment to another or from one project to another or one team to another. Lower barrier to entry for new environments, projects and teams to achieve CD.
- **Portable** - Jenkins and Docker are supported on most popular operating systems. Adding support for new OS's becomes a trivial task.
- **Cost effective** - Docker's optimal use of IT resources, compared to traditional virtualization technologies, and Jenkins plugins ecosystem (1000+ plugins integrating with most common tools required during SDLC needing minimal to no customizations) make them very cost effective.

ways to achieving continuous delivery.

So, how can you benefit from this integration between Jenkins and Docker? Here are some use cases of how you can leverage Jenkins and Docker in your continuous delivery process

1. **On Demand Slaves** [1] - Jenkins uses build slaves to execute all the jobs. In a typical environment these build slaves are static, i.e a fixed set of VM's or hardware is dedicated to slaves. With Docker, these slaves can be configured dynamically thus getting better utilization from the underlying hardware.
2. **Repeatable Deployments** [2] - Every code commit triggers a Jenkins CD pipeline that ends with a new Docker image in the docker repository and Docker containers being deployed in appropriate environments (QA, Staging, Production) as defined in the CD pipeline.
3. **Ephemeral resources** [3] - Some steps in CD pipeline require a clean environment, particularly test, and resources for the duration of the steps. Having the ability to spin up these resources quickly to run through the steps and destroy them immediately after is very efficient and effective use of resource.  
4. **Elastic scaling** - Sometimes its hard to know how many instances of a single application/service are required for a certain task. For ex: Maintaining 1sec SLA under peak load. Stress tests tests limits of an application but its hard to predict the application performance every time a change is made. Having the ability to spin up instances on demand takes the guess work out of the equation when automating these types of tasks.

These are just some ways of leveraging the existing integration between Jenkins and Docker but not the only ways. I believe there will be more and more integration [4] between the two that may or may not fall into a standard pattern but something that is very unique to a specific environment. Either way Jenkins is extremely flexible and provides several plugins (listed below) to integrate with Docker. If there are use cases that none of these plugins can address, you can write your own plugin [5]. 

**Jenkins Docker integration plugins**

1. [Dynamic Jenkins Slaves](https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin) - The aim of the docker plugin is to be able to use a docker host to dynamically provision a slave, run a single build, then tear-down that slave
2. [Build and manage Docker images / containers](https://wiki.jenkins-ci.org/display/JENKINS/Docker+build+step+plugin) - This plugin allows to add various docker commands into you job as a build step
3. [Build / Publish Docker images](https://wiki.jenkins-ci.org/display/JENKINS/Docker+build+step+plugin) - This [plugin](https://github.com/jenkinsci/docker-build-publish-plugin) provides the ability to build projects with a Dockerfile, and publish the resultant tagged image (repo) to the docker registry
4. [Integration with DockerHub](https://wiki.jenkins-ci.org/display/JENKINS/DockerHub+Plugin) - This plugin offers integration with DockerHub, using dockerhub hook to trigger Jenkins job to interact with docker image and implement continuous delivery pipelines based on Docker
 
**References**

- [1] https://developer.jboss.org/people/pgier/blog/2014/06/30/on-demand-jenkins-slaves-using-docker
- [2] http://blog.buddycloud.com/post/80771409167/using-docker-with-github-and-jenkins-for-repeatable
- [3] http://www.activestate.com/blog/2014/01/using-docker-run-ruby-rspec-ci-jenkins
- [4] http://www.ebaytechblog.com/2014/05/12/delivering-ebays-ci-solution-with-apache-mesos-part-ii/#.VENjX4eslbw
- [5] https://wiki.jenkins-ci.org/display/JENKINS/Plugin+tutorial

---

Original source: [Jenkins and Docker - The Possibilities](http://randamthoughts.blogspot.jp/2014/10/jenkins-and-docker-possibilities.html)
