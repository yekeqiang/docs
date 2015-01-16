# Docker at Shopify: How we built containers that power over 100,000 online shops

---

Author: Graeme Johnson 

---


![alt](http://resource.docker.cn/astroids.jpg)

This is the second in a series of blog posts describing our evolution of Shopify toward a  Docker-powered, containerized data center. This instalment will focus on the creation of the container used in our production environment when you visit a Shopify storefront.

Read the [first post]((http://www.shopify.com/technology/15563928-building-an-internal-cloud-with-docker-and-coreos) ) in this series.

## Why containerize?

Before we dive into the mechanics of building containers, let's discuss motivation. Containers have the potential to do for the datacenter what consoles did for gaming. In the early days of PC gaming, each game typically required video or sound driver massaging before you got to play. Gaming consoles however, offered a different experience:

- **predictability**: cartridges were self-contained fun: always ready-to-run, with no downloads or updates.
- **fast**: cartridges used read-only memory for lightning fast speeds.
- **easy**: cartridges were robust and largely child-proof - they were quite literally plug-and-play.

Predictable, fast, and easy are all good things at scale. Docker containers provide the building blocks to make our data centers easier to run and more adaptable by placing applications into self-contained, ready-to-run units much like cartridges did for console games.

## Bootstrapping

To pull off a containerization effort you need a mix of development and operations skills. First, talk to your operations team, as you'll need to ensure that your containers faithfully duplicate your production environment.

If you're running an OSX (or Windows) desktop and deploy on Linux, use a virtual machine such as Vagrant as a local test environment. As a first step, get your operating system and supporting packages installed. Pick a base image that matches your production environment (we use Ubuntu 14.04) and resist any urge to deviate - you don't want to be fighting containerization and operating system/package upgrades at the same time.

## Choose a container style

Docker affords you ample room in choosing a container style, from a 'thin' single-process container, to a 'fat' container (for example, [Phusion](https://github.com/phusion/baseimage-docker)) which feels much like a traditional virtual machine.

We chose to follow the 'thin' container path and eliminate all extraneous machinery from inside the containers. This is the harder of the two choices, but yields smaller, simpler containers that burn less CPU and memory. This approach is elaborated on on the [Docker blog](http://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/).

## Environment setup

We use Chef to manage our production nodes. While we could simply have run Chef inside a container, this pulled in services (e.g. log indexing and stats collection) that we didn't want duplicated into every container. Rather than tolerating duplication, we challenged ourselves to share a single copy of these services on each Docker host machine.

The path to 'thin' containers required transliterating our Chef recipes into a Dockerfile (which we later replaced with a custom build process - but that’s another post). This is a golden opportunity to take stock of your production environment and review and document what's really needed (some archaeology may be required). Be ruthless with removing as much as possible, and be sure to code review heavily at this stage.

This process was less painful than it may sound. We wound up with a heavily commented 125-line Dockerfile that defines the 'base' image shared by all containers at Shopify. This base image includes 25 packages which span language runtimes (Ruby, Python, Node), development tools (Git, Vim, build-essential, Go) and commonly used libraries. The base image also includes a handful of utility scripts for tasks such as launching Ruby with tuning parameters, or sending events to Datadog.

Applications are free to add their specific needs to this base image. Even then, our largest application only adds another couple of operating system packages, so our base image has remained fairly lean.

## The 100 rule

When choosing what services to put into a container, imagine that you have 100 small containers running on a single host. Then, ask yourself if you really need 100 copies of a given service running, or if it's better to invest the effort to share a single host-provided copy.

Here are a few practical examples where the 100 rule influenced the shape of our containers:

- **Log Indexing** : Logs are critical to diagnose production problems quickly, and are even more important in a containerized world when your filesystem disappears at container exit. We purposely avoided changing the logging behaviour of applications (e.g. forcing them to redirecting to syslog) and allowed them to continue logging to the filesystem.  Running 100 log relay agents seemed wrong, so we built a daemon to handle some key tasks:
	- Runs on the host-side and subscribes to Docker events.
	- On container start, configures the log indexer to watch the container.
	- On container destroy, removes the indexing instructions.

	Take note that you’ll want to delay destruction of the container for some time after it exits to ensure all logs are indexed.

- **Statistics**: Shopify produces runtime statistics at several levels: system, middleware, and directly from the application. Statistics are typically relayed either via an agent or sent directly from application code.
	- Many of our statistics are transported via StatsD, and we were fortunately able to configure our Datadog host-side collector to accept traffic from containers (e.g. [Network traffic and proxy configuration](https://github.com/DataDog/dd-agent/wiki/Network-Traffic-and-Proxy-Configuration)). With this configuration in place, just pass the address of the StatsD relay into the container.
	- A host-side system monitor agent will be able to see across container boundaries, as containers are ultimately just a process tree. Sharing a single system monitor is therefore essentially free.
	- For a more container-centric view, consider Datadog’s Docker integration which adds Docker metrics to the host-side monitoring agent.
	- Application-level metrics for the most part ‘just worked’ for us as they tend to either emit events via StatsD, or will talk directly to another service. It’s important to specify a container name so that metrics appear with a sensible name.

- **Kafka**: We use Kafka as an event bus to publish real-time events from across the Shopify stack to interested parties. We publish Kafka events from our Ruby on Rails code by building messages and placing them onto a SysV message queue. A simple daemon written in Go empties the queue and sends messages to Kafka. This arrangement minimizes Ruby processing time, and allows us to gracefully handle a Kafka server outage. Unfortunately, SysV message queues are part of the IPC namespace, and as a result we couldn’t use a queue for container:host connections. We solved this issue by adding a socket endpoint on the host for placing messages onto a SysV queue. Once again, we pass the address of the endpoint to the container via the environment.  You can find more detail in a separate post [here](http://www.shopify.ca/technology/14909841-kafka-producer-pipeline-for-ruby-on-rails).

The 100 rule requires some ingenuity. In some cases, it’s just a matter of writing some glue between components. Other times it can be achieved through configuration, and sometimes redesign is required to achieve this goal. At the end of this process, you should have a container that has all of the pieces required to run your application, and a mildly modified host environment that provides Docker hosting and shared services.

## Containerizing your application

With the environment set up, we can turn our attention to containerizing the application itself.

At Shopify we prefer 'thin' containers that do exactly one thing, such as a unicorn master and workers for serving web requests, or a Resque worker servicing a particular queue. Thin containers allow fine-grained scaling to match demand. For example, we can increase the number of Resque workers checking for spam in response to an attack.

We've found it useful to adopt some standard conventions for code layout within containers:

- Applications are always rooted at `/app` inside the container
- Applications currently expose services on a single port

We have also established some conventions for git repos that are containerized:

- `/container/files` holds a tree of files that are copied directly into the container when it is built. For example, to request Splunk indexing of an application log it suffices to add `/container/files/etc/splunk.d/inputs.conf` file to your git repo. Note: this is a subtle but major shift in responsibility - developers are now in control of log indexing, which was previously within the ops domain.
- `/container/compile` is a shell script that compiles your app and produces a 'ready to run' container. Building this file and adapting your application is where the bulk of complexity lies.
- `/container/roles.json` holds the command-lines used to run the workload in a machine-readable form. Many of our applications run the same codebase in multiple roles - some handle web traffic while process background tasks. This was partially inspired by Heroku's [procfile](https://devcenter.heroku.com/articles/procfile). 

Here’s an example `roles.json` file:

```
{
    'web': 'unicorn -c /app/unicorn.conf.rb -l 8000 -E $RAILS_ENV',
    'resque': 'bundle exec rake environment resque:work',
    '.shell': '/bin/bash -l',
  }
```

We drive builds with a simple Makefile which can also be run locally. Our Dockerfile looks something like this:


```
# Copy all the static files into the container
ADD ./container/files /
 
# Copy the application source the /app directory
ADD . /app
 
# Produce a ready-to-run container
EXEC /app/container/compile
```

Recall the goal of the compile phase is to produce a container which is **ready-to-run instantly**. One of the key Docker advantages is super-fast startup which you don't want to erode by doing extra work at container boot. In order to achieve this goal you'll need to understand your entire deploy process. A few examples:

- We are using Capistrano to deploy code to machines, and asset compilation previously happened as part of deployment. By moving asset compilation into container build, deploying new code is simpler and faster by several minutes.
- Our unicorn master startup interrogated the database for table shapes. Not only was this slow, but our smaller container size meant a lot more database connections would be required. Again, it's possible to do this at container build time to speed startup.

In our case, the compile phase includes the following logical steps:

- bundle install
- asset compile
- database setup

Note: in order to keep the size of this post reasonable, we've simplified some details. Secret management is one major detail we haven't discussed here. **Do not check secrets into source control**. We've made the code we use to encrypt secrets available [here](https://rubygems.org/gems/ejson). A blog post dedicated to that topic will come soon.

## Debugging and details

Applications running inside containers behave identically to their non-containerized counterparts in the vast majority of cases. Additionally, most of your standard debugging techniques (strace, gdb, /proc filesystem) 'just work' from the Docker host.

The one additional tool to become familiar with is [nsenter or nsinit](http://jpetazzo.github.io/2014/03/23/lxc-attach-nsinit-nsenter-docker-0-9/), which will allow you to attach to a running container to debug.  As of Docker 1.3 there is a new docker exec facility that can inject a process into a running container. Unfortunately, if you need the injected process to have root permissions, you’ll still need nsenter.

A few areas where things didn't work out exactly as one would expect were:

## Process Hierarchy

Although we run thin containers, we still have an init process (pid=1) which allows tight integration with our monitoring tools, secret management, service discovery and gives us fine-grained health checks.

In addition to the init process, we added a ppidshim process which occupies pid=2 in every container and simply launches the application process (pid=3) to run. The ppidshim exists so that applications don't inherit directly from init (i.e. ppid != 1), as this can cause them to believe they are daemonized and misbehave.

The resultant process hierarchy looks like this:

```
pid  process                                
---  -------
 1     init                             
 2        \__ppidshim
 3                \_application
```

## Signals

If you're containerizing, you'll likely be modifying existing run scripts or crafting new ones which include a call to `docker run`. By default, `docker run` will proxy signals to your application, meaning that you need to understand how your application deals with signals.

Standard UNIX conventions send `SIGTERM` to request orderly shutdown of a process. Ensure that your application obeys this convention, as we encountered at least one application, Resque, that used `SIGQUIT` for orderly shutdown and `SIGTERM` for emergency shutdown. Luckily Resque (>1.22) can be configured to use `SIGTERM` to [trigger graceful shutdown](http://hone.heroku.com/resque/2012/08/21/resque-signals.html).

## Hostnames

We chose container names that describe the workload (e.g. `unicorn-1`, `resque-2`) and combined this with the hostname of the machine to provide easy traceability. The end result is something like: `unicorn-1.server2.shopify.com`.

We used Docker's `--hostname` flag to pass this into the container, which made most applications report the correct value. Some programs (Ruby) which interrogated the hostname got the short name (`unicorn-1`) rather than the expected `FQDN`.

As Docker manages `/etc/resolv.conf` and our current version doesn't allow arbitrary modifications we used `LD_PRELOAD` instead to inject a library which intercepts and overrides `gethostname()` and `uname()` functions. The end result is that monitoring tools publish our desired hostnames without changes to application code.

## Registry and deploy

We found the process of building a container that replicated the behaviour of a ‘bare metal’ equivalent was mainly a debugging exercise.  Once you’ve got something that’s sane you’ll want to start building containers automatically.

We use github commit hooks to trigger a container build for every master push, and [commit statuses](https://developer.github.com/v3/repos/statuses/) to indicate whether or not the build succeeded.  We use the git commit SHA to ‘docker tag’ the container: so you can see at a glance exactly what version of code is included in the container.  We also put the SHA into a file (`/app/REVISION`) file inside the container for easier debugging and use by scripts.

Once your build is healthy you’ll want to push the container to a central registry.  We chose to run our own registry inside the datacenter for deploy speed and to minimize external dependencies.  We run multiple copies of the standard python registry behind an nginx reverse proxy which caches GET requests, the arrangement looks like this:

![alt](http://resource.docker.cn/shopify-docker.png)

We found that large network interfaces (10 Gbps) and the reverse proxy were effective in combatting ‘thundering herd’ issues around code deploys when multiple docker hosts request the same image.  The proxy approach also allows us to run multiple registries and provide automatic failover in the event of a registry outage.

If you follow this blueprint you’ll now have containers built automatically and stored safely in a central registry ready to be woven into your deploy process.

Our next post in this series will describe how we’re managing application secrets. Following that, we’ll dive deep into how we customized the build process to create ultra-small containers.

---

Original source: [Docker at Shopify: How we built containers that power over 100,000 online shops](http://www.shopify.com/technology/15934308-docker-at-shopify-how-we-built-containers-that-power-over-100-000-online-shops)