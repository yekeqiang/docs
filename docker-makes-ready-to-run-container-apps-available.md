# Docker让即开即用的容器应用成为可能
## Docker makes ready-to-run container apps available


随着[Docker 1.0](http://www.zdnet.com/docker-1-0-brings-container-technology-to-the-enterprise-7000030333/)的正式发布，[docker](http://www.docker.com/)紧接着就宣布启动为发布应用量身打造的云平台服务，主要包括以下功能：容器镜像的发布，管理权限的变更，用户团队的协作，生产周期流程的自动化，以及第三方服务的集成。

On the heels of Docker 1.0 being released, Docker has announced the launch of cloud-based platform services for distributed applications, including container image distribution and change management, user and team collaboration, life-cycle workflow automation and third-party services integrations.

当然，你并不是一定要使用Docker的“官方”应用。毕竟Docker是一个提供给开发者和系统管理员去构建、传送和运行应用的开放平台。这种容器技术能够让应用程序快速的在容器中完成装配并且从笔记本部署到服务器上去-包括云端。如果你对容器知识一无所知，并且想要分分钟的迅速入门，那么Docker的官方仓库（Docker Official Repository）应该就是你想要的答案。 

Of course, you don't have to use Docker's "Official" apps. Docker is, after all, an open platform for developers and sysadmins to build, ship and run distributed applications. This container technology enables applications to be assembled quickly from components and deployed on anything from laptops to servers — and to the cloud. Still, if you're new to containers and want to move to this technology in a hurry, the Docker Official Repository is just what you need.

正如[451 Research](https://451research.com/)的资深分析员Jay Lyman所说：“企业机构一直在苦苦努力，想让应用程序和开发工作更加轻便，并且能够通过一种更有效、标准、可复用的途径发布应用。就像Github通过分享代码这种方式促进了合作和创新一样，Docker hub的官方仓库和商业支持通过改善部署和发布应用的方式，能够帮助企业很好的解决这个问题。”

As Jay Lyman, senior analyst at 451 Research, said in a statement, "Enterprise organizations are seeking and sometimes struggling to make applications and workloads more portable and distributed in an effective, standardized and repeatable way. Just as GitHub stimulated collaboration and innovation by making source code shareable, Docker Hub, Official Repos and commercial support are helping enterprises answer this challenge by improving the way they package, deploy and manage applications."

准确来讲，Docker Hub有以下优点：

To be exact, here are Docker Hub’s major features. 


- 一个综合的控制台，包含用户、团队、容器、仓库和工作流程的管理。  
An integrated console for managing users, teams, containers, repositories, and workflows.

- Docker Hub Registry提供多于14000个Docker化的应用程序，给用户们在构建他们自己的应用时使用。  
The Docker Hub Registry, offering more than 14,000 "Dockerized" applications available to all users as building blocks for their own applications.

- 协作工具能够让用户们通过公有和私有仓库管理并分享他们的应用程序，并且能在应用生命周期的任意阶段邀请合作伙伴加入一起开发。  
Collaboration tools, enabling users to manage and share their applications through both public and private repositories, and to invite collaborators to participate in any stage of the application life-cycle.


- 只要代码是通过Github或者Atlassian Bitbucket这种基于Git和Mercurial的代码分享服务更新的，不论何时，Automated Builds这个自动构建服务都能够自动重新构建应用程序，并且更新应用的公有和私有仓库，保证应用程序时刻保持最新。
在Docker Hub Registry的14000个Docker化应用中，超过25%的应用程序现在是通过Automated Builds服务创建的，因为Automated Builds服务不仅提供了自动化操作，还提供了源容器的终端用户保障。  
The Automated Build Service, which keeps applications up-to-date by automatically rebuilding and updating an application’s public or private repository whenever the source code is updated on GitHub or Atlassian Bitbucket, a source code sharing service based around Git and Mercurial. Over 25 percent of the more than 14,000 Dockerized applications in the Docker Hub Registry are now created using Automated Builds — which provides both automation and end-user assurance of container origin.

- Webhooks服务让用户能够让重复的工作流程自动化，比如构建管道和连续部署。Webhooks和RESTful API可以共同使用，它让组织机构能够利用任何服务或者软件包的网络接口，比如Github、AWS、或者Jenkins。  
The Webhooks service, which enables users to automate repetitive workflows for build pipelines or continuous deployment. Interoperable with any RESTful API, Webhooks enables organizations to take advantage of the Web APIs published by any service or software package, like GitHub, AWS, or Jenkins.

- Docker Hub的接口包括一个用户授权服务，所以第三方应用和服务可以通过授权访问用户公有或者私有仓库。已经整合到Docker Hub接口的第三方服务包括：AWS Elastic Beanstalk, Deis, Drone.io, Google Compute Engine, Orchard, Rackspace, Red Hat, Tutum, 以及许多其他服务。  
The Docker Hub API includes a user authentication service so that third party applications and services can gain authenticated access to applications in a user’s public and private repositories. Third-party services that have already integrated with the Docker Hub API include AWS Elastic Beanstalk, Deis, Drone.io, Google Compute Engine, Orchard, Rackspace, Red Hat, Tutum, and many others.


大部分的公司和企业最感兴趣的地方是Docker的官方仓库。这里包括了最优化的、被维护的、官方支持的Docker化应用，并且提供给所有的Docker Hub用户使用。现在，仓库里包括Docker Hub Registry中搜索次数前13名的应用程序，包括CentOS, MongoDB, MySQL, Nginx, Redis, Ubuntu, and WordPress。这些程序向所有的组织开放，或者一些愿意根据程序的指导手册提供持续维护的独立软件开发商。

What most companies will find most interesting will be Docker's Official Repositories. These contains "Dockerized" applications that are optimized, maintained, and supported and available to all Docker Hub users. At this time, the Repostories includes the top 13 most-searched-for applications in the Docker Hub Registry. These include CentOS, MongoDB, MySQL, Nginx, Redis, Ubuntu, and WordPress. This program is open to any community group or ISV willing to commit resources to on-going maintenance of an application according to the program’s guidelines.

想把你的程序或者操作系统“Docker化”吗？给partners@docker.com发送一封邮件，用户则可以在[免费Docker Hub账户](http://hub.docker.com/) 注册。  
Interested in getting your application or operating system officially "Dockerized?" Send an e-mail message to partners@docker.com. Users can sign up for free Docker Hub accounts today.













