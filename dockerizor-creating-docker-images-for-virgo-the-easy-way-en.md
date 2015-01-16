# Dockerizor – Creating Docker Images for Virgo the Easy Way

---

##### Author: Florian Waibel

---

During the preparations for our EclipseCon talk “[Web Applications with Eclipse RT and Docker in the Cloud](https://www.eclipsecon.org/europe2014/node/918)” about our first Docker project, we spent quite some time building Docker images for Virgo powered applications.


![alt](http://resource.docker.cn/cloud-flask.png)


We investigated how to improve the continuous delivery of such applications using Gradle.
In the first step we used the ‘[Gradle Docker plugin](https://github.com/Transmode/gradle-docker)‘ to get started quickly.
In your build script you specify Docker related configuration like this:

```
apply plugin: 'docker'
docker {
    baseImage = 'ubuntu:14.04'
    maintainer = 'Florian Waibel <fwaibel@eclipsesource.com>'

    version = 'latest'

    useApi true
    hostUrl 'http://localhost:4243'
}
```

Wouldn’t it be cool to configure Virgo just like we did the Docker related stuff?

```
apply plugin: 'dockerizor'

dockerizor {
    javaVersion = '1.7'
    virgoFlavour = 'VJS'
    enableUserRegionOsgiConsole = true
    embeddedSpringVersion = '3.2.4.RELEASE'
}
```

The outcome is the Gradle Plugin ‘[dockerizor](https://github.com/eclipsesource/dockerizor)‘.

We used this Plugin to build Virgo base images for Docker Hub:
- [virgo-tomcat-server](https://registry.hub.docker.com/u/eclipsesource/virgo-tomcat-server/)
- [virgo-jetty-server](https://registry.hub.docker.com/u/eclipsesource/virgo-jetty-server/)
- [virgo-rap-server](https://registry.hub.docker.com/u/eclipsesource/virgo-rap-server/)

In the next post I will describe how we customize and build our own Virgo powered applications (or microservices as they are called nowadays) with dockerizor.

---

Original source: [Dockerizor – Creating Docker Images for Virgo the Easy Way](http://eclipsesource.com/blogs/2014/10/09/dockerizor-creating-docker-images-for-virgo-the-easy-way/)