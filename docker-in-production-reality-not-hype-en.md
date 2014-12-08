# Docker in production: Reality not Hype 

---

Author: Bridget Kromhout

---

## Why Docker?

When I started talking with [DramaFever](http://www.dramafever.com/) in summer 2014 about joining their ops team, one of many appealing factors was that they’d already been running Docker in production since about October 2013 (well before it even went 1.0). Cutting (maybe bleeding) edge? Sounds fun!

But even before I joined and we were acquired by SoftBank (unrelated events! I am not an acquisition magnet, even if both startups I worked at in 2014 were acquired), DramaFever was already a successful startup, and important technology stack decisions are not made by running a Markov text generator against the front page of Hacker News.

So, why Docker? Simply put, it makes development more consistent and deployment more repeatable. Because developers are developing locally all using the same containers, integration is much easier when their code moves on to their EC2-based personal dev environment, the shared dev environment, QA, staging, and production. Because a production instance is serving code from a container, every new autoscaled instance that has any code at all is going to have the correct code.

As [renowned infosec professional Taylor Swift says](https://twitter.com/SwiftOnSecurity/), “haters gonna hate”. And I’ve been guilty of “get off my lawn” snark about the recent hype, pointing out that containerization isn’t exactly new. We’ve had FreeBSD chroot jails and Solaris Zones (pour one out for Sun Microsystems) for ages. But the genius of Docker is that it provides just enough [training wheels for LXC](https://twitter.com/0x74696d/status/384838181008310275) that everyone can use it (for rapidly increasing values of everyone).

## Our Own Private Registry

We’re using a local copy of the registry backed by an S3 bucket accessible to those with developer IAM credentials. (If you don’t use AWS, that just means it uses a shared storage location that our devs can access without needing production keys.) Apparently people usually go with a centralized private registry; instead, we traded a SPOF for S3’s eventual consistency. We start the local registry on a host via upstart, and there are a few configuration items of interest:

```
# this goes in /etc/default/docker to control docker's upstart config
DOCKER_OPTS="--graph=/mnt/docker --insecure-registry=reg.example.com:5000"
```
Since on some instances we are pulling down multiple Docker images that can be hundreds of megabytes in size, and running or stopped containers also take up room on disk, we use `--graph=/mnt/docker` to set the root of the docker runtime to the ephemeral disk instead of to the default `/var/lib/docker`.

With Docker 1.3’s improved security requiring TLS, this means we need to allow our localhost-aliased non-TLS registry.

The docker registry upstart job (used on all EC2 instances) runs these commands:

```
docker pull public_registry_image
docker run -p 5000:5000 --name registry \
-v /etc/docker-reg:/registry-conf \
-e DOCKER_REGISTRY_CONFIG=/registry-conf/config.yml \
public_registry_image
```

That second line may require some explanation:

```
# this is the local port we'll run the registry on
docker run -p 5000:5000 \  

# giving the container a name makes it easier to identify
--name registry \  

# we're mounting in the directory holding a config file that specifies
AWS credentials, etc. Unlike mount(1), this creates the directory it's
mounting to.
-v /etc/docker-reg:/registry-conf \  

# defining the config file location
-e DOCKER_REGISTRY_CONFIG=/registry-conf/config.yml

# the publicly-registered image we're launching this local registry from
public_registry_image
```

To run locally, we pull the image and then run like this (with DFHOME being where we have the source code checked out):

```
docker run -d -p 5000:5000 --name docker-reg -v ${DFHOME}:${DFHOME} -e 
DOCKER_REGISTRY_CONFIG=${DFHOME}/config/docker-registry/config.yml
public_registry_image
```

## docker build; docker push

Weekly Jenkins jobs build a base container for the main django app and another that mimics our RDS environment with a containerized, all-data-fixtures-loaded MySQL.

We do trunk-based development with developers submitting pull requests. After being peer-reviewed and merged to master, the new code is available for Jenkins to poll GitHub and build. If all tests pass, then it’s time for exciting post-build action! (What? If you’ve gotten this far in a post about container strategy, then you probably agree with me that this stuff is exciting.)

While all our Go microservices are built essentially the same way, let’s focus on the main django app. Its Dockerfile starts from the weekly base build, as that speeds things up a bit:

```
FROM our-local-repo-alias:5000/www-base
```

We keep a number of Dockerfiles around, and in this case, since we have both a base build and a master build for www, we have multiple Dockerfiles in this github repository. Since it’s not possible to pass a file to docker build, it’s necessary to rename the file:

```
mv 'Dockerfile-www' Dockerfile; sudo docker build -t="67cd893" .
```

Jenkins builds the new layers for the www master image, tags it with the git SHA, then tests it. Only if it passes the tests do we retag it as dev and then push it to our private docker repository.

```
sudo docker push our-local-repo-alias:5000/www:'dev'
```

When we’re ready to cut a release, we build the www-QA job from the release branch. After testing, that same container is re-tagged for staging, then production, and new autoscaling instances will pick it up (giving us the flexibility to do blue/green deploys, which we’re just starting to explore).

## Docker in a Mac-using Dev World

Before summer 2014, we were using Vagrant for local development. Building a new image with a local chef-solo provisioner took 17 minutes to install everything, and the local environment diverged enough from the production environment to be annoying. Moving all development into Docker containers proved very effective, especially as we worked through some of the inevitable gotchas and corner cases.

For the local developer environment, we’re using `boot2docker`, and we’re just about to move back to mainline from a fork that Tim Gross, our head of operations, wrote to get around [VirtualBox shared folder mount issues](https://github.com/boot2docker/boot2docker/pull/284#issuecomment-43640542) present in previous versions of boot2docker.

One issue we’ve noticed using boot2docker on Mac laptops is that when they wake from sleep, the clock in the VM can be skewed. This means [the local registry doesn’t work, since it relies on S3 and S3 expects a correct clock](https://github.com/boot2docker/boot2docker/issues/469).

```
$ boot2docker ssh sudo date -u
Mon Nov 24 16:09:02 UTC 2014

$ date -u
Tue Nov 25 01:43:49 UTC 2014

$ docker pull our-local-repo-alias:5000/mysql
Pulling repository our-local-repo-alias:5000/mysql
2014/11/24 19:44:31 HTTP code: 500
```

[Ry4an Brase](http://ry4an.org/), our head of back-end development, came up with this delightful incantation:

```
$ boot2docker ssh sudo date --set \"$(env TZ=UTC date '+%F %H:%M:%S')\"
```

Adding that to the utils sourced by all our various wrapper scripts (so that devs don’t need to remember a lot of docker syntax to go about their daily lives) seemed like a better alternative than having slackbot deliver it as a reply to all local registry questions.

## Containerizing Front-End Dev

I created a container for front-end development which allows us to replicate a front-end environment on Jenkins, using angular, npm, grunt, and bower; you know, the sort of mysterious tools that are inordinately fond of $CWD and interactive prompts.

There are a number of Dockerfiles out there for this; here’s what I found helpful to know (for values of “know” that include “asking Ryan Provost, our head of front-end development” and “mashing buttons until it works”).

Although it defies all logic, node is already old enough to have legacy something. (Insert rant about you kids needing to get off my lawn with your skinny jeans and your fixies.)

```
RUN apt-get install -y nodejs nodejs-legacy npm
```

You need a global install of these three; they can’t come from your package.json:

```
RUN npm install -g grunt-cli@0.1.13
RUN npm install -g bower@1.3.8
RUN npm install -g phantomjs@1.9.7-14
```

And bower doesn’t want to be installed as root, and sometimes will ask questions that expect an interactive answer:

```
ADD bower.json /var/www/dependencies/bower.json
RUN cd /var/www/dependencies && bower install --allow-root 
--config.interactive=false --force
```

The nice thing about having this container is that it allows someone without all the right versions of the front-end tools installed to try out running such parts of the site locally, and it also allows a more replicable deployment of something that will definitely be the same between all the environments as opposed to the “it works on their laptop” fun we all know and love.

## Getting the Logs Out

A certain Docker Orthodoxy treats a container as entirely apart from the host instance. Since we aren’t using containers for isolation, we approach this a little differently. Tim [blogged about Docker logs](http://0x74696d.com/posts/docker-logging/) when DramaFever first started using Docker.

On EC2 we want to ship logs to our ELK stack, so we mount in a filesystem from the host container:

```
-v /var/log/containers:/var/log
```

On local developer machines we want to be able to use a container for active development, editing code locally and running it in the containerized environment. We use the -v flag to mount the developer’s checked-out code into the container, effectively replacing that directory as-shipped:

```
-v ${DFHOME}/www:/var/www
```

We still want logs, too, so we expose those for the dev here:

```
-v ${DFHOME}/www/run:/var/log
```

## Totally Weird Bugs for $1,000, Alex

On an instance where the docker runtime root disk filled up, the container images became corrupt. Even after a reboot, they started yielding inconsistent containers whose behavior would vary over time. For example, a running container (invoked with `/bin/bash`) would have the ls command, and then a few minutes later, it would not. Eventually, a docker run would lead to errors like these:

```
Error response from daemon: Unknown filesystem type on /dev/mapper/
docker-202:16-692241-81e4db1aaf5ea5ec70c2ef8542238e8877bbdb4b0
7b253f67b888e713a738dc2-init

Error response from daemon: mkdir /var/lib/docker/devicemapper/mnt/
9db80f229fdf9ebb75ed22d10443c90003741a6770f81db62 f86df881cfb12ae-init/
rootfs: input/output error
```

It’s likely that we’re seeing one of the devicemapper bugs that seem to plague docker. Replacing the local volume in question was a reasonable workaround.

## About Those Race Conditions

A much more prevalent (and annoying) place we’ve run into docker race conditions is in the Jenkins builds. We’d been seeing builds fail with messages like this:

```
Removing intermediate container 4755dce8cfcc
Step 5 : ADD /example/file /example/file
2014/11/18 18:46:59 Error getting container init rootfs 
a226d3503180de091fde2a410e2b037fde94237dd2171d49a866d43ff03e724c from 
driver devicemapper: Error mounting '/dev/mapper/docker-9:127-14024705-
a226d3503180de091fde2a410e2b037fde94237dd2171d49a866d43ff03e724c-init'
on '/var/lib/docker/devicemapper/mnt/
a226d3503180de091fde2a410e2b037fde94237dd2171d49a866d43ff03e724c-
init': no such file or directory
```

I added the [Naginator plugin](https://wiki.jenkins-ci.org/display/JENKINS/Naginator+Plugin) so it would retry failed jobs if they’d failed with the most common strings we’d see:

```
(Cannot destroy container|Error getting container init rootfs)
```

While that’s an acceptable workaround, I still plan to change what gets reported to Slack, since it’s annoying to have to click on the broken build to find out if it’s just Docker again.

## Cron Zombies

A few weeks ago, a developer noticed an unwelcome new message in interactive use on QA:

```
Error: Cannot start container appname: iptables failed: iptables -t 
nat -A DOCKER -p tcp -d 0/0 --dport 8500 ! -i docker0 -j DNAT --to-
destination 172.17.0.7:8500:  (fork/exec /sbin/iptables: cannot
allocate memory)
```

On specific instances (such as QA) that aren’t part of the production autoscaling groups, we run cron jobs that invoke a container and give it arguments. A look at the process table showed that multiple days of `docker run` commands started by cron and the python processes they’d spawned were still running. `docker ps` disagreed, though; the containers weren’t running anymore, so they weren’t getting cleaned up by these cron jobs:

```
# remove stopped containers
@daily docker rm `sudo docker ps -aq`
# remove images tagged "none" 
@daily docker rmi `sudo docker images | grep none | awk -F' +' 
'{print $3}'`
```

At the time, we were starting the cron containers with `docker run -i -a stdout -a stderr`.

Changing the container-invoking cron jobs to instead use `docker run -it` cleared it up. It appears a controlling tty was necessary for them to successfully signal their child processes to exit.

## Containerize All the Things?

We’re actually running just about everything in containers currently - including the more static bits of our infrastructure. Do Sentry, Jenkins, Graphite, and the ELK stack actually benefit from being in containers? Possibly not; at the time of containerizing all the things, it was the closest thing we had to a configuration management system.

But while it’s excellent for releasing software, it’s a giant hassle sifting through all the changes in a monolithic “config” repo to figure out why the graphite container no longer builds to a working state. Now that we’re using Chef and Packer to drive our AMI creation, we’ll likely move to using Chef cookbooks to manage our next iteration on those infrastructure pieces.

Just because it’s possible to run everything inside a container doesn’t mean it’s useful. While we are no longer using containers as our main method of capturing all configuration, we continue to see great value in using them for consistency throughout development and repeatability of deployments.

## Not (Just) Hype

The core of devops is empathy, and it’s important to remember there are people behind all the software we’re discussing. Nathan LeClaire of Docker took to Twitter recently, talking about [how it feels to have your project called “marketing BS”](https://twitter.com/upthecyberpunks/status/534123083687161856). (Let’s pause for a moment while I feel guilty about everything I’ve ever said about MongoDB. I don’t think I ever called it marketing BS, but I’ve definitely made “web scale” jokes.)

Given the recent announcements out of AWS re:Invent about [EC2 Container Service](http://aws.amazon.com/blogs/aws/cloud-container-management/), it’s safe to say that containers are about as mainstream as they’re going to get. Do I think ECS is going to be ready for prime time immediately? Anyone who read [my sysadvent post from last year](http://sysadvent.blogspot.com/2013/12/day-18-wide-columns-shaggy-yaks-hbase.html) about HBase on EMR (Amazon’s training wheels for Hadoop) is saying “lolnope” right about now.

But containers are definitely not just for the Googles of the world anymore, and they’re increasingly no longer just for those of us who are willing to chase devicemapper bugs down a rabbit-hole into GitHub issue land. Docker is the real deal, it works in production, and if you’d like to go stream some dramas powered by it from our site or native apps, you can do that today. (If you’d like to read more about containers, stay tuned for an exciting post tomorrow…)

## Sound Like Fun?

If this sounds like exactly the sort of fun you enjoy having at work, we’re hiring ops and dev folks at DramaFever. We’re remote-friendly with NYC and Philly offices. You can read more about our positions on the [DramaFever careers page](http://www.dramafever.com/company/careers.html) or contact me via email or on twitter. I’d love to talk with you!

---

Original source: [Docker in production: Reality not Hype](http://sysadvent.blogspot.nl/2014/12/day-1-docker-in-production-reality-not.html)