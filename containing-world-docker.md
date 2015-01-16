# Containing the World With Docker

---

##### Author: Evan Machnic

---

Today, I want to discuss the latest rage in the virtualization world. Everyone’s talking about Docker, and it’s so much more than a virtual machine.

![alt](http://resource.docker.cn/docker-logo.png)

If you haven’t heard of Docker, let me introduce you. Docker allows you to neatly package up applications and software in a nice little container. You can then take your container and run it wherever you want without having to worry about experiencing any downtime. For anyone who has ever tried to run an application on different systems, this should really get you excited.

Upon initially looking at Docker, you may be wondering what you could use this for. Let’s say you’re developing a Ruby on Rails application, you can use Docker to keep your application dependencies, like Redis or PostgreSQL, contained without having to worry about conflicts with the rest of your host system. This is a huge benefit for development teams because you can distribute the same container to each team member, allowing them to start development on a new app in a fraction of the time that it would take without Docker.

Another great use case of Docker is for deployment of applications. Let’s say we have a container for our application — one for our database and another for Redis. We can deploy and update those containers wherever we want. What does that mean for us? It means it is incredibly easy to have a production server on one provider and staging on another. As long as the servers have Docker running, the different containers will run the same everywhere.

## Installing Docker

Before we get into containers, we’ll need to get Docker running locally. You can do this by installing the package for your system (tip: you can find yours [here](https://docs.docker.com/installation/#installation)). Running a Mac? You’ll need to install the [boot2docker application](http://boot2docker.io/) before using Docker. Once that’s set up, you’re ready to start running.

## Docker Commands

All interactions with Docker are done through the command line, so if you’ve never used the terminal, now is the time to get started. The main command you’ll use is the `docker` command. Running this command without any additional options will display the help screen, and example of this is shown below.

```
~ ❯ docker
Usage: docker [OPTIONS] COMMAND [arg...]
 -H=[unix:///var/run/docker.sock]: tcp://host:port to bind/connect to or unix://path/to/socket to use

A self-sufficient runtime for Linux containers.

Command List:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from a container's filesystem to the host path
    diff      Inspect changes on a container's filesystem
    events    Get real-time events from the server
    export    Stream the contents of a container as a tar archive
    history   Show the history of an image
    images    List images
    import    Create a new filesystem image from the contents of a tarball
    info      Display system-wide information
    inspect   Return low-level information on a container
    kill      Kill a running container
    load      Load an image from a tar archive
    login     Register or log in to the Docker registry server
    logs      Fetch the logs of a container
    port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
    pause     Pause all processes within a container
    ps        List containers
    pull      Pull an image or a repository from a Docker registry server
    push      Push an image or a repository to a Docker registry server
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image to a tar archive
    search    Search for an image on the Docker Hub
    start     Start a stopped container
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Lookup the running processes of a container
    unpause   Unpause a paused container
    version   Show the Docker version information
    wait      Block until a container stops, then print its exit code
```

To ensure everything is working properly, go ahead test docker by running the following command:

```
~ ❯ docker run hello-world
```

This command looks for an image named “hello-world” on your local computer. If the image is not found Docker will attempt to download the image. A new container based on the image is created and ran. Here is an example of what this looks like:

```
~ ❯ docker run hello-world
Unable to find image 'hello-world' locally
Pulling repository hello-world
ef872312fe1b: Download complete 
511136ea3c5a: Download complete 
7fa0dcdc88de: Download complete 
Hello from Docker.
```

This message shows that your installation is working correctly. There are a few things going on internally here, lets take a look at what happened:

1. The Docker client attempted to find the image locally, but failed

2. The Docker client contacted the Docker daemon.

3. The Docker daemon pulled the “hello-world” image from the Docker Hub.

4. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
 
5. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.

Now, try something more ambitious — run an Ubuntu container with:


```
$ docker run -it ubuntu bash
```

## The Dockerfile

Now that we know Docker is working, let’s dive into the Dockerfile. Here’s example code that will create a container from the Ubuntu image and prints “Howdy from Code School!!!”

```
FROM ubuntu
CMD echo 'Howdy from Code School!!!'
```

The Dockerfile offers many different keywords that you can use to describe your container. I’ve made a list of the most important ones below, so keep them handy:

```
FROM — Specify which image to use for the container.
RUN — Run a command on the container. Used for things like installing packages, etc.
EXPOSE — Open ports from the Docker container to the host.
CMD — The default command to run when the container is started. Can be overridden from the command line at runtime.
ENTRYPOINT — Similar to the CMD option in that it will be the default command that is run but this one cannot be overridden from the command line.
```

There are even more options and I recommend checking out the documentation to understand what they are and how to use them. To build our container using the Dockerfile, we can run the following command:

```
~ ❯ docker build -t code_school .
```

When this command finishes, you can run the container with the command:

```
~ ❯ docker run code_school
```

Awesome job! Now you’re ready to start playing around with Docker — you’ll notice the difference it makes with your projects pretty quickly.

---

Original source: [Containing the World With Docker](https://www.codeschool.com/blog/2014/10/21/containing-world-docker/)

