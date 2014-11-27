# Your First Docker Image & Container (Ubuntu + NodeJS)

---

Author: Ryan Pploetz 

---

Docker has been getting a lot of development press lately and I have been interested in checking the containers out. I’ve done some reading, watched some videos, and even saw some of the Docker engineers give demos at AWS Re:Invent 2014 this year. The Docker sessions at AWS Re:Invent this year were always packed with long lines to get into the sessions. Docker was the tool to checkout.

I am learning more about Docker and working on getting into my production workloads at work, but I wanted to share some basic Docker information and help everyone get setup with a simple Docker running Ubuntu 14.04 with NodeJS.

If you are on a Mac, you will need to get [Boot2Docker](http://boot2docker.io/) or run a [Vagrant](https://www.vagrantup.com/) Image with Linux and Docker installed. I found Boot2Docker to be easier.

I have posted the code for this tutorial on my GitHub: [https://github.com/Zephadias/NodeJSDockerExample](https://github.com/Zephadias/NodeJSDockerExample) 

So feel free to fork or clone to follow along.

Lets first make a new folder to hold our NodeJS source code and make the necessary files.

```
mkdir src
cd src
touch index.js
touch package.json
```

Let’s edit our package.json file to setup our Node project. I won’t go into details on the Node  since this is a Docker tutorial.

```
{
  "name": "docker-ubuntu-node-example",
  "private": true,
  "version": "0.0.1",
  "description": "Node.js Hello World on Ubuntu using Docker",
  "author": "Ryan Ploetz <ryan.ploetz@gmail.com",
  "dependencies": {
    "express": "4.x.x"
  }
}
```

Next let’s edit out index.js file so we have a Node application ready for deployment. All this is doing is sending a Hello World to anyone that connects to the Node application on port 8080.

```
var express = require('express');
 
var PORT = process.env.PORT || 8080;
 
var app = express();
app.get('/', function( req, res) {
  res.send('Hello World');
});
 
app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
```

Let’s now get into Docker.

In your root directory we need to go make a file called Dockerfile. This is going to describe what we want our container to do when we build it. Let’s start editing it.

```
FROM ubuntu:14.04
```

The first Docker command we are going to look at is `FROM`. This tells the container what operating system we want to use and after the : tells Docker specifically what tag or version we want to use. This can be any image from the [Public Repositories](http://docs.docker.com/userguide/dockerrepos/#using-public-repositories). The FROM must be the first non-comment line inside a valid Dockerfile. If a tag isn’t specified then the latest image is assumed.

The next Docker command is MAINTAINER. This is the author of the Dockerfile.

```
MAINTAINER Ryan Ploetz <ryan.ploetz@gmail.com>
```

Our next command is RUN. This allows us to run commands on the Docker image during the build and there are two forms and we will look at the RUN command that runs in a shell. Here we are doing an update and an install to get nodejs and npm installed.

```
RUN apt-get update
RUN apt-get install -y nodejs npm
```

Next we are going to copy our src folder that contains our code to the the local image.

```
COPY ./src /src
```

Next we want to install our dependencies found in our package.json file so we first cd into the directory and run our npm install.

```
RUN cd /src; npm install
```

`EXPOSE` tells the container to listen on a specified network port at runtime. Since we specified 8080 in our NodeJS application, we will expose that to our container.

```
EXPOSE 8080
```

Our last command is `CMD`.

The main purpose of a `CMD` is to provide defaults for an executing container.

That is exactly what we are doing with our nodejs executable and passing it our index.js file.

```
CMD ["nodejs", "/src/index.js"]
```

That’s it from a Dockerfile sense, now we are ready to build our image. You do this by using the build command and can easily tag it with a -t. The period at the end of the line is important, this says to look for the Dockerfile in the current directory that you are in.

```
docker build -t <your username>/ubuntu-nodejs .
```

Now you have your image built, next thing to do is to run your container. We are going to use the run command with the -p option to publish the network port and use a high port and publish our internal 8080 to it. The preference is to use a large port number so you can run multiple containers without them all conflicting to 80 or 8080. The -d command says to use what image. When you run this, docker will return your long container UUID letting you know that it is running.

```
docker run -p 49160:8080 -d <your username>/ubuntu-nodejs
```

Let’s see what containers are running…

```
docker ps
```

This will print out the containers you have running, notice the NAMES column. If you don’t specify a name, docker will make one up for you. You could have used a –name to specify you’re own name if you wanted.

So now we have a container running, how do we connect to it? If you are on Linux, you can connect to it by localhost:49160. If you on a Mac using Boot2Docker, then you need to get the forwarded IP address by the image. You can get this by calling:

```
boot2docker ip
```

Now you can type that IP:49160 in your browser and see it running!

The last thing I will leave you with is how to stop your container so you don’t have it continuously running. It is rather simple as well…

```
docker stop <container name>
```

Remember the name here is the docker name that was made up for you, not your image name.

So that is your initial Docker tutorial. We are excited to read and learn more about Docker and how to use it in a production environment. Come back for our next blog post later!

---

Original source: [Your First Docker Image & Container (Ubuntu + NodeJS)](http://www.fireplacecoders.com/?p=15)