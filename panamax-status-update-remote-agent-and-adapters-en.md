# Panamax Status Update - Remote Agent and Adapters 

---

##### Author: [Ross Jimenez](http://www.centurylinklabs.com/author/rossjimenez/)

---

![alt](http://resource.docker.cn/pmx-ship.png)

First off, thank you! We have had almost 5,000 downloads of [Panamax](http://panamax.io/?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1415163435198.1415166223189.3&__hssc=257401556.1.1415166223189&__hsfp=2081402262), some great feedback, and over 600 stars on our primary Github repository. The entire Panamax team has had a great time over the last couple of months releasing the Panamax project and getting to know the user community more. We are still just getting started but I thought now would be a good time to discuss some of our plans for the future.

## THE PANAMAX VISION

Our Mantra “Docker Management for Humans” will continue to stay front and center in everything we do. We will continue to strive to make it easy to leverage Docker and the Docker ecosystem. We will also continue to focus on providing a tool that is transparent as possible rather than abstracting the technology for the sake of simplicity. This is why we try to consistently expose what Panamax is doing behind the scenes with Docker itself vs. abstracting it away as a PaaS would. We don’t want to be a black box, but rather a simple easy to use tool that makes the complexity of Docker less intimidating.

## PANAMAX TODAY

Today, Panamax is a multi-container Docker application itself. Panamax assumes that it is installed on a CoreOS host and uses Fleet and etcd in order to deploy and run multi-container application templates that can be created from within Panamax. Today, if you want to run an application somewhere other then your local laptop, you need to install Panamax and use the UI running from the desired environment. This will accomplish the goal of running your application somewhere else, but it really isn’t practical to have the entire Panamax application on remote infrastructure.

In the spirit of agile development, today’s development and deployment model is nothing more than our MVP. Before we knew the utility Panamax provided to users, and how the community wanted the software to evolve, supporting single-host deployment was very reasonable architecture and allowed us to introduce the UX and the important idea of Panamax templates. But our team and our users are ready for more.

## THE COURSE AHEAD

Given the ever-expanding Docker ecosystem we have quickly realized that going forward, we have a new goal:

> **Panamax Templates should be agnostic to host, orchestration, and cluster management technology**.

The idea of a Panamax template is simple: a user should be able to assemble all important and useful information about a particular application in one place, and using the template, be confident that their application will run the same time, every time. You can stitch together Docker containers easily into an application, then immediately run it locally and validate that the containers are working together as expected.

In the past, these templates could only run in the same environment where Panamax was deployed — CoreOS with Fleet and etcd. But what about multi-host architecture? Schedulers? Cluster managers? Shouldn’t users be able to have Panamax help them with deploying to these complex environments too? In the wild, Docker applications can vary tremendously based on the infrastructure host, cluster management, or orchestration technologies installed in the target environment. Our next major release will allow Panamax to work seamlessly with whatever infrastructure technologies you choose.

## PANAMAX REMOTE AGENT AND ADAPTERS

![alt](http://resource.docker.cn/pmx-arch.png)

pmx arch Panamax Status Update – Remote Agent and Adapters

In order for Panamax templates to be agnostic to these variables, we are introducing a new deployment process and a pluggable microservice-based architecture that will allow a Panamax template to be deployed to a remote infrastructure.

Panamax Adapters have one job: take a Panamax template and deploy the application represented within it to the remote target environment. Adapters will be specific to the remote host and a specific Orchestrator/Scheduler and Cluster Management technologies given this expect multiple adapters to be developed by the Panamax core team as well as others. We also recognize that Panamax templates don’t always directly translate to these other technologies, there is no way a tool like Panamax can replicate every option locally but the Adapter model will allow a decoupled architecture that will provide the greatest level of choice for ours users vs. adopting one specific technology.

Adapters will also work with the Panamax Agent, which is responsible for security and interaction with the Panamax client. The security model being put in place will allow owners of a Panamax Agent to share a token with another Panamax Client so that multiple Panamax UI users can deploy Panamax Templates to a shared remote infrastructure environment.

Our goal in creating this architecture was to make Adapter development as easy as possible by limiting what an Adapter has to accomplish. We also didn’t want to dictate technology of the Adapters so, just as any container, they can be created in any language of choice by developers and only need to worry about interfacing with the Panamax Agent.

Now you should have a better basic understanding of the course we are charting for Panamax. Keep an eye out on the [Panamax Wiki](http://j.mp/pmx_wiki?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1415163435198.1415166223189.3&__hssc=257401556.1.1415166223189&__hsfp=2081402262) where we will soon release some detailed technical documentation on the Agent/Adapter architecture and a development guide for creating new Adapters.