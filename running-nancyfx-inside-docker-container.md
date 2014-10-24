# Running NancyFX inside a Docker container

---

##### Author: Ben Hall

---

A number of years ago I described how to run a [NancyFX](http://nancyfx.org/) based application on [Heroku via Mono](http://blog.benhall.me.uk/2012/01/experiment-deploying-c-mono-on-heroku/). Recently I’ve been focusing my attention on Docker and considered the same problem, how can you run NancyFX as a Docker container? At a high level, a Docker container is an isolated process comprising of the application and it’s direct dependencies. With everything isolated we can manage multiple containers more effectively. More details can be found at [https://www.docker.com/whatisdocker/](https://www.docker.com/whatisdocker/).

It turns out to be fairly straight forward. Firstly you need a Docker image with Mono installed which I’ve previously built and made available an image at [https://github.com/BenHall/docker-mono](https://github.com/BenHall/docker-mono).

This image can be used as the foundation for the Nancy project we want running inside the Docker container. A Docker container is baed on an image that can be pre-built and shared. These images are created based on a Dockerfile, a set of commands that are executed in order to configure and run the application.

A Dockerfile for a NancyFX first needs a base Docker image. In this case we’re using the image created with Mono installed.

```
FROM benhall/docker-mono
```

We then copy the source code for the application into it’s own directory.

```
COPY . /src
```

Setting the working directory ensures that future commands are executed from inside this folder

```
WORKDIR /src
```

Using xbuild we can compile our project

```
RUN xbuild Nancy.Demo.Hosting.Docker.sln
```

As the application is sandboxed and isolated we need to expose any ports which we want to be accessible. In this scenario the self-hosted web app maps to port 8080

```
EXPOSE 8080
```

Finally, when the container boots up we can specify which application needs to run.

```
ENTRYPOINT ["mono", "src/bin/Nancy.Demo.Hosting.Docker.exe"]
```

This is everything required to define a Docker image capable of running a Nancy project. The complete file can be found at [https://raw.githubusercontent.com/BenHall/nancy-demo-hosting-docker/master/Dockerfile](https://raw.githubusercontent.com/BenHall/nancy-demo-hosting-docker/master/Dockerfile).

To launch the container we first need to build the image. This image can be reused, shared and made available for other environments or people to use. Using the -t we can define a friendly tag to help recognise the image followed by the path to the repository or directory containing the Dockerfile

```
$ docker build -t benhall/nancy-demo-hosting-docker github.com/benhall/nancy-demo-hosting-docker
```

Once built the image can be started as a container with port 8080 accessible.

```
$ docker run -d --name nancy-demo -p 8080 benhall/nancy-demo-hosting-docker
```

Inside the container the NancyFX site is up and running. The docker port command tells us the port number of the host mapped to the port inside the container. This allows us to run multiple identical containers on a host as they’ll all be assigned different ports.

```
$ docker port nancy-demo 8080
0.0.0.0:49153
```

A quick curl and you can see the Nancy and Mono response headers from inside our container.

```
$ curl -I 0.0.0.0:49153
HTTP/1.1 200 OK
Nancy-Version: 0.9.0.0
Content-Type: text/html
Server: Mono-HTTPAPI/1.0
```

While this is a simple example it demonstrates the starting point of how you can create isolated and repeatable containers for different parts of your software stack. With the recent announcement that [Microsoft and Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/) are working more closely together this is here to stay and the future of how we deploy software. If you’re interested in hearing more then I’m speaking on “Architecting .NET Applications for Docker and Container Based Deployments” at [NDC London](http://www.ndc-london.com/) in December. Alternatively please feel free to email me (Blog @ BenHall.me.uk) or via Twitter ([@Ben_Hall](http://twitter.com/@Ben_Hall)).

---

Original source: [Running NancyFX inside a Docker container](http://blog.benhall.me.uk/2014/10/running-nancyfx-inside-docker-container/)