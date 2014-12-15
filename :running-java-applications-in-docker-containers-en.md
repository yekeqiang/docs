# Running Java applications in Docker containers

---

Author: IIya Dimitrichenko

---

## INTRODUCTION

In this post I would like to describe my approach to running Java applications in Docker containers. The key takeaway is that by reducing the base image, one wins a solid foundation, uncluttered of miscellaneous and unnecessary dependencies, which makes it easier to audit for security issues and avoid potential dependency conflicts. This post, unlike most others on this blog, does not describe a particular usecase for weave network, instead the intent is to shares the outcome of one of many engineering activities Weaveworks team does.

## BACKGROUND AND MOTIVATION

While working on one of [our latest demo](http://weaveblog.com/2014/12/03/using-weave-network-for-bigdata-elasticsearch-apache-spark/), I have yet again faced the issue of how many containers there are on Docker registry for every popular open-source service components. Two containers I needed were ElasticSearch and Apache Spark. After looking around for a while, I figured that reading other people’s Dockerfiles would take me longer then writing my own.

Sometime earlier, I prototyped a [very simple JDK base container](https://registry.hub.docker.com/u/errordeveloper/oracle-jdk/) that had not much apart from *busybox*, *curl* and *Oracle JDK*. The reason I have picked Oracle’s implementation instead of OpenJDK, is that it does not depend on any shared libraries and thereby can be installed on a bare-basic OS, such as one based on *busybox*. I have originally tested it with a large Clojure project that had been compiled to an uberjar, and it worked with just a small addition of C++ runtime (*libstdcpp*) that was needed by one native library that in turns was a dependency of DataStax Cassandra driver. So I decide to give my JDK container a full spin, using an off-the-shelf open-source application.

## APPROACH

Without too much work I have managed to build [a container for ElasticSearch](https://registry.hub.docker.com/u/errordeveloper/weave-elasticsearch-minimal/), and then [Spark](https://registry.hub.docker.com/u/errordeveloper/weave-spark-base-minimal/dockerfile/), which also have some child images all of which are available [on the registry](https://registry.hub.docker.com/u/errordeveloper). Interestingly enough, I came across same library (that is *libsnappy*) and installed *libcpp* again, later I also added another very commonly used library, *zlib*, for some other dependency. Having satisfied these dependencies, I was pretty much done with it. Although, I have quickly realised that JDK is still an overkill for what I am running, and all I really need for most part is the JRE. Hence, I have moved on and build an even smaller [JRE base container](https://registry.hub.docker.com/u/errordeveloper/oracle-jre/), on which ElasticSearch as well as Spark ran happily.

The Dockerfiles for my base containers can be found here:

- [errordeveloper/oracle-jre](https://github.com/errordeveloper/dockerfile-oracle-java/blob/master/jre/Dockerfile)
- [errordeveloper/oracle-jdk](https://github.com/errordeveloper/dockerfile-oracle-java/blob/master/jdk/Dockerfile)

These are pretty generic, and you should be able to use them. The size of the JRE version is 2 times smaller then the size of Docker’s [official](https://registry.hub.docker.com/u/dockerfile/java/) OpenJDK JRE build.

The containers I needed to build primarily were  ElasticSearch and Spark ones, but along the side I have also need to compile a few different JVM-based project, so [errordeveloper/mvn](https://registry.hub.docker.com/u/errordeveloper/mvn/dockerfile/), [errordeveloper/sbt](https://registry.hub.docker.com/u/errordeveloper/sbt/dockerfile/) as well as [errordeveloper/lein](https://registry.hub.docker.com/u/errordeveloper/lein/dockerfile/) came out as a byproduct. You are welcome use my Spark and ElasticSearch Dockerfiles Dockerfiles as a reference, although they are slightly tweaked for the specific purpose I have had, whereby I set DNS to be preferred method of hostname resolution. However, you should be able to use the mvn, sbt and lein images as they are.

## USAGE EXAMPLE: BUILD AND RUN A CLOJURE APP

If you have a Clojure project, you can simply compile it like so from the project directory:

```
docker run -v `pwd`:/io errordeveloper/lein uberjar
```

And then deploy with:

```
docker run -v `pwd`/target:/app errordeveloper/oracle-jre /app/standalone.jar
```

Make sure to set correct the name of the jar to match what you have.

## CONCLUSION

As you can see, it’s practically feasible to avoid pulling an entire Linux distribution to provide a base container for your JVM-based application. The most important advantage of taking this approach is to minimise the base set of dependencies, as any modern applications you are likely to run is in itself already complex enough. The size of image on disk is of some benefit too, it’s not critical however.

This is the first post in the series which is going to cover [our latest demo usecase](http://weaveblog.com/2014/12/03/using-weave-network-for-bigdata-elasticsearch-apache-spark/), please be sure to send any comments or suggestions to [team@weave.works](mailto:team@weave.works) or drop by [#weavenetwork on Freenode](https://botbot.me/freenode/weavenetwork/).

---

Original source: [Running Java applications in Docker containers](http://weaveblog.com/2014/12/09/running-java-applications-in-docker-containers/)