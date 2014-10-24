# How to Optimize Your Dockerfile

---

##### Author: Maxime Heckel

---

Difficulty: Beginner

## Prerequisites

### Docker

To follow this tutorial, you need a working installation of Docker on your computer. [The official docker documentation](https://docs.docker.com/installation/) will help you to get Docker running if that’s not already the case.

## Introduction to Dockerfiles

Dockerfiles are text documents that allow you to build images for Docker automatically. They contain all the instructions and commands that would need to be run manually in your container in order to get the desired image.

To build a Docker image from a Dockerfile, you need to run the docker build -t [TAG][PATH_TO_YOUR_DOCKERFILE] command and the image will begin to build itself from the intructions you wrote.

For example, if you were creating a Dockerfile for Ruby On Rails in your current directory you would run :

```
docker build -t rubyonrails .
```

Note: although helpful, the -t flag is not mandatory.

For more information on Dockerfiles and how to write them you can read [the official dockerfile documentation](https://docs.docker.com/reference/builder/).

## Optimizing your Dockerfile : a simple example

The goal of this tutorial is to show some tricks that will help us write the most efficient Dockerfiles possible.

Let’s take the following Dockerfile as an example :

```
FROM ubuntu
MAINTAINER Maxime Heckel
 
RUN apt-get update
RUN apt-get install -y wget build-essential python python-dev python-pip python-virtualenv
RUN wget http://nodejs.org/dist/node-latest.tar.gz
RUN tar xvzf node-latest.tar.gz
RUN cd node-v* && ./configure && CXX="g++ -Wno-unused-local-typedefs" make && CXX="g++ -Wno-unused-local-typedefs" make install
RUN echo 'n# Node.jsnexport PATH="node_modules/.bin:$PATH"' >> /root/.bashrc
 
WORKDIR /data
```

This particular Dockerfile creates a working Node.JS image. Let’s run it to see the build process in action. Create a file named Dockerfile, paste the previous lines into it, and run the following command:

```
docker build -t node .
```

You should see the following output :

```
Step 0 : FROM ubuntu
---> 6b4e8a7373fe
Step 1 : MAINTAINER Maxime Heckel
---> Running in 00b1432856b1
---> 7bf3089aa3f0
Removing intermediate container 00b1432856b1
Step 2 : RUN apt-get update
---> Running in 503e6a8c4dab
Ign http://archive.ubuntu.com trusty InRelease
Ign http://archive.ubuntu.com trusty-updates InRelease
Ign http://archive.ubuntu.com trusty-security InRelease
Ign http://archive.ubuntu.com trusty-proposed InRelease
Get:1 http://archive.ubuntu.com trusty Release.gpg [933 B]
....
 
Fetched 20.4 MB in 28s (710 kB/s)
Reading package lists...
---> 471ecb17b4d8
Removing intermediate container 503e6a8c4dab
Step 3 : RUN apt-get install -y wget build-essential python python-dev python-pip python-virtualenv
---> Running in 4938a43d70de
Reading package lists...
Building dependency tree...
Reading state information...
...
---> a177e9b08ef4
Removing intermediate container 55e4357bdec2
Step 7 : RUN echo 'n# Node.jsnexport PATH="node_modules/.bin:$PATH"' >> /root/.bashrc
---> Running in 75bd370d874b
---> 396f2b3a3edd
Removing intermediate container 75bd370d874b
Step 8 : WORKDIR /data
---> Running in de1f81e1a473
---> 6865656487bc
Removing intermediate container de1f81e1a473
Successfully built 6865656487bc
```

After several minutes we will have a working image for our Node.JS projects.

### Layers

Before we can talk about optimization we need to understand the concept of layers. Let’s analyze the previous output :

- The MAINTAINER command (step 1) creates a container with the ID 00b1432856b1
- Then this container is stopped resulting in a new image with the ID : 7bf3089aa3f0
- The RUN command (step 2) creates a new container with another ID : 503e6a8c4dab which is also stopped at the end of the instruction and results in another image with the ID : 471ecb17b4d8

The important thing to understand here is that each instruction run during the build process results in a new layer. Images and layers are not different from each other, each layer has its own image.

Let’s look at the layers of our image to illustrate that point with this command:

```
docker images -t
```

we should get the following output:

```
b18d0a2076a1 Virtual Size: 192.6 MB
67b66f26d423 Virtual Size: 192.7 MB
25c4824a5268 Virtual Size: 192.7 MB
8b1c48305638 Virtual Size: 192.8 MB
c900195dcbf3 Virtual Size: 194.9 MB
6b4e8a7373fe Virtual Size: 194.9 MB Tags: ubuntu:latest
7bf3089aa3f0 Virtual Size: 194.9 MB
471ecb17b4d8 Virtual Size: 215.3 MB
9a82dc63083c Virtual Size: 391.8 MB
1a49a0cf1c17 Virtual Size: 405.4 MB
59d93b6e3347 Virtual Size: 466.6 MB
a177e9b08ef4 Virtual Size: 573.9 MB
396f2b3a3edd Virtual Size: 573.9 MB
6865656487bc Virtual Size: 573.9 MB Tags: node:latest
```

Any of the layers would work fine as an image.

However, every layer gets stored in case other future Dockerfiles need them. Even though our current Dockerfile works fine and creates an image that fits our needs, it is not optimized at all.

Note : If you want to know all the details about layers you can take a look [here](http://docs.docker.com/terms/layer/?__hstc=257401556.663ad0b1b8ad942c39bbcec184c0d3f6.1412718746857.1412718746857.1412848572453.2&__hssc=257401556.1.1412848572453&__hsfp=3501542854).

### Chaining commands

This fairly common trick will allow us to trim the number of layers.

If we combine two instructions we avoid making another layer so we’re not storing intermediate (and maybe useless) states. However this could make the Dockerfile less readable.

Here’s our Dockerfile where all the commands have been chained :

```
FROM ubuntu
MAINTAINER Maxime Heckel
 
RUN apt-get update &&
apt-get install -y wget build-essential python python-dev python-pip python-virtualenv &&
wget http://nodejs.org/dist/node-latest.tar.gz &&
tar xvzf node-latest.tar.gz &&
cd node-v* &&
./configure &&
CXX="g++ -Wno-unused-local-typedefs" make &&
CXX="g++ -Wno-unused-local-typedefs" make install &&
echo 'n# Node.jsnexport PATH="node_modules/.bin:$PATH"' >> /root/.bashrc
 
WORKDIR /data
```

This dockerfile will reduce substantially the number of layers, let’s give it a try. First remove the first Node.JS image :

```
docker rmi node
```

and build the new one:

```
docker build -t node .
```

After several minutes of build we finally have our new docker image and if we look at the layer we should see something like this:

```
b18d0a2076a1 Virtual Size: 192.6 MB
67b66f26d423 Virtual Size: 192.7 MB
25c4824a5268 Virtual Size: 192.7 MB
8b1c48305638 Virtual Size: 192.8 MB
c900195dcbf3 Virtual Size: 194.9 MB
6b4e8a7373fe Virtual Size: 194.9 MB Tags: ubuntu:latest
b34e03b4f98d Virtual Size: 194.9 MB
d8c395378857 Virtual Size: 573.9 MB
0276262d8ed2 Virtual Size: 573.9 MB Tags: node:latest
```

The final size of the node image hasn’t changed but if we take a look closer we can see that without chaining commands we had 7 layers between our base (ubuntu) and the node image, now we only have 2 of them.

### Build smart and reusable Dockerfiles

Our last example wasn’t a “smart” Dockerfile. Indeed, remember that each layer is an image that can be used by other Dockerfiles. If we just keep chaining our commands the way we just did, there will be some redundancy between images.

For instance, if, with our current Dockerfile, another image needs all the python packages we just installed, it would have to completely re-install them, whereas if we had a Dockerfile like the following one it wouldn’t have to do so :

```
FROM ubuntu
MAINTAINER Maxime Heckel
 
RUN apt-get update &&
apt-get install -y wget build-essential python python-dev python-pip python-virtualenv
RUN wget http://nodejs.org/dist/node-latest.tar.gz &&
tar xvzf node-latest.tar.gz &&
cd node-v* &&
./configure &&
CXX="g++ -Wno-unused-local-typedefs" make &&
CXX="g++ -Wno-unused-local-typedefs" make install &&
echo 'n# Node.jsnexport PATH="node_modules/.bin:$PATH"' >> /root/.bashrc
 
WORKDIR /data
```

By adding a RUN just before the <code>wget</code> instruction we are creating an extra layer which is the ubuntu base image with all the python packages installed. This layer could be used by other Dockerfiles in the future and will substantially reduce the build process.

Now let’s go even further and add some extra structure to our Docker images. First, delete our current Node Docker image with :

```
docker rmi node
```

Let’s create a folder called Python

```
mkdir Python
```

create a Dockerfile inside:

```
touch Python/Dockerfile
```

paste the following instructions:

```
FROM ubuntu
MAINTAINER Maxime Heckel
 
RUN apt-get update &&
apt-get install -y wget build-essential python python-dev python-pip python-virtualenv
```

and build it:

```
docker build -t python ./Python
```

Now let’s do the same thing but for Node.JS:

```
mkdir Node
```

```
touch Node/Dockerfile
```

paste the following content inside the Dockerfile:

```
FROM python
MAINTAINER Maxime Heckel
 
RUN wget http://nodejs.org/dist/node-latest.tar.gz &&
tar xvzf node-latest.tar.gz &&
cd node-v* &&
./configure &&
CXX="g++ -Wno-unused-local-typedefs" make &&
CXX="g++ -Wno-unused-local-typedefs" make install &&
echo 'n# Node.jsnexport PATH="node_modules/.bin:$PATH"' >> /root/.bashrc
 
WORKDIR /data
```

and build it:

```
docker build -t node ./Node
```

We just split our initial Dockerfile into two seperate Dockerfiles. One contains all of the instructions that install Python from the ubuntu base, and the second one installs Node from the python image.
Nothing is optimized here but this will help us to keep track of our different images.

### Remove useless files

Removing useless files such as .tar.gz or temporary files could also be a way to optimize our Dockerfile. Not only will it help us to have a cleaner image but it can also reduce the size of the final image and save some extra megabytes.

Note: If we look at other Dockerfile examples on the internet, we can see that a lot of people add the following lines:

```
RUN apt-get clean
RUN apt-get purge
```

Those instructions don’t reduce the size of the final image at all so you don’t need to add them.

## Conclusion

The primary goal of this article was to quickly introduce the concept of layers, images and more specifically how to write an optimized and smart Dockerfile using a simple example. Many other tricks can be found in the [Dockerfile best practices](https://docs.docker.com/articles/dockerfile_best-practices/) article from Docker.

---

Original source: [How to Optimize Your Dockerfile](http://blog.tutum.co/2014/10/22/how-to-optimize-your-dockerfile/)