# Deploying NGINX and NGINX Plus with Docker

---

##### Author: RICK NELSON

---

[Docker](https://www.docker.com/) is an open platform for building, shipping, running and orchestrating distributed applications. NGINX, being a software application, is a great use case for Docker and we have created an open source [NGINX image](https://registry.hub.docker.com/_/nginx/) on [Docker Hub](https://registry.hub.docker.com/), the repository of Docker images. This article will describe how you can deploy NGINX with Docker using the image from Docker Hub or create your own Docker image using NGINX Plus,  the enhanced and commercially supported version of NGINX.

## Introduction

The Docker open platform includes the Docker Engine – the open source runtime that builds, runs and orchestrates containers, and Docker Hub – a hosted service where Dockerized applications are distributed, shared, and collaborated on across the entire development community or within the confines of a specific organization. Docker containers allow developers to focus their efforts on application “content” by separating applications from the constraints of infrastructure. Dockerized applications are instantly portable to any infrastructure – laptop, bare-metal server, VM or cloud – making them modular components that can be readily assembled/re-assembled into full featured distributed applications that can be continuously innovated on in-real time. For more information about Docker please see: [What is Docker](https://www.docker.com/whatisdocker/).

## Using the NGINX Docker Image

You can create an instance of NGINX in a Docker container using the [NGINX image](https://registry.hub.docker.com/_/nginx/) from Docker Hub. For more information on Docker please see the [NGINX Docker Hub page](https://registry.hub.docker.com/_/nginx/) and the [Docker documentation](http://docs.docker.com/).

Lets start with a very simple example. To launch an instance of NGINX running in a container, using the default NGINX configuration, run the command:

```
#docker run --name mynginx1 -P -d nginx
fcd1fb01b14557c7c9d991238f2558ae2704d129cf9fb97bb4fadf673a58580d
```

This will create a container named mynginx1, based off of the NGINX image and will run it in detached mode, which means that the container will be started and will stay running until stopped but will not be listening to the command line. We will discuss later how we will interact with the container.

The NGINX image exposes ports 80 and 443 in the container and the -P option tells Docker to map those ports to ports on the Docker host using random port numbers between 49153 and 65535. We do this because if we create multiple NGINX containers on the same Docker host, we will create a conflict on ports 80 and 443. The port mappings are dynamic and are set each time the container is started or restarted. If you want the port mappings to be static you can set them manually with the -p option. The long form of the Container Id will be returned.

We can see that the container was created, is running and the port mappings by running the command:

```
#docker ps
CONTAINER ID  IMAGE         COMMAND               CREATED         STATUS         
PORTS                                         NAMES
fcd1fb01b145  nginx:latest  "nginx -g 'daemon of  16 seconds ago  Up 15 seconds  
0.0.0.0:49166->443/tcp, 0.0.0.0:49167->80/tcp mynginx1
We can also see that NGINX is running by making an HTTP request to port 49166 on the Docker host, the port on the host to which port 80 in the container has been mapped to, and we should see the default NGINX welcome page:

#curl http://localhost:49166
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
 body {
 width: 35em;
 margin: 0 auto;
 font-family: Tahoma, Verdana, Arial, sans-serif;
 }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Working with the NGINX Docker container

So now we have a working NGINX Docker container, but how do we manage the content and the NGINX configuration?  And what about logging?  It is common to have SSH access to NGINX instances, but Docker containers are generally intended to be for a single purpose, in this case running NGINX, so the NGINX image does not have OpenSSH installed and for normal operations there is no need to get shell access directly to the NGINX container. We will use other methods supported by Docker. For a detailed discussion of alternatives to SSH access see [Why you don’t need to run SSHd in your Docker Containers](https://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/).

### Managing Content and Configuration files

There are multiple ways you can mange the NGINX content and configuration files and we will cover a few options.

### Maintain the content and configuration on the Docker host

When the container is created we can tell Docker to mount a local directory on the Docker host to a directory in the container. The NGINX image uses the default NGINX configuration, so the root directory is /usr/nginx/share/html and the configuration files are in /etc/nginx. For example, lets say that the content is in the local directory /var/www and the configuration files are in /var/nginx/conf. To start a container that uses the files in these local directories we would use the command:

```
# docker run --name mynginx2 -v /var/www:/usr/share/nginx/html:ro \
  -v /var/nginx/conf:/etc/nginx:ro -P -d nginx
```

Now any change made to the files in the local directories /var/www and /var/nginx/conf will be reflected in the directories /usr/nginx/share/html and /etc/nginx in the container. The :ro option causes these directors to be read only inside the container.

#### Copy the files from the Docker host

Another option is to have Docker copy the content and configuration files from a local directory on the Docker host when a container is created. Once a container is created, the files will be maintained by creating a new container when the files change or by modifying the files in the container. A simple way of copying the files is to create a Dockerfile to generate a new Docker image, based off of the NGINX image from Docker Hub. When copying files in the Dockerfile, the path to the local directory is relative to the build context where the Dockerfile is located. For this example, the content is in the directory content and the configuration files are in the directory conf, both in the same directory as the Dockerfile. The NGINX image has the default NGINX configuration files, including default.conf and example_ssl.conf in /etc/nginx/conf.d. Since we want to use the configuration files from the host, we will add commands in the Dockerfile to delete these default files. We can create the following Dockerfile:

```
FROM nginx
RUN rm /etc/nginx/conf.d/default.conf
RUN rm /etc/nginx/conf.d/example_ssl.conf
COPY content /usr/nginx/share/html
COPY conf /etc/nginx
```

We can then create our own NGINX image by running the following command from the directory where the Dockerfile is located:

```# docker build -t mynginx_image1 .```

Note the “.” at the end of the command. This tells Docker that the build context is the current directory. The build context contains the Dockerfile and the directories to be copied. Now we can create a container using the image by running the command:

```# docker run --name mynginx3 -P -d mynginx_image1```

If we want to make changes to the files in the container you can use a helper container as described below.

#### Maintain files in the container

As mentioned previously, we are not able to get SSH access to the NGINX container, so if we want to edit the content or configuration files directly we can use a helper container that has shell access. In order for the helper container to have access to the files, [volumes](https://docs.docker.com/userguide/dockervolumes/) must be specified for the image. We can do that by creating a new image that has the proper volumes specific. Assuming we want to copy the files as in the example above while also specifying volumes for the content and configuration files we can use the following Dockerfile:

```
FROM nginx
COPY content /usr/nginx/share/html
COPY conf /etc/nginx
VOLUME /usr/nginx/share/html
VOLUME /etc/nginx
```

We can then create the new NGINX image by running the following command:

```# docker build -t mynginx_image2 .```

Now we can create an NGINX container using the image by running the command:

```# docker run --name mynginx4 -P -d mynginx_image2```

We can then start a helper container with a shell and access the content and configuration directories of the NGINX container we created in the previous example by running the command:

```
# docker run -i -t --volumes-from mynginx4 --name mynginx4_files debian /bin/bash
root@b1cbbad63dd1:/#
```

This creates an interactive container named mynginx4_files that runs in the foreground with a persistent standard input (-i) and a tty (-t)  and mounts all the volumes defined in the container mynginx4 as local directories in the new mynginx4_files container. This container will use the Debian image from Docker Hub which is the same operating system used by the NGINX image. Since all of the examples shown so far are using the NGINX image and therefore Debian, it is more efficient to use Debian for the helper container rather then having Docker load another operating system. After the container is created it will run the bash shell. You will be presented with a shell prompt for the container and can then modify the files as needed. You can also install other tools in this container, such as Puppet or Chef to manage these files. If you exit the shell with exit, the container will terminate. If you want to exit while leaving the container running use Control-p followed by Control-q. The container can be started and stopped with the commands:

```# docker start mynginx4_files```

and

```# docker stop mynginx4_files```

Shell access can be regained to a running container with the command:

```# docker attach mynginx4_files```

### Managing Logging

#### Default Logging

The NGINX image has setup the main NGINX access and error logs to go to the Docker log collector. This is done by linking them to stdout and stderr. This will cause all messages to either log to be stored in the file /var/lib/docker/containers/<container id>/<container id>-json.log on the Docker Host. <container id> is the long form Container Id returned when you create a container. You can get the long form Id for a container with the command:

```# docker inspect --format '{{ .Id }}' <container name>```

The access and error message are combined into this one log file. You can use both the Docker command line and the [Docker Remote API](https://docs.docker.com/reference/api/docker_remote_api_v1.14/#get-container-logs) to extract the log messages. From the command line you can use the command:

```# docker logs <container name>```

The Docker Remote API can be used to extract just the access log messages (stdout) or error log messages (stderr). To enable the Docker Remote API, add the following line to the file /etc/default/docker:

DOCKER_OPTS='-H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock'
When Docker is restarted, it will listen on port 4243 (the port can be changed) for HTTP API requests and also on a socket. To get all the messages you can issue the GET request:

`http://<docker host>:4243/containers/<container name>/logs?stdout=1&stderr=1`

To limit the output to the access log messages, include only stdout=1 and to limit the output to the error log messages, include only stderr=1. There are additional options available that are described in the Docker documentation.

#### Customized Logging

If you want to implement another method of log collection or if you want to specify logging at different levels of the NGINX configuration such as servers and locations, you can use a volume for the directory or directories where you want to store the log files in the container. You can then use a helper container to access the log files and you can use whatever logging tools you like. To implement this, a new image can be created that contains the volume or volumes for the logging files. For example, if we are going to configure NGINX to store log files in /var/log/nginx/log, then we can start with the Dockerfile shown in the earlier example of copying files from the Docker host to the container and we can simply add a volume declation for this directory:

```
FROM nginx
COPY content /usr/nginx/share/html
COPY conf /etc/nginx
VOLUME /var/log/nginx/log
```

We can then create an image as described above and using this image we can create an NGINX container and a helper container which will now have access to the log directory. This helper container can have any desired logging tools installed.

### Controlling NGINX

Since we do not have access to the command line of the NGINX container directly we cannot control NGINX using the nginx command. Fortunately NGINX can be controlled by [signals](http://nginx.org/en/docs/control.html) and Docker provides the ability to send signals to a container using the Docker kill command. For example, to reload the NGINX configuration you can run the command:

```docker kill -s HUP <container name>```

If you want to restart the NGINX process you should restart the container with the command:

```docker restart <container name>```

## Deploying NGINX Plus with Docker

So far we have been discussing using the open source version of NGINX with Docker, but the commercial version, NGINX Plus, can also be used with Docker. The difference is that since NGINX Plus is a commercial offering, an image is not available at Docker Hub, so you have to create your own. Fortunately that is quite easy to do.

### Creating a Docker Image of NGINX Plus

In order to generate your own image of NGINX Plus, you have to first create a Dockerfile. We have provided example Dockerfiles for Ubuntu 14.04 and CentOS 7 along with instructions on how to create the images, and these Dockerfiles can be used as templates for other operating systems. For both Ubuntu and CentOS you have to download your version of the files nginx-repo-crt and nginx-repo-key from cs.nginx.org. A login to cs.nginx.org is provided to customers who have purchased or are evaluating NGINX Plus and this key and certificate provide access to the NGINX Plus repository. These files should be copied to the Docker build context, which is the directory where the Dockerfile resides.

As with the NGINX open source Dockerfile, the default NGINX access and error logs are linked to the Docker log collector. No volumes are specified, but they can be added if descried, or these Dockerfiles can be used to create base images from which new images can be created which have volumes specified, as described above.

No files are copied from the Docker host when creating a container. This can also be added to the Dockerfiles, or the image created can be used as the basis for another image as described above.

#### Ubuntu Dockerfile

```
FROM ubuntu:14.04
MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"
# Set the debconf frontend to Noninteractive
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
RUN apt-get update && apt-get install -y -q wget apt-transport-https
# Download certificate and key from the customer portal https://cs.nginx.com
# and copy to the build context
ADD nginx-repo.crt /etc/ssl/nginx/
ADD nginx-repo.key /etc/ssl/nginx/
# Get other files required for installation
RUN wget -q -O /etc/ssl/nginx/CA.crt https://cs.nginx.com/static/files/CA.crt
RUN wget -q -O - http://nginx.org/keys/nginx_signing.key | apt-key add -
RUN wget -q -O /etc/apt/apt.conf.d/90nginx https://cs.nginx.com/static/files/90nginx
RUN printf "deb https://plus-pkgs.nginx.com/ubuntu `lsb_release -cs` nginx-plus\n" >/etc/apt/sources.list.d/nginx-plus.list
# Install NGINX Plus
RUN apt-get update && apt-get install -y nginx-plus
#forward request logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
```

####CentOS Dockerfile

```
FROM centos:centos7
MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"
RUN yum install -y wget
# Download certificate and key from the customer portal https://cs.nginx.com
# and copy to the build context
ADD nginx-repo.crt /etc/ssl/nginx/
ADD nginx-repo.key /etc/ssl/nginx/
# Get other files required for installation
RUN wget -q -O /etc/ssl/nginx/CA.crt https://cs.nginx.com/static/files/CA.crt
RUN wget -q -O /etc/yum.repos.d/nginx-plus-7.repo https://cs.nginx.com/static/files/nginx-plus-7.repo
# Install NGINX Plus
RUN yum install -y nginx-plus
# forward request logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
```

### Creating the NGINX Plus Image

Once you have Dockerfile and the nginx-repo.crt and nginx-repo.key files in the same directory, you can generate a Docker image named nginxplus by running the following command from that directory:

```# docker build --no-cache -t nginxplus .```

Note the –no-cache option. This tells Docker to build the image from scratch and ensures that the latest version of NGINX Plus will be installed. If this option is not specified and an image has been built from this Dockerfile previously, then the new image will use the NGINX Plus version from the Docker cache. This is because we have not specified the NGINX version in the Dockerfile so the Dockerfile does not need to be changed for each new release of NGINX Plus. If you want to use the NGINX Plus version from the previously built image, then you can leave this option off.

If the image was created successfully you will see it by running the command:

```
# docker images nginxplus
REPOSITORY  TAG     IMAGE ID      CREATED        VIRTUAL SIZE
nginxplus   latest  ef2bf65931cf  6 seconds ago  281.3 MB
```

You can then use this NGINX Plus image to create a container named mynginxplus with the command:

```# docker run --name mynginxplus -P -d nginxplus```

These containers can be controlled and managed in the same way as the NGINX open source containers described above.

## Summary
NGINX and Docker are tools that work extremely well together. By using the NGINX open source image from the Docker Hub repository or by creating your own NGINX Plus image, you can easily spin up new instances of NGINX in Docker containers. You can also take these images and easily create new Docker images from them to give you even more control of your containers and how you manage them. One thing to keep in mind is that NGINX Plus is sold on a per instance basis, so each Docker container running NGINX Plus will require an NGINX Plus subscription.

There is much more to Docker then we have been able to cover in this article.  More information is available at www.docker.com.

---

Original source: [Deploying NGINX and NGINX Plus with Docker](http://nginx.com/blog/deploying-nginx-nginx-plus-docker/)
