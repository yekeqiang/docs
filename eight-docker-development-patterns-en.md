# Eight Docker Development Patterns

---

##### Author: Vidar Hokstad

---

In the past I've written about my "[home cloud](http://www.hokstad.com/the-home-cloud)" that used to be [OpenVz](http://openvz.org/) containers, and [how I've come to advocate rebuilding the "build server" for every build](http://www.hokstad.com/rebuilding-the-build-server-on-every-build).

[Docker](http://docker.io/) has quickly become one of my favorite enabling tools for this overall philosophy of creating repeatable builds that results in as-static-as-possible server environments.

Here I will outline some patterns that have started to show up repeatedly in my use of Docker. I don't expect any of them to be particularly novel or any big surprises, but I hope some can be useful, and I would very much like to hear from others about patterns you come across while working with Docker.

A foundation for all of my Docker experiments, is keeping state that should persist in volumes, so that the Docker containers themselves can be re-created at will without data loss (unless I've been naughty and modified container state without updating the Dockerfile's - and regularly rebuilding the containers helps stop that bad habit).

The examples Dockerfiles below are all focused on that: Creating containers where the containers themselves can be replaced at any time without having to think about it.

The more regularly the containers are recreated; the more habitual this becomes, the more it reinforces a habit of avoiding state outside of clearly defined locations that are explicitly persisted.

## 1. The Shared Base Container(s)

Docker encourages "inheritance" so this should not come as a surprise - it is a fundamental aspect of effectively using Docker, not least because it contributes to reducing the time it takes to build new containers by not having to re-execute steps as often. Docker does a good job - sometimes too good - of trying to cache intermediate steps, but it is easy to lose out on opportunities for sharing when not being explicit about it.

One of the first things that became apparent on migrating my various containers to Docker was the amount of redundant setup.

I run separate containers for most projects that I expect to ever deploy anywhere, at least the moment it needs any long running processes, or needs any specific packages beyond my "standard" set, so I have many containers, and the set is rapidly growing.

To the point where I'm considering moving towards trying to run "everything" (including the few desktop apps I depend on) in Docker in order to make mybase environment entirely disposable.

So I quickly started extracting my basic setups into base containers for various purposes. Here's my current "devbase" Dockerfile:

```
	FROM debian:wheezy
	RUN apt-get update
    RUN apt-get -y install ruby ruby-dev build-essential git
    RUN apt-get install -y libopenssl-ruby libxslt-dev libxml2-dev
    
    # For debugging
    RUN apt-get install -y gdb strace
    
    # Set up my user
    RUN useradd vidarh -u 1000 -s /bin/bash --no-create-home
    
    RUN gem install -n /usr/bin bundler
    RUN gem install -n /usr/bin rake
    
    WORKDIR /home/vidarh/
    ENV HOME /home/vidarh
    
    VOLUME ["/home"]
    USER vidarh
    EXPOSE 8080
```

There's nothing much special to note about it - it installs some specific tools I tend to like to always have available. These will presumably be different for most people. The distribution choice is pretty arbitrary. The one thing about it worth considering is to specify a specific tag to avoid surprises if/when you rebuild a container (and I need to get better at that too - I do it for the distributions, but I too rarely tag my own containers).

It exposes port 8080 by default, since that's where I usually expose my web apps, which is what I usually use these containers for.

It adds a user for me, and forces the userid to the user id I have on my server, and doesn't create a /home directory. This latter is because I bind mount a shared /home from the host. Which brings us to the next pattern:

## 2. The Shared Volume Dev Container

All of my dev containers share at least one volume with the host: /home. This is for ease of development. For many apps it lets me leave the apps running with a suitable file-system-change based code-reloader in dev mode, so that the containers encapsulate the OS/distro-level dependencies, and help verify that the app-as-bundled works in a pristine environment, without making me restart/rebuild the VM fully on every code change. Meanwhile I can, and do restart them fairly frequently to ensure that nothing slips past.

For the rest, it lets me just restart (rather than rebuild) the container to pick up code changes.

For test/staging and production containers I would in most cases avoid sharing code via volumes, and instead use "ADD" to add the code in question to the Docker container itself.

Here is the Dockerfile for my "homepage" dev container, for example, which contains my homegrown personal wiki, that draws on the already shared /home volume from my "devbase" container, and so showcases the shared base and how I use the shared /home volume:

```
    FROM vidarh/devbase
    WORKDIR /home/vidarh/src/repos/homepage
    ENTRYPOINT bin/homepage web
```

(note, I really should version tag my devbase container)

And for my dev-version of my blog:

```
    FROM vidarh/devbase
    
    WORKDIR /
    USER root
    
    # For Graphivz integration
    RUN apt-get update
    RUN apt-get -y install graphviz xsltproc imagemagick
    
    USER vidarh
    WORKDIR /home/vidarh/src/repos/hokstad-com
    ENTRYPOINT bundle exec rackup -p 8080
```

Because they pull in the code from a shared repository, and are based on a shared base container, these containers are in general extremely fast to rebuild when I add/modify/remove dependencies, which I find important in order to ensure I don't get tempted to resort to workarounds that leave undocumented dependencies.

Even so, there are certainly things I'd like to improve on. Even though the base for the above are lightweight, they could certainly be much more so: Most of what are in these containers remain unused. Since Docker uses copy-on-write overlays, this does not result in a huge overhead, but it does still mean I am not truly capturing the bare minimal requirements, nor minimizing the attack- or error-surface as much as possible. (I'm not so concerned about attack surface for these specific cases, as e.g. my blog does not maintain any important state in the "live" version that's not sitting in two git repositories in two other locations, and it gets overwritten on every push I do; but it's the principle)

## 3. The Dev Tools Container

This may be most attractive for those of us who like to rely on ssh to a screen session to work on code, and less for the IDE crowd, but for me, one of the nice things about the above setup is that it lets me separate out editing and test-executing parts of code from running the in-development application.

One of the annoying thing in the past for me with dev-systems have been that dev and production dependencies and development tool dependencies easily get co-mingled. You can try to keep them apart, but unless these setups are genuinely kept separated, it is all too easy to create undocumented dependencies by e.g. in a bind during debugging apt-get'ing strace or tcpdump - or even emacs - in an environment that does not really need it, and pull in a boatload of dependencies for the various tools at the same time.

In the past I had this driven home after spending weeks "reverse engineering" the dependencies of an application that had accreted all kinds of undocumented dependencies due to co-mingling of dev-environment, testing and initial prototype deployment environment.

While there are many ways of solving this by ensuring you do regular test-deployments, combined with the patterns above I have a solution I quite like because it prevents the problem from arising in the first place:

I have a separate container that contains my Emacs installation, and various other tools that I like to have available. I still try to keep it sparse, but the point is, my screen session can live in this container, and coupled with "autossh" set up on my laptop, there's pretty much always a running connection to it, where I can edit code that is shared "live" with my other dev containers.

This container is also one where I allow the occasional transgression of installing packages directly (with the caveat that if I want to keep them around, I need to put them in my Dockerfile, or they'll get blown away next time I rebuild) because it only affects debugging and development.

Currently it looks like this:

```
FROM vidarh/devbase
RUN apt-get update
RUN apt-get -y install openssh-server emacs23-nox htop screen

# For debugging
RUN apt-get -y install sudo wget curl telnet tcpdump

# For 32-bit experiments
RUN apt-get -y install gcc-multilib 

# Man pages and "most" viewer:
RUN apt-get install -y man most

RUN mkdir /var/run/sshd
ENTRYPOINT /usr/sbin/sshd -D

VOLUME ["/home"]
EXPOSE 22
EXPOSE 8080
```

Combined with the shared "/home", this gets me a sufficiently functional little place to ssh into. I'm sure I'll augment it as I use it more, but for now it has proven quite sufficient for my needs.

## 4. The Test In A Different Environment containers

One aspect of Docker I particularly love is the ease with which I can test my code in different environments. For example, when I upgraded my [Ruby compiler project](http://www.hokstad.com/compiler) to handle Ruby 1.9 (much overdue), I created this trivial little Dockerfile to let me spawn a shell in a Ruby 1.8 environment after I'd transitioned my main dev environment to 1.9:

```
FROM vidarh/devbase
RUN apt-get update
RUN apt-get -y install ruby1.8 git ruby1.8-dev
```

Of course you can achieve a similar effect with rbenv etc.. But I've always found those tools annoying as I prefer to deploy as much as possible with distro-packages (as painful as that has been with Ruby in the past), not least because it makes it easier for others to use my code if I ensure that works smoothy.

Having a docker container that I can just "docker run" when I need a different environment for a few minutes solves the problem neatly, and also has the benefit that it is not constrained to things like Ruby that have pre-packaged custom tools to deal with versioning.

I could also achieve it with full VM's, as is the case for many of the things I describe, but I can start the above docker containers in far less time.

## 5. The Build Container

Most of what I write are in interpreted languages these days, but even then there are often useful "build" steps that are expensive enough that I'd prefer not to execute them all the time.

An example is running "bundler" for Ruby applications. Bundler updates cached dependencies for Rubygems (and optionally full gem files, or even unpacked contents) and it can take some time to run for a larger app.

It also often requires dependencies that are not needed at runtime for the application. E.g. installing gems that rely on native extensions can often depend on so many packages - often without documenting exactly which - that it's easier to just start by pulling in all of build-essential and it's dependencies. At the same time, while you can make bundler do all the work up-front, I really don't want to run it in the host environment that may or may not be compatible with the containers I want to deploy in.

A solution for this is to create a build container. You can create a separate Dockerfile if the dependencies are different, or you can reuse the main app Dockerfile and just override the command to run your desired build commands. E.g. the Dockerfile may look like this:

```
FROM myapp
RUN apt-get update
RUN apt-get install -y build-essential [assorted dev packages for libraries]
VOLUME ["/build"]
WORKDIR /build
CMD ["bundler", "install","--path","vendor","--standalone"]
```

Then run this while mounting your build/source directory in the containers "/build" directory whenever you have updated your dependencies. Replace as suitable.

The point is that you can decouple build of your applications, or parts of it from the final packaging, while still encapsulating both processes and their dependencies in Docker containers by breaking the process into two or more containers.

(I often check in the build artefacts, to be 100% certain I can reproduce the final packaging steps; an alternative is to do a full end-to-end build and test on a regular basis as well, to make sure the split accurately reflects th full process)

## 6. The Installation Container

This is not my own, but really deserves a mention. The excellent [nsenter and docker-enter tools](https://github.com/jpetazzo/nsenter) comes with an installation option that is a nice step forward from the popular but terrifying "curl [some url you have no control over] | bash" pattern. It does this by providing a Docker container that implements the "Build Container" pattern from above, but goes one step further. It deserves a look.

This is the last portion of the Dockerfile, from after it has downloaded and built a suitable version of nsenter (my one caveat is no integrity check of the downloaded archive):

```
ADD installer /installer
CMD /installer
```

"installer" looks like this:

```
#!/bin/sh
if mountpoint -q /target; then
       echo "Installing nsenter to /target"
       cp /nsenter /target
       echo "Installing docker-enter to /target"
       cp /docker-enter /target
else
       echo "/target is not a mountpoint."
       echo "You can either:"
       echo "- re-run this container with -v /usr/local/bin:/target"
       echo "- extract the nsenter binary (located at /nsenter)"
fi
```

While the potential is still there for a malicious attacker to try to exploit potential privilege escalation problems with containers, the attack surface is at least substantially smaller.

But the most likely immediate win for most of us with this pattern is to avoid the risk of [wellmeaning developers who occasionally make very dangerous mistakes in installation scripts](https://github.com/MrMEEE/bumblebee-Old-and-abbandoned/issues/123).

I really love this approach. And hopefully it will help curtail the abomination that is "curl [something] | bash", (but even if it doesn't, at least we can easily contain that within containers too...)

## 7. The Default-Service-In-A-Box Containers

While I relatively quickly if I get "serious" with an app will prep suitable containers to handle the database etc. for the project, I find it invaluable to have a series of "basic" infrastructure containers sitting there, with suitable tweaks for me to be able to spin up e.g. the database of my choice, or queueuing system of choice with suitable default settings.

Of course you can get "mostly there" by just trying "docker run [some-app-name]" and cross your fingers that there's a great alternative in the Docker index, and often there is. But I like to vet them first, and figure out e.g. how they handle data, and then I will be more likely to add my own tweaked version to my own "library".

E.g. I have one sitting around for Beanstalkd:

```
FROM debian:wheezy
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -q update
RUN apt-get -y install build-essential
ADD http://github.com/kr/beanstalkd/archive/v1.9.tar.gz /tmp/
RUN cd /tmp && tar zxvf v1.9.tar.gz
RUN cd /tmp/beanstalkd-1.9/ && make
RUN cp /tmp/beanstalkd-1.9/beanstalkd /usr/local/bin/
EXPOSE 11300
CMD ["/usr/local/bin/beanstalkd","-n"]
```

(For e.g. Postgres I have a vastly more complicated one that amongst others also set up persistent volumes for data, configuration etc.)

My goal is to be able to go "ok, I'm going to need memcached, postgres and beanstalk" and three quick "docker run"'s later have three containers up and running that are tweaked to my environment and personal preferences for a default environment, and ready to go. Setting up "standard" infrastructure components should be a 1 minute job rather than take away from the development itself.

## 8. The Infrastructure / Glue Containers

Many of these patterns focus on the development environment (and this means there's many more to discuss some other time for production environments), but there is one big category missing:

Containers whose purpose is to glue your environment together into a cohesive whole. This is an under-explored area for my part so far, but I will mention one particular example:

To enable easy access to my containers, I have a small little haproxy container. I have a wildcard DNS entry pointing at my home server, and an iptable entry opening up access to my haproxy container. The Dockerfile is nothing special:

```
FROM debian:wheezy
ADD wheezy-backports.list /etc/apt/sources.list.d/
RUN apt-get update
RUN apt-get -y install haproxy
ADD haproxy.cfg /etc/haproxy/haproxy.cfg
CMD ["haproxy", "-db", "-f", "/etc/haproxy/haproxy.cfg"]
EXPOSE 80
EXPOSE 443
```

The fun part here is the haproxy.cfg, which is generated by a script that generates backend sections like this from the output of "docker ps"

```
backend test
    acl authok http_auth(adminusers)
    http-request auth realm Hokstad if !authok
    server s1 192.168.0.44:8084
```

and a bunch of acls and use_backend statements in the frontend definition that dispatches [hostname].mydomain to the right backend.backend test.

If I wanted to be fancy, I'd deploy something like [AirBnB's Synapse](https://github.com/airbnb/synapse), but it has far more options than I need for my dev use.

For my home environment, this makes up most of my infrastructure needs. While at work I'm rolling out an increasing array of infrastructure containers whose purpose is to make deployment of the actual applications a breeze as I'm transitioning a full private cloud system towards Docker.

---

Original source: [Eight Docker Development Patterns](http://www.hokstad.com/docker/patterns)