# PeopleSoft and Docker's value proposition

---

Author: Javier Delgado 

---


If you haven't heard yet about [Docker](http://docker.com/) and/or container technologies, you will soon do. Docker has made one of the biggest impacts in the IT industry in 2014. Since the [release](http://www.businesswire.com/news/home/20140609005457/en#.VGjq_Yc03pZ) of its 1.0 version on past June, it has captured the attention of many big IT vendors, including Google, Microsoft and Amazon. As far as I'm aware, Oracle has not announced any initiative with Docker, except for the [Oracle Linux container](https://docs.docker.com/installation/oracle/#to-enable-the-addons-repository-via-oracle-public-yum). Still, Docker can be used with PeopleSoft, and it can actually simplify your **PeopleSoft system administration**. Let's see how.

## What is Container Technology?

Docker is an open platform to build, ship, and run distributed applications. Docker enables apps to be quickly assembled from components and eliminates the friction between development, QA, and production environments. As a result, IT can ship faster and run the same app, unchanged, on laptops, data center VMs, and any cloud.

In a way, it is similar to virtualization technologies like VMWare or Virtualbox where you can get an image of a machine and run it anywhere you have the player installed. Docker is similar except that it just virtualizes the application and its dependencies, not the full machine.

Docker *virtual machines* are called **containers**. They run as an isolated process in userspace on the host operating system, sharing the kernel with other containers. Thus, it enjoys the resource isolation and allocation benefits of VMs but is much more portable and efficient.

Docker uses a layered file system for its containers, in a way that they can be updated by just including the changes since the last update. This greatly reduces the volume of information that needs to be shipped to deliver an update.

## How can it be used with PeopleSoft?


As we have seen, Docker containers are much easier to deploy than an entire virtual machine. This means that activities such as **installations** can be greatly simplified. All you need is to have Docker installed and then download the PeopleSoft container. Of course, this requires that you first do an installation within a Docker container, but this is not more complex than doing an usual installation, it just requires some Docker knowledge in order to take advantage of all its features. Under my point of view, if you are doing a new installation, you should seriously consider Docker. At [BNB](http://www.bnetbuilders.com/) we have prepared containers with the latest PeopleSoft HCM and FSCM installations so we can quickly deploy them to our customers.

Also, when you make a change to a Docker container, just the incremental changes are applied to existing running instances. This poses a great advantage when you apply a patch or run a PeopleTools upgrade. If you want to apply the patches to a new environments, you just need to make sure that you apply the latest container changes in all the servers running the environment.

**Isolation** between running instances is also a major advantage when you have multiple environments in the same server. Suppose you want to apply the later Tuxedo patch just in the Development environment, which coexists with other environments on the same server. Unless you had one Tuxedo installation for each environment (which is possible but normally unlikely), you would need to go ahead and hope the patch did not break anything (to be honest, this happens very rarely with Tuxedo, but some other product patches are not so reliable). If you have a separate container for the Development environment you can apply the patch just to it and later deploy the changes to the rest of environments.

Last but not least, the reduced size of Docker containers compared to an entire virtual machine greatly simplifies the distribution to and from the cloud. Docker is of great help if you want to move your on premise infrastructure to the cloud (or the other way around). This is even applicable when you want to keep a contingency system in the cloud, as delivering the incremental container changes made to your on premise system requires less time than using other methods.

Not only that, Docker can be hosted in most operating systems. This means that moving a container from one public cloud facility to another is significantly easier than it was with previous technologies. Exporting a virtual machine from Amazon EC2 to Google Cloud was quite complex (and under some circumstances even not possible).


## Limitations

But as any other technology, Docker is no panacea. It has some limitations that may restrict its adoption for your PeopleSoft installation. The main ones I can think of are:

- Currently there is no support for containers using Windows as a guest operating system. This is not surprising, as Docker in intimately linked to Unix/Linux capabilities. Still, Microsoft has [announced](http://www.zdnet.com/docker-container-support-coming-to-microsofts-next-windows-server-release-7000034708/) a partnership with Docker that will hopefully help to overcome this limitation. For the moment, you will not be able to use Docker for certain PeopleSoft components, such as the PSNT Process Scheduler, which is bad news if you are [still using Crystal Reports](http://javier-ps.blogspot.com.es/2014/10/peopletools-854-will-be-last-release-to.html) or Winword reports. Also, if you are using Microsoft SQL Server as your database, this may be a major limitation.


- Docker is most useful when used for applications, but not data. Logs, traces and databases should normally be kept out of the Docker container.


## Conclusions

Although container technology is still in its initial steps, significant benefits are evident for maintaining and deploying applications, **PeopleSoft** included. Surely enough, the innovation coming on this area will have a big impact in the way PeopleSoft systems are administered.

*PS: I would like to thank [Nicolás Zocco](https://plus.google.com/114042368314704132016/about) for his invaluable research on this topic, particularly in installing the proof of concept using PeopleSoft and Docker.*

---

Original source: [PeopleSoft and Docker's value proposition](http://javier-ps.blogspot.it/2014/11/peoplesoft-and-dockers-value-proposition.html)