# dockerizability is better than dockerized

---

Author: Behrooz Nobakht

---

Around 2003, a project was incepted that throughout the last decade has had an influence on how many of us develop software: [Spring Framework](http://spring.io/). Remember?!

The [original mission statement](http://web.archive.org/web/20030814094703/http://www.springframework.org/statement.html) of Spring mentions:



- J2EE should be *easier* to use.
- Program to *interfaces* rather than classes.
- Object-oriented *design* is more important than technology: J2EE.
- Your application code should *not* depend on Spring APIs.


## *“Docker” is to deployment what “Spring Framework” was to development.*


## Design by interface

There was something very interesting about Spring: *It was more fun to read Spring code than to use it*. The following object-orientation principles were prominent in Spring:

- An interface holds a single responsibility
- Interfaces can be composed together

## Easier deployment

Spring Framework set a remarkable example on how JEE applications can be more *easily* deployed:

- *Convention over configuration* allowed minimal configurations necessary for deployment compared to its predecessors: EJB.
- *Container neutrality* allowed deployment of a “Springified” application into basically any application/servlet container or even “no” container.


Lately, the term “dockerized” has been used a lot to refer to an application that is delivered in form of a [docker image](https://docs.docker.com/userguide/dockerimages/). A docker image enables others:

- to reuse the image to create other images
- to use the image and deploy it

To create a docker image, the creator provides a [Dockerfile](http://docs.docker.com/reference/builder/) that captures the configuration that is necessary to build the docker image. In essence, a Dockerfile is the configuration of an application runtime for a docker container. A Dockerfile has interesting properties:

- It is *simple* to read
- It can be *composed* with other Dockerfiles
- It facilitates *neutral* description of an application runtime
- It *encapsulates* platform-dependent configurations through composition

When an application is “dockerized”, its runtime and in turn its architecture design is inevitably reflected in its Dockerfile. That’s why *dockerizability* of an application is better and even more important than just being dockerized.

*dockerizability conditions* can be intuitively generalized as:

- an application runtime delivers service(s) with *limited responsibility*
- an application runtime can be *composed* with that of another
- an application runtime requires *simple configurations*
- an application runtime is *neutral* to its host environment

Interestingly enough, the above break-down and analysis conforms well with the recent [microservices architectures](http://martinfowler.com/articles/microservices.html).

# What does dockerizability represent?

In software requirement analysis, the requirements are typically categorized *as functional* and *[cross-functional](https://twitter.com/martinfowler/status/423453811433558016)* from one perspective; and *quantitative* and qualitative from another. The most challenging ones are claimed to be qualitative cross-functional requirements; e.g. security. Alternative viewpoints also refer to same categories as [discrete and continuous capabilities](http://lizkeogh.com/2014/02/10/discrete-vs-continuous-capabilities/).

> Dockerizability is a cross-functional and qualitative
software requirement for deployment.

The *dockerizability conditions* can be used independently of docker to verify the deployment process of an application. docker is not the only deployment platform for applications, therefore, dockerizability conditions are more important that just dockerized. However, docker presents a successful implementation of the above software requirement because it provides the dockerizability conditions to enable any application to go dockerized.

Of the valuable contributions of Spring Framework were to shape new JSR’s that gradually became part of the standards of JEE stack. Naturally, more implementations of such JSR’s were later born that enriched the community much more.

The real value and potential of docker is how it has started to influence the mindset of deployment process rather than being just a technology: a remarkable contribution towards a more precise definition of *dockerizability* and the software requirement that it represents.



#### Notes and Resources

- [Microservices](http://martinfowler.com/articles/microservices.html), Martin Fowler
- [Why did docker catch on so quickly and why is it so interesting?](http://www.battery.com/powered/the-whole-stack/2014/09/12/why-did-docker-catch-on-so-quickly-and-why-is-it-interesting/), Adrian Cockcroft
- “Dockerizability” is a coinage after familiar “X-ability” in software requirement domain such as reliability, availability, and such. Please write back if there’s already a proper term for it.
- “docker” is used in this note with its trademark form.

---

Original source: [dockerizability is better than dockerized](https://medium.com/@behruz/dockerizability-is-better-than-dockerized-3c08b9dbd84c)