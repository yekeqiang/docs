# Docker in Action - Development to Delivery, Part 1

---

Michael Herman

---

Let’s face it: If your development environment varies from production, you never really know how your application will perform once deployed.

Docker solves this problem. It lets you easily spin up **isolated** development environments (containers) that are **near exact snapshots** of the production environment in which it’s deployed. Then when you’re done developing and testing, you can rest assured that your application will behave the same in production since you deploy the entire snapshot (an image), not just the code.

This three part series will teach you everything you need to know about developing with Docker - from setting up your environments and utilizing Flask on Docker to detailing a powerful development workflow that covers setting up a fully functional development environment, on your Mac, and managing continuous integration and delivery.

- **Part 1: Local Docker Setup (current)**
- Part 2: Continuous Integration
- Part 3: Continuous Delivery

**END GOAL**:

![alt](http://resource.docker.cn/steps.jpg)

Let’s begin Part 1…

> Tools used: Python v2.7.8, Flask v0.10.1, Docker v1.3.0, boot2docker v1.3.0

First, some Docker-specific terms:

- A Dockerfile is a file that contains a set of instructions used to create an image.
- An image is used to build and save snapshots (the state) of an environment.
- A container is an instantiated, live image that runs a collection of processes.

> Be sure to check out the Docker [documentation](https://docs.docker.com/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy) for more info - [Dockerfile](https://docs.docker.com/reference/builder/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy), [image](https://docs.docker.com/terms/image/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy), and [container](https://docs.docker.com/terms/container/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy).

With that, let’s get started!

## Docker Setup

Since Darwin (the kernel for OS X) does not have the Linux kernel features required to run Docker containers, we need to install [boot2docker](http://boot2docker.io/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy) - which is a lightweight Linux distribution designed specifically to run Docker. In essence, it starts a small VM that’s configured to run Docker containers.

Make sure you have [VirtualBox](https://www.virtualbox.org/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy) installed then create a new directory called “flask-docker” to house your Flask project. Download the [zip fle](https://github.com/realpython/flask-docker-workflow/blob/master/_starter.zip?raw=true) from the project [repo](https://github.com/realpython/flask-docker-workflow). Unzip the file and add the contents to the “flask-docker” file.

Next follow the instructions from [Docker](https://docs.docker.com/installation/mac/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy) to install both Docker and the official boot2docker package.

To get the boot2docker ISO image and set up the VM, initialize boot2docker:

```
$ boot2docker init
```

Again, make sure you have VirtualBox installed, and then run the following command to start the VM and run the Docker daemon:

```
$ boot2docker start
```

This command should return the `DOCKER_HOST` environment variables, which should look something like this:

```
To connect the Docker client to the Docker daemon, please set:
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/michaelherman/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
```

Copy and paste them directly in your terminal to set the variables. Optional: To make these stick (so they load each time you open your terminal), add them to your bash profile.

## The Dockerfile

Add a new file called Dockerfile in the project directory and add the following:

```
# start with a base image
FROM ubuntu:14.10
MAINTAINER Real Python <info@realpython.com>

# install dependencies
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get install -y supervisor
RUN apt-get install -y python3-pip
RUN pip3 install uwsgi flask

# update working directories
ADD ./app /app
ADD ./config /config

# setup config
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
RUN rm /etc/nginx/sites-enabled/default

RUN ln -s /config/nginx.conf /etc/nginx/sites-enabled/
RUN ln -s /config/supervisor.conf /etc/supervisor/conf.d/

EXPOSE 80
CMD ["supervisord", "-n"]
```

Here we start with an Ubuntu base, install the necessary dependencies on top of it, and then build the Flask application. This file is literally all we need in order to “Dockerize” our app. For more info, check out the [official documentation](https://docs.docker.com/reference/builder/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy).

### Build and run

Let’s get the app up and running!

### Build the image

```
$ docker build --rm -t flask-docker-workflow .
```

Grab a cup of coffee. Or two. This will take some time to build. That said, since Docker caches each step (or *[layer](https://docs.docker.com/terms/layer/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy)*) of the build process from the Dockerfile, rebuilding will be much quicker because only the steps that have changed since the last build are rebuilt.

### Run the container

With the image built, you can now run an instance of your image (the container):

```
$ docker run -p 80:80 flask-docker-workflow
```

Docker should now be running a [supervisord](http://supervisord.org/?utm_source=rainforestqa&utm_medium=link&utm_campaign=deployment-academy) session.

Open your web browser and navigate to the IP address associated with the `DOCKER_HOST` variable - i.e., http://192.168.59.103/, in this example. (Run `boot2docker ip` to get the address.) You should see the text “Flask is running on Docker!” in your browser. Try out the `/data` endpoint as well.

## Conclusion and Next Steps

Once done, kill supervisor (Ctrl-C), then run `boot2docker down` to (gracefully) shutdown the VM. Commit your changes locally, and then push to Github.

Next time we’ll look at a workflow for continuous integration. Cheers!

Also, be sure to check out these excellent resources for more Docker fun:

- [Docker Cheat Sheet](https://github.com/wsargent/docker-cheat-sheet)
- [Docker First Hand](https://www.docker.com/tryit/)

---

Original source: [Docker in Action - Development to Delivery, Part 1](https://blog.rainforestqa.com/2014-11-19-docker-in-action-from-deployment-to-delivery-part-1-local-docker-setup/)




