# Deploying Your Own Private Docker Registry on Azure

---

##### Author: AHMET ALP BALKAN(Software Engineer, Azure Compute)

---

Our contributions to Docker continue at a rapid pace with the addition of a feature that enables the management of your own Docker Registry on Microsoft Azure.

In June, Microsoft Azure added [support](http://msopentech.com/blog/2014/06/09/docker-on-microsoft-azure/) for Docker containers on Linux VMs to ease deployment of Dockerized applications into the cloud and just over two weeks ago we [announced](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/) Windows Server Containers and Azure Support for Docker Open Orchestration APIs. Last week, at TechEd Europe, Mark Russinovich, CTO of Azure, showed a [live demo](http://azure.microsoft.com/blog/2014/10/28/new-docker-coreos-topics-linux-on-azure/) of Docker Client for Windows which we plan to open source very soon.

It is so exciting to see the fast adoption of container technologies and Docker in the community. We are also hearing jaw-dropping news about containers and cool hacks people are developing in the open source world.

The new **Azure Storage Driver for Docker Registry** is the result of our work on the the second [Global Docker Hack Day](https://blog.docker.com/2014/10/announcing-docker-global-hack-day-2/). As Corey Sanders [discussed here](http://azure.microsoft.com/blog/2014/11/03/a-great-week-at-teched-with-docker/) this event coincided with TechEd EU where Docker was featured quite heavily. Ross Gardler also described this feature in his recent roundup of Docker community news relating to Azure on the [Microsoft Open Technologies, Inc. Blog](http://msopentech.com/blog/2014/10/31/docker_community_updates/). However, because the blog entries did not go into any technical detail and because the code was ready to use and available as open source [on GitHub](https://github.com/ahmetalpbalkan/docker-registry-driver-azure) on the day we developed it, I thought it would be good to provide some technical guidance on how to use it.

## What is a “Private Docker Registry”?

In order to ship your containerized applications to the cloud, you use Docker tools to construct a Docker container image. While creating a container image and shipping it to the cloud is easy, it is a challenge to store the generated image reliably. For that reason, Docker offers a service called [Docker Hub](https://hub.docker.com/) to store container images on the cloud and allows you to create containers anytime using those images.

Although the Docker Hub is a paid service for storing private images, Docker respects developers’ needs and provides the open source “[Docker Registry](https://github.com/docker/docker-registry)” software used to build the Docker Hub. It is designed to store and provide container images, but the best part about it is that you can host your own private registry with it. This means, if you wish to host it in your choice of cloud storage provider or on-premise, it is very easy to do that.

## What does that mean for Azure customers?

Until today, there was no plugin support to store the container images on Microsoft Azure Blob Storage. Docker Registry, by default, ships with a local file storage plugin that stores the images on local disk of the virtual machine. This is obviously not the most reliable (or distributable) way to store these binaries and Azure Blob Storage is just the right service for it.

This is where **[Azure Storage Driver for Docker Registry](https://github.com/ahmetalpbalkan/docker-registry-driver-azure)** comes in. It is a plugin that easily allows you to use your Azure Storage Account to host the Docker images you have on Azure Storage as blobs, taking advantage of Azure Storage’s security, reliability, and geographical distribution.

## Setting Up a Private Docker Registry on Azure

If you don’t have a virtual machine on cloud with Docker installed, you can follow [this tutorial](http://msopentech.com/blog/2014/06/09/docker-on-microsoft-azure/) to get a Docker host running on Microsoft Azure Virtual Machines.

The rest is rather easy: On your local development machine, create a new folder somewhere with a file called `Dockerfile` in it with the following contents:

```
FROM registry
RUN pip install docker-registry-driver-azureblob
```

This file will be used to create a new Docker image in which we bundle our Azure Storage plugin on top of Docker’s [registry](https://registry.hub.docker.com/u/library/registry/) image. We build this Dockerfile by running the following commands in this folder:

```
export DOCKER_HOST=tcp://<your-docker-host>.cloudapp.net:4243
docker --tls build --tag=registry-azure .
```

After the image is built, we need to start a container running the Docker Registry server with this image. In order to do that, we just execute the following command:

```
docker run -d -p 5000:5000 \
    -e SETTINGS_FLAVOR=azureblob \
    -e AZURE_STORAGE_ACCOUNT_NAME="<account name>" \
    -e AZURE_STORAGE_ACCOUNT_KEY="<account key>" \
    -e AZURE_STORAGE_CONTAINER=registry \
    registry-azure
```

This starts a container on the cloud listening on port 5000 of the virtual machine and pass the specified configuration to the container as environment variables. However, your new Docker image registry is not yet accessible from outside world.

We need to create an endpoint for the Docker Host Virtual Machine that maps the default Docker internal VM endpoint on port 5000 to an external Cloud Service endpoint on port 80. You can do this either through portal or the Azure Cross-Platform Command Line Interface by running the following:

```
azure vm endpoint create <VMname> 80 5000 -n HTTP
```

After the update finishes, you are now ready to use your private registry. There is a very good video recording by Docker engineer Victor Vieux on [How to Use Your Own Private Registry](https://www.youtube.com/watch?v=CAewZCBT4PI). You can also read `docker-registry` documentation to configure further settings like authentication and caching for your self-hosted Docker Registry.

## Summary

Until today, it was possible to host a Private Docker Registry on a local file system, AWS S3 or Google Cloud Storage with either built-in or community provided storage driver plugins.

Our Docker Hack Day project with Jeff Mendoza, engineer at Microsoft Open Technologies, brought this successful opportunity to enable Azure Blob Storage for hosting Private Docker Registry. Source code for this project is [open source](https://github.com/ahmetalpbalkan/docker-registry-driver-azure) on GitHub and it is written in Python like the rest of the Docker Registry codebase.

We are happy to announce that we will be collaborating with Docker to bring even a higher level of Azure Storage support for the rewrite of the Docker Registry “v2” to ship the Azure Storage Driver as one of the built-in storage drivers.

Stay tuned for more containers and Docker news!

> We are hiring open source enthusiasts and self-initiators who would like to hack on Open Source and Linux in the Azure Team. Please send an email if you are interested in working with us: [ahmetb@microsoft.com](mailto:ahmetb@microsoft.com)

Ahmet Alp Balkan
Engineer, Azure Compute
[@ahmetalpbalkan](http://twitter.com/ahmetalpbalkan)

---

Original source: [Deploying Your Own Private Docker Registry on Azure](http://azure.microsoft.com/blog/2014/11/11/deploying-your-own-private-docker-registry-on-azure/)