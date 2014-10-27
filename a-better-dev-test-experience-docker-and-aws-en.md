# A Better Dev/Test Experience: Docker and AWS

---

##### Author: AWS Activate

---

Here’s a familiar story:

You’ve been tasked with creating the REST API for a mobile app for tracking health and fitness. You code the first endpoint using the development environment on your laptop. After running all the unit tests and seeing that they passed, you check your code into the Git repository and let the QA engineer know that the build is ready for testing. The QA engineer dutifully deploys the most recent build to the test environment, and within the first few minutes of testing discovers that your recently developed REST endpoint is broken.

How can this be? You’ve got thorough code coverage with your unit tests, and all were passing before the handoff to QA. After spending a few hours troubleshooting alongside the QA engineer, you discover that the test environment is using an outdated version of a third-party library, and this is what is causing your REST endpoints to break.

This is an all too common problem in software development. Slight differences between development, test, stage, and production environments can wreak mayhem on an application. Traditional approaches for dealing with this problem, such as change management processes, are too cumbersome for today’s rapid build and deploy cycles. What is needed instead is a way to transfer an environment seamlessly from development to test, eliminating the need for manual and error prone resource provisioning and configuration.

AWS has long offered services that address the need to reliably and efficiently automate the creation of an environment. Services like Amazon EC2 and AWS CloudFormation allow infrastructure to be managed as code. Through the CloudFormation service, AWS resources can be provisioned declaratively using JSON. CloudFormation templates can be versioned right alongside the application code itself. Combined with the automation capabilities of EC2, this allows for a complex environment to be spun up and torn down quickly and reliably. These are just some of the reasons why AWS is such a good choice for development and test workloads.

