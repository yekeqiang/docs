# How We Use Docker To Run And Deploy devo.ps

---

##### Author: Vincent Viallet

---

devo.ps is a complex system with a lot of moving pieces both at the code and infrastruture levels. We're deploying major releases at least once a week, and dozens of micro-releases in between for minor improvements. Having a simple yet reliable deployment workflow is essential for this to even be possible.

![alt](http://resource.docker.cn/devops-architecture.png)

## Infrastructure

The devo.ps infrastructure is composed of 8 different components, each associated with a micro-service (API, Git, registry, ...) and each of them sitting in their own Docker container. This allows us to update them individually, pushing security updates when and where needed, easily reverting changes and more generally scale horizontally.

To orchestrate those containers, we opted for [maestro-ng](https://github.com/signalfuse/maestro-ng) over [fig](http://www.fig.sh/). This allows us to easily manage the containers across several hosts, enforce the start order, mount volumes for data persistency and fetch and upgrade containers when new versions are available in our docker registry.

Host discovery and DNS are handled by [Consul](http://www.consul.io/), with a cluster of consul hosts powered by [Progrium's awesome build](https://registry.hub.docker.com/u/progrium/consul/) and agents running in the various containers. We expose the DNS on the docker bridge (`172.17.42.1`) allowing containers to reference this "permanently available" IP address in their config. One (small) problem remains though; referencing that IP address in the `resolv.conf` of an underlying docker hosts makes docker change the IP address of the bridge on start...

Each of the container then lives as a micro-service, performing tasks and handling requests independently. We can deploy releases on any of these when needed.

## Code

We use AngularJS, Node.js, python and various other scripts to glue things together. The code base is split across multiple repos on GitHub to allow devs to work independently and get things moving fast and in parallel (which is why we built the multi-repos support [SweepBoard](http://sweepboard.com/) in the first place).

## Development cycle

To avoid conflicts/merging nightmares, features are developed independently on dedicated branches. New code is continuously deployed and heavily tested on dev environments before making their way back in the master branches. These are then tested on staging before being released on production.

Upon deployment, each repo is tagged with the version number before being finally pushed on the production environment.

## Code deployment workflow

Our containers are running as micro-services and do not have to be taken down when a new release is deployed. Only the running code needs to be "refreshed" and the various inner-services updated.

The deployment workflow is almost the same on either dev / staging or production platform and rely heavily on [supervisord](http://supervisord.org/).

Our code repositories are usually mounted in each of the containers as read-only volumes, making the latest code available at all time. A deployment script is then defined as a command in a supervisord configuration file and can be fired at any time to release the code.

We're taking the exact logic described in a previous post about using [supervisor for deployment pipelines](http://devo.ps/blog/supervisord-for-deploy-pipelines) and can deploy in no time any release on either of our platforms.

This allows us to build, push and release as frequently as needed with minimal manual intervention and yet the senerity of a reproductible deployment on any platform. This also allows us to reduce downtime to a matter of seconds.

Oh and by the way, we're using devo.ps to deploy devo.ps !

---

Original source: [How We Use Docker To Run And Deploy devo.ps](http://devo.ps/blog/how-we-use-docker-to-run-and-deploy-devops/)