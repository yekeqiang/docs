# Creating an ASP.NET vNext Docker Container using Mono

---

Author: Ross Gardler

---

This tutorial demonstrates how you build Docker container to host ASP.NET vNext applications and was written with the invaluable help of Ahmet Alp Balkan, Software Engineer at Microsoft Azure Linux Team.

[Docker](http://www.docker.com/) is a popular open source containerization tool. It provides an open platform for developers and sys-admins to build, ship, and run distributed applications. Docker enables applications to be assembled from individual and isolated components, each providing a single service. It can therefore eliminate much of the inherent friction between development, QA, and production environments. Many IT organizations are now using Docker because it can ship faster and run the same app, unchanged, on laptops, data center VMs, and any cloud - including Microsoft Azure.

Docker is, at the time of writing, a Linux only technology, although on October 15 2014 Microsoft announced their intention to [bring Docker to Windows Server](http://msopentech.com/blog/2014/10/15/docker-containers-coming-microsoft-linux-server-near/). Fortunately, ASP.NET developers do not need to wait for that work to be continued before they can build and deploy applications as micro-services contained in Docker containers. In this tutorial we build a simple “hello world” Dockerized application using [ASP.NET vNext](http://www.asp.net/vnext) and [Mono](http://www.mono-project.com/) on Linux.

ASP.NET vNext is a lean and composable framework for building web and cloud applications. ASP.NET vNext is fully open source and available on [GitHub](http://github.com/aspnet/home). ASP.NET vNext is currently in preview.

## Setting up a Docker client locally and a Docker host on Azure

Docker containers need a “Docker Host”; we will also need a Linux based machine to act as our Docker client. The Docker client will manage containers on the Docker host. In our case, the host will be a Linux virtual machine running on Microsoft Azure, while the client can be a local machine, a VM on your laptop or a second VM on Azure. The process of creating and configuring both the client and the host is detailed in “[Getting Started with Docker on Microsoft Azure](http://msopentech.com/blog/2014/08/15/getting_started_docker_on_microsoft_azure/)”.

## Creating a base Docker image with ASP.NET vNext and Mono

Once we have a Docker host setup, we can move on to creating a Docker image with Mono and ASP.NET vNext installed.

Docker has two methods to create container images:

1. Manually create images by executing a series of commands inside a Docker container to prepare the desired image and commit the changes. This method is not very elegant or practical; whenever the image needs to be updated, this time-consuming and error-prone process must be repeated.
2. Use a Dockerfile, a plain text file that contains the set of instructions needed to build the Docker image desired. This method allows you to simply script the image building process. You can read the full details in the Dockerfiles section of the Docker documentation, though you’ll find enough information to get started right here.

On our client machine, we must create a new directory (e.g. tutorial/aspnet) and create a file called Dockerfile with the following contents (lines that start with ‘#’ are comments and explain the purpose of each line in this file):

```
# We will use an existing image based on the
# verson 3.10 of Mono.
# This will be automatically retrieved from the Docker Hub.
# Docker Hub is Dockers community repository of pre-built containers
FROM mono:3.10
# Ensure everything is up to date on the machine and install unzip
RUN apt-get -qq update && apt-get -qqy install unzip
# Install ASP.NET vNext and latest K Runtime Environment (KRE)
RUN curl -s https://raw.githubusercontent.com/aspnet/Home/master/kvminstall.sh | sh
RUN bash -c "source /root/.kre/kvm/kvm.sh \
      && kvm upgrade \
      && kvm alias default | xargs -i ln -s /root/.kre/packages/{} /root/.kre/packages/default"
# "ENV" sets an environment variable which will be set
# whenever the container is started
ENV PATH $PATH:/root/.kre/packages/default/bin
```

The file listed above contains the instructions to retrieve a preconfigured Mono image which includes the 3.10 version of Mono. We then install ASP.NET vNext. The required binaries are added to the $PATH environment variable for future use in our application image.

After creating this file, we can go into the tutorial/aspnet folder and execute docker build command as shown below. NOTE: You will need to replace “docker-ubuntu” with the DNS of the Docker host you created earlier:

```
$ export DOCKER_HOST=tcp://docker-ubuntu.cloudapp.net:4243
$ docker --tls build --tag=aspnet aspnet
```

Here we set the DOCKER_HOST environment variable as a convenience. This, in conjunction with the "--tls" flag, means that our Docker command is being executed on the host we previously created. This functionality is explained further in the tutorial linked to in the previous section (that is "Getting Started with Docker on Microsoft Azure"). The remainder of this tutorial assumes that you have set the DOCKER_HOST environment variable.

The first time you run this command it will download the base Mono image, which is a fairly large download. Subsequent executions will be much faster since the image will be available locally. Once complete, you will have a new Docker image tagged as aspnet on your Docker host. This can be verified by listing the images available:

```
$ docker --tls images
REPOSITORY         TAG                IMAGE ID           CREATED            VIRTUAL SIZE
aspnet             latest             ee563c6b77c8       15 seconds ago     738.1 MB
mono               latest             25daec02219d       5 days ago         85.18 MB
```

We now have a Docker image with ASP.NET vNext and Mono installed, but we do not yet have an application on that image.

## Creating an ASP.NET vNext "Hello World!" Image

We will be using David Fowler's [HelloWorldVNext](https://github.com/davidfowl/HelloWorldVNext) sample web application which can be found on GitHub. This is a simple web server which replies to all HTTP requests with a "Hello World” response. We will change to our tutorial directory and use Git to clone the application repository from GitHub into this directory:

```
$ cd ..
$ git clone https://github.com/davidfowl/HelloWorldVNext.git
```

Now that we have our application code locally we can create a new Docker container for our application. We could simply add our application code to the image we created above by adding the necessary configuration to the previous Dockerfile. However, it would be more effective to create a separate Dockerfile. This would mean that we can both manage and reuse the aspnet image created above independently of the application that it contains. In the case of a simple “hello world” application like this it may not seem important. However, consider an application that consists of a number of ASP.NET vNext micro-services. By reusing the same base image in each micro-service we can easily upgrade all containers.

In the Dockerfile above the first command (“FROM”) specified the base image. Our aspnet image used “FROM nmono:3.10” to indicate that we wanted to build on top of the 3.10 release of Mono. Our hello world container will build upon the latest aspnet image.

The git clone command, above, will have create a folder called “HelloWorldVNext”. Our new Dockerfile will reside in this directory and will have the following content:

```
FROM aspnet
# copy the contents of the local directory to /app/ on the image
ADD . /app/
# set the working directory for subsequent commands
WORKDIR /app
# fetch the NuGet dependencies for our application
RUN kpm restore
# set the working directory for subsequent commands
WORKDIR /app/src/helloworldweb
# expose TCP port 5000 from container
EXPOSE 5000
# Configure the image as an executable
# When the image starts it will execute the “k web” command
# effectively starting our web application
# (listening on port 5000 by default)
ENTRYPOINT ["k", "web"]
```

As before we can now build this application image using this Dockerfile and tag it with a name such as myapp:

```
$ docker --tls build -t myapp HelloWorldVNext
...
$ docker --tls images
REPOSITORY         TAG                IMAGE ID           CREATED            VIRTUAL SIZE
myapp              latest             6c5b916b7d43       10 minutes ago     793.5 MB
aspnet             latest             ee563c6b77c8       13 minutes ago     738.1 MB
mono               latest             25daec02219d       5 days ago         85.18 MB
```

This image is a self-contained and portable implementation of our HelloWorldVNext application and all its dependencies.

#### Debugging Hint

If you have a problem building the container you can interactively connect to the aspnet image and run the commands manually to see their outut. To do this run “docker --tls run –t –i [aspnet or myapp] bash” and you will create a container from the aspnet image and connect to it via a Bash terminal.

## Starting the Docker Container

To start a Docker container using the myapp image we have just created, we need to run:

```
$ docker --tls run -d -t -p 8080:5000 myapp
```

This command performs the following actions:

Start a container from image myapp. Note that since we specified an ENTRYPOINT in our Dockerfile, effectively making an executable container, there is no need to specify a command to run.

- -d switch to run it as a daemon

- -t switch to allocate a pseudo-TTY for the container.

- -p switch binds container's port 5000 to the host Azure VM's port 8080.

The output of this command is the id of the container we just started. We can use this id with docker logs command to examine its current status:

```
$ docker --tls logs -t 111ad8a44aa523ba9ba6badc350e758b70a1dbb39b034842947b24cc17908fc0
2014-09-10T00:53:55.93952Z Started
```

It appears that our application has started. We can run docker ps command to see how it is doing:

```
$ docker --tls ps
CONTAINER ID       IMAGE               COMMAND             CREATED             STATUS             PORTS                   NAMES
111ad8a44aa5      myapp:latest       "k web"             45 seconds ago     Up 43 seconds       0.0.0.0:8080->5000/tcp   ecstatic_fermat
```

According to the output of the docker ps command above, our container is up. Take a note of the “Names” column here. This is a randomly generated locally unique name which can be used in place of the ID returned above. It’s much easier to remember!

While our application is now running, we are not quite done yet. We need to enable the HTTP endpoint on the Docker host Virtual Machine we created earlier. This can be done using the Azure management portal or via the cross-platform CLI tools with the following command (replace “docker-ubuntu” with your cloud service name):

```
$ azure vm endpoint create docker-ubuntu 80 8080 -n HTTP
info:   Executing command vm endpoint create
+ Getting virtual machines
+ Reading network configuration
+ Updating network configuration
info:   vm endpoint create command OK
```

This command creates a TCP endpoint named HTTP and binds the host Azure VM's private port 8080 to public port 80.

Since our application was running at port 5000 inside the container and we bound this to port 8080 of the Docker host in our docker run command, the public port 80 can be used to communicate our application from outside world. That is public port 80 maps to 8080 on the host VM which in turn is mapped to 5000 on our container.

We can now navigate to port 80 on our Docker host machine where we will find our web server application ready and willing to respond with "Hello World":

```
$ curl http://ubuntu-docker.cloudapp.net:80
Hello World
```

Voilà! Our container is very much alive and running.

## Stopping the Container

To stop this container, you can run docker stop command with container name returned from the "docker --tls ps" command above:

```
$ docker --tls stop ecstatic_fermat
```

## Conclusion

Using this aspnet Dockerfile in this tutorial you can take ASP.NET vNext for spin in a container environment. Remember this is a preview of the next version of ASP.NET, it should not be used for production. The ASP.NET vNext team would love to hear your feedback via the projects [GitHub issue tracker](https://github.com/aspnet/home).

---

Original source: [Creating an ASP.NET vNext Docker Container using Mono](http://msopentech.com/blog/2014/11/07/creating-asp-net-vnext-docker-container-using-mono-2/)