Container technology, like that being developed by [Docker](http://docker.io/), takes the concept of declarative resource provisioning a step further. Similar to the way CloudFormation can provision an EC2 instance, Docker provides a declarative syntax for creating containers. But Docker containers don’t depend on any specific virtualization platform, nor do they require a separate operating system. A container simply requires a Linux kernel in order to run. This means they can run almost anywhere — be it on a laptop or EC2 instance.

## The Docker container architecture is as follows:

![alt](http://resource.docker.cn/docker-contain-architecture.jpeg)

Docker containers use an execution environment called libcontainer, which is an interface to various Linux kernel isolation features, like namespaces and cgroups. This architecture allows for multiple containers to be run in complete isolation from one another while sharing the same Linux kernel. Because a Docker container instance doesn’t require a dedicated OS, it is much more portable and lightweight than a virtual machine.

## The Docker platform architecture consists of the following:

![alt](http://resource.docker.cn/docker-platform-architecture.jpeg)

A Docker client doesn’t communicate directly with the running containers. Instead, it communicates with the Docker daemon via TCP sockets or REST. The daemon communicates directly with the containers running on the host. The Docker client can either be installed local to the daemon, or on a different host altogether.

There are three key concepts to understand when working with Docker: images, registries, and containers.

An image is the build component of a container. It is a read-only template from which one or more container instances can be launched. Conceptually, it’s similar to an AMI.

Registries are used to store images. Registries can be local or remote. When we launch a container, Docker first searches the local registry for the image. If it’s not found locally, then it searches a public remote registry, called DockerHub. If the image is there, Docker downloads it to the local registry and uses it to launch the container. DockerHub is similar to Github, in that we can create both public and private image repositories. This makes it easy to distribute images efficiently and securely.

Finally, a container is a running instance of an image. Docker uses containers to execute and run the software contained in the image.

You can create Docker images from a running container, similar to the way we create an AMI from an EC2 instance. For example, one could launch a container, install a bunch of software packages using a package manager like APT or yum, and then commit those changes to a new Docker image.

But a more powerful and flexible way of creating images is through something called a DockerFile, which allows images to be defined declaratively. The DockerFile syntax consists of a set of commands that we can use to install and configure the various components that comprise the image. Writing a DockerFile is not at all unlike using UserData to configure an EC2 instance after launch. Like a CloudFormation template, a DockerFile can be tracked and distributed using a version control system. You can think of a DockerFile as the build file for an image.

## So how could Docker help with our mobile health & fitness application example? The application architecture consists of the following components:

![alt](http://resource.docker.cn/application-architecture.jpeg)

First, let’s define a Docker image for launching a container for running the REST endpoint. We can use this to test our code on a laptop, and the QA engineer can use this to test the code in EC2. The REST endpoints are going to be developed using Ruby and the Sinatra framework, so these will need to be installed in the image. The back end will use Amazon DynamoDB. To ensure that the application can be run from both inside and outside AWS, the Docker image will include the DynamoDB local database. Here’s what the DockerFile looks like:

```
FROM ubuntu:14.04
MAINTAINER Nate Slater <slatern@amazon.com>
RUN apt-get update && apt-get install -y curl wget default-jre git
RUN adduser --home /home/sinatra --disabled-password --gecos '' sinatra
RUN adduser sinatra sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER sinatra
RUN curl -sSL https://get.rvm.io | bash -s stable
RUN /bin/bash -l -c "source /home/sinatra/.rvm/scripts/rvm"
RUN /bin/bash -l -c "rvm install 2.1.2"
RUN /bin/bash -l -c "gem install sinatra"
RUN /bin/bash -l -c "gem install thin"
RUN /bin/bash -l -c "gem install aws-sdk"
RUN wget -O /home/sinatra/dynamodb_local.tar.gz https://s3-us-west-2.amazonaws.com/dynamodb-local/dynamodb_local_2013-12-12.tar.gz
RUN tar -C /home/sinatra -xvzf /home/sinatra/dynamodb_local.tar.gz
```

The contents of the DockerFile should be pretty self-explanatory. The RUN keyword is used to execute commands. By default, commands execute as the root user. Since we’re using RVM to install Ruby, we switch to the Sinatra user with the USER keyword, so that the Ruby distribution is installed under the user’s home directory. From the point at which the USER command is specified, all subsequent RUN commands will be executed as the Sinatra user. This also means that when the container is launched, it will execute commands as the Sinatra user.

The Docker daemon is responsible for managing images and running containers, and the Docker client is used to issue commands to the daemon. So to build our image from the above DockerFile, we’ll execute this client command:

```
$ docker build --tag=”aws_activate/sinatra:v1" .
```

Full documentation of the Docker client commands can be found on the [docker.io](http://docker.io/) website. But let’s take a closer look at the command used to build our image. The tag option sets an identifier on the image. The typical value for the tag option is **owner/repository:version**. This makes it easy to identify what an image contains and who owns it when viewing the images in a registry.

After we execute the **build** command, we should have an image configured using the declarations in the DockerFile:

```
$ docker images
REPOSITORY                  TAG            IMAGE ID       CREATED                     VIRTUAL SIZE
aws_activate/sinatra        v1             84b6d4a5a22b   
36 hours ago                942.2 MB
ubuntu                      14.04          96864a7d2df3    
6 days ago                  205.1 MB
```

Sure enough, we can see that Docker created our image, assigned it the tag we specified on the command line, as well as a unique image ID. Now let’s launch a container from this newly created image:

```
$ docker run -it aws_activate/sinatra:v1 /bin/bash
```

This command launches the container and drops us into a bash shell. From here, we can interact with the container just as we would a Linux server. Since we’re developing a web application, we’ll clone our latest version into the container from the Git repository, run our unit tests, and get ready to hand it off to QA. Once the code has been cloned into the container and is ready for testing, we’ll commit our changes in the running container to a new image. To do this, we need to determine the container ID:

```
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
b9d03d60ba89 aws_activate/sinatra:v1 "/bin/bash" 11 minutes ago Up 11 minutes nostalgic_davinci
```

Next, we run the commit command:

```
$ docker commit -m “ready for testing” b9d03d60ba89 aws_activate/sinatra:v1.1
```

Now we have a new container in our local registry:

```
$ docker images
REPOSITORY                 TAG               IMAGE ID 
CREATED                    VIRTUAL SIZE
aws_activate/sinatra       v1.1           40355be9eb8f 
21 hours ago               947.5 MB
aws_activate/sinatra       v1             84b6d4a5a22b 
3 days ago                 942.2 MB
ubuntu                     14.04          96864a7d2df3 
8 days ago                 205.1 MB
```

Version 1.1 of our container includes the Sinatra application that will serve up our REST endpoint. We can run this web application as follows:

```
$ docker run -d -w /home/sinatra -p 10001:4567 aws_activate/sinatra:v1.1 ./run_app.sh
```

This tells Docker to do the following:

1. Create a container from image aws_activate/sinatra:v1.1

2. Run the container in detached mode (-d)

3. Set the working directory to be /home/sinatra (-w)

4. Map the container port of 4567 to the host port of 10001 (-p)

5. Execute a shell script called run_app.sh in the container

The shell script starts up the local DynamoDB database in the container and launches the Sinatra application using the Thin webserver on port 4567. Now if we point our browser on the laptop running this Docker container to [http://localhost:10001/activity/1](http://localhost:10001/activity/1), we should see the following:

```
{"activity_id":"1",
"user_id":" db430d35-92a0-49d6-ba79-0f37ea1b35f7",
"type":"meal",
"calories":100,
"date":"2014-09-26 15:33:58 +0000"}
```

Our endpoint seems to be working properly — the activity record was pulled from the local DynamoDB database and returned as JSON from the Sinatra application code.

To make this container available to the QA engineer for further testing, we can push it into DockerHub, the public registry. Similar to Github, DockerHub offers both public and private options if we don’t want to make this container available to the general public.

The QA engineer will be running this container in EC2, which means we’ll need an EC2 instance configured with the Docker daemon and client software. Assuming we’re going to bring up the EC2 instance and the DynamoDB table using CloudFormation, we can bootstrap in the Docker software installation using the UserData property of the CloudFormation AWS::EC2::Instance type. Here’s what the JSON for provisioning the EC2 instance in CloudFormation looks like:

```
"DockerInstance": {
     "Type": "AWS::EC2::Instance",
     "Properties": {
          "InstanceType": "t2.micro",
          "ImageId": {"Fn::FindInMap" : ["RegionMap",{"Ref" :
          "AWS::Region"}, "64"]},
 "KeyName": {"Ref": "KeyName"},
 "SubnetId": {"Ref": "SubnetId"},
 "SecurityGroupIds": [{"Ref": "SecurityGroupId"}], 
 "Tags": [{"Key": "Name", "Value": "DockerHost"}],
 "UserData": {"Fn::Base64":
"#include https://get.docker.io"}
 }}
```

Now when the QA engineer logs into the EC2 instance created by the CloudFormation stack, the image can be pulled from the remote DockerHub registry:

```
$ docker pull aws_activate/sinatra:v1.1
```

The command used to start the container from this image is virtually identical to the one shown above. The one difference is that an environment variable will be set using the “-e” option to start the Sinatra application using the “test” environment configuration. This configuration will use the regional endpoint for connecting to DynamoDB, instead of the local endpoint:

```
$ docker run -d -w /home/sinatra –e “RACK_ENV=test” -p 10001:4567 aws_activate/sinatra:v1.1 ./run_app.sh
```

Now the QA engineer can access the REST endpoint over HTTP using the public DNS name of the EC2 instance and port number 10001 (this requires a security group rule that allows ingress on port 10001). If any bugs are found, the running container can be committed to a new image, tagged with an appropriate version number, and pushed to the registry. The state of the container will be completely preserved, making it easier for us (the software developers) to reproduce any issues found in QA, examine log files, and generally troubleshoot the problem.

We hope this has been a good introduction to Docker, and that you’ve seen how Docker and AWS are a great combination. The portability of Docker containers makes them an excellent choice for dev and test workloads, because we can so easily share the containers across teams. EC2 and CloudFormation make a great combination for running containers in AWS, but the story doesn’t end there. Other services, like AWS ElasticBeanstalk, include support for deploying entire application stacks into Docker containers. Be sure to check out this and the other AWS blogs for more information about running Docker in AWS!

#### *Nate Slater*

*Solution Architect*

---

Original source: [A Better Dev/Test Experience: Docker and AWS](https://medium.com/aws-activate-startup-blog/a-better-dev-test-experience-docker-and-aws-291da5ab1238)