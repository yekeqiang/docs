# Building Docker Images within Docker Containers via Jenkins

---

##### Author: Dan Tehranian

---

If you’re like me and you’ve Dockerized your build process by [running your Jenkins builds from within dynamically provisioned Docker containers](http://dantehranian.wordpress.com/2014/09/08/docker-jenkins-dynamically-provisioning-sles-11-build-containers/), where do you turn next? You may want the creation of any Docker images themselves to also happen within Docker containers. In order words, [running Docker nested within Docker](https://blog.docker.com/2013/09/docker-can-now-run-within-docker/) (DinD).


I’ve recently published a Docker image to facilitate building other Docker images from within Jenkins/Docker slave containers. Details at:

- [https://registry.hub.docker.com/u/tehranian/dind-jenkins-slave](https://registry.hub.docker.com/u/tehranian/dind-jenkins-slave)
- [https://github.com/tehranian/dind-jenkins-slave](https://github.com/tehranian/dind-jenkins-slave)

Why would one want to build Docker images nested within Docker containers?

- For consistency. If you’re building your JARs, RPMs, etc, from within Docker containers, it makes sense to use the same high-level process for building other artifacts such as Docker images.

- For Docker version freedom. As I mentioned in a [previous post](http://dantehranian.wordpress.com/2014/09/08/docker-jenkins-dynamically-provisioning-sles-11-build-containers/), the [Jenkins/Docker plugin](https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin) can [be](https://github.com/jenkinsci/docker-plugin/issues/93) [finicky](https://github.com/jenkinsci/docker-plugin/issues/90) with regards to compatibility with the version of Docker that you are running on your base OS. In order words, Jenkins/Docker plugin 0.7 [will not work with Docker 1.2+](https://github.com/jenkinsci/docker-plugin/issues/90), so if you really need a feature from a newer version of Docker when building your images you either have to wait for a fix from the Jenkins plugin author, or you can run Docker-nested-in-Docker with the Jenkins plugin-compatible Docker 1.1.x on the host and a newer version of Docker nested within the container. Yes, this actually works!

---

Original source: [Building Docker Images within Docker Containers via Jenkins](http://dantehranian.wordpress.com/2014/10/25/building-docker-images-within-docker-containers-via-jenkins/)