# Say Yes to Docker!


---

Authour: Jonathan Chauncey

---

If you haven't heard of [Docker](http://www.docker.com/) I'm going to assume you've been living under a rock for the last six months, because it's pretty much everywhere. My team started exploring Docker to help us with a variety of problems a few months ago and now we're not looking back. In fact, we're rapidly adopting Docker as our main deployment architecture for all production deploys. I want to share with you some of the main reasons we like Docker and the problems it's solving for us.

## The Unified Deployment Artifact

We have teams exploring all types of technologies and languages. This means our deployment tools are often playing catch up to support these frameworks, adding tons of complexity along the way. Furthermore, our teams, which support these services in a 24x7 capacity, also had to keep up with the intricacies of how they were deployed.

With Docker we're encouraging teams to generate Docker images as their build artifact, and upload that Docker image to our registry (quay.io). This means the deployment tool we've built only needs to know how to pull an image from the registry and then start that image on a machine -- all of which are just API calls to the Docker host we're using for deployment. (More to come on our deployment tool for Docker =)

## Consistency

We use Chef to manage the state of our machines. It provides us with a mechanism for managing configuration drift -- to a degree. While it will manage the state of things it installs, Chef does not manage the state of things that we install by hand (which happens.) With Docker, an image that's created during our build step never changes between the moment it's created and when it's deployed; it's always consistent. Users cannot log into a running container and install things by hand or change the state from the outside.

## Remember the "Build Once, Run Anywhere" That Java Promised?

Yeah. Well, Docker gives us this ability without having to adopt another programming language (we're a Java shop but you get what I mean.) We can bundle any application we create as a Docker image and run on it on any host that has Docker installed. Our teams no longer have to worry if the correct version of Ruby or Java is installed on a machine for their application to run. They perform the necessary operations to install that system dependency in their Dockerfile and no longer have to worry about that part of the equation.

## It's Fast!

Docker containers start almost instantly. This means our deployment tool only needs to wait for the application to start and the health check to pass (which happens almost at the same time) before moving on to the next host in the deployment. For most of our production deployments, the time for a commit to be deployed to all hosts is around five minutes, and most of that is wrapped up in our commit tests (which we could optimize even further.) This is a huge difference from teams using Chef to deploy, where our more complicated pipelines can take as long as 35 to 40 minutes.

## A/B Deployments Are Now Possible

A true zero downtime deployment is the unicorn of deployment strategies. Teams can push out a change knowing that users in the system are not affected by the deployment. Right now at Rally we do serial rolling deploys, which take down one host at a time. We upgrade that host with the new version of the application (or container in Docker) and wait for the health check to succeed. (This means users on that node when the application is taken down could see weird behavior, but it prevents having to support a parallel stack and then switching over the connections to the new system.)

Docker makes moving to the zero downtime deployment strategy easier by allowing us to run multiple versions of the same application on one host. We can then tell the load balancer to move traffic to the newly deployed set of containers when we're ready, which will bleed off any remaining connections.

## Use It for Self-service and Automation Tooling

You know that tool that only works on someone's machine because they painstakingly took hours to get it setup just right? Docker is perfect for those kinds of applications. Create an image that contains that environment and the next time you need to use it all you have to do is just do a docker run my-awesome-tool.

So what are some other ideas for automation or self-service containers?

- Compiling something from source
 
- Allowing developers to run a job or report against another system

- Building custom packages

It never ceases to amaze me the ways in which people use Docker in these kinds of situations.

## Coming up ...

So I've talked a bit about Docker and some of the problems it's helping us solve; over the next few weeks I'll talk about how we're moving forward with Docker in our production stack as our main deployment architecture -- including a post on our custom deployment tool, Armada, which I think provides a low barrier to entry for teams looking to use Docker in production.

---

Original source: [Say Yes to Docker!](https://www.rallydev.com/community/engineering/say-yes-docker)