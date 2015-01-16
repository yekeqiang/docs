# A RAILS DEVELOPMENT ENVIRONMENT USING DOCKER THROUGH FIG

---

Author: José Ribeiro

---

**Warning**: This post is written by the DevOps guy, not the Rails guy. This post, however, is to be read by you, Rails girls and boys. It assumes the 
reader is not familiar with Docker. For those of you who already know Docker, 

I'm sure you'll enjoy Fig.

If you're in a hurry, jump straight to the [TL;DR](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#tldr).

## INTRODUCTION

Onboarding a member to a project is usually a painful process, especially when it has multiple dependencies: a PostgreSQL database, a Redis instance, ElasticSearch... It may take a while until the development environment is properly configured so we may actually... code. Additional pain comes from managing multiple Ruby versions. Setting everything up properly takes way too much time and energy, and it shouldn't be that way.

We wanted to ease the pain of setting up a basic Rails development 
environment, so it becomes **repeatable** on every developer machine with fewer keystrokes and headaches. For that we chose [Docker](https://www.docker.com/) with the help of a neat tool called [Fig](http://www.fig.sh/).

By the end of this post, you should be able to tell your coworkers to just `fig up` and they'll be ready to go! `fig rm` and it's gone [[1]](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#note-1)!

## DOCKER

Some people refer to [Docker](https://www.docker.com/) as a "lightweight VM", but that's a description we would like to avoid. [Solomon Hykes](https://github.com/shykes), Docker's Founder & CTO calls Docker "a standard format for shipping software".

The techy explanation: Docker uses low level primitives of the Linux kernel that make it possible for us to sandbox the execution of processes ([cgroups](https://en.wikipedia.org/wiki/Cgroups), [kernel namespaces](http://blog.dotcloud.com/under-the-hood-linux-kernels-on-dotcloud-part)). This means our processes are able to run isolated from the remaining processes of the host system (the system on which the Docker containers run). The advantages of Docker go beyond process isolation, but I won't go there.

Since it uses Linux kernel features, Docker containers only run on Linux 
systems, but hey, don't worry OS X and Windows guys! There's a simple solution 
for you, too!

## FIG

Although Docker is pretty easy to use once you understand it, most people just want the web app and the database up and running as fast as they can. [Fig](http://www.fig.sh/) simplifies the process of setting up multiple services using Docker by allowing us to describe them and their relations on a `fig.yml` like this:

```
web:  
  build: .
  command: python app.py
  links:
   - db
  ports:
   - "8000:8000"
db:  
  image: postgres
```

It is mostly self explanatory (probably with the exception of the `build: .` bit), but we rather explain it in detail for our specific Rails setup.

## PREREQUISITES

For our Rails Development Environment approach, it is necessary to have the following software installed:

- Docker 1.3 (or higher)
- Fig 1.0 (or higher)

There's a timing for this post and a reason for being specific about the Docker and Fig releases: they all come after the announcement of the [Docker 1.3](http://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/) release, which fixed Shared directories on *Mac OS X* on `boot2docker` (explained below). This is good news for our Mac OS X coworkers, since this means it is now possible for them to update-and-refresh 
[[2]](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#note-2), the Rails way (as long as the code is under `/Users`, such as `/Users/some-user/some-dir/code-dir`)!

**To install Docker**, choose your OS on the [Docker installation](https://docs.docker.com/installation/) page and follow the instructions. While Linux systems are able to install it directly on 
their machines, OS X relies on the use of a VM with a lightweight Linux distro, called `boot2docker`; the [Mac OS X instructions](https://docs.docker.com/installation/mac/) are really easy to follow, so you should be good. Please make sure to setup your environment properly, (the `$(boot2docker shellinit)` part), so that Fig is able 
to find Docker.

**To install Fig**, just follow the [Fig installation instructions](http://www.fig.sh/install.html) (you're just a `pip install` away!) and you'll be all set.

## THE RAILS DEV ENV

If you read so far, congratulations! The fun part begins now.

As we said when talking about Fig, we'll be using a `fig.yml` that describes the services that compose our application. In this setup we'll Dockerize:

- A **Ruby on Rails** web application, listening on port **3000**.
- A **PostgreSQL** database, listening on port **5432**, **only accessible by the RoR web app**.

For this, you may clone our **barebones development environment** [GitHub repo](whitesmith/fig-tree/rails-pg-fig-devenv) or follow the instructions on our **sample Rails app** [GitHub repo](whitesmith/rails-pg-fig-sample).

As you can see from the barebones dev env repo, the magic relies on two files: a `fig.yml` and a `Dockerfile`.

Let's see our `fig.yml`:


```
db:  
  image: "postgres:9.3"
  volumes:
    - ~/.docker-volumes/app-name/db/:/var/lib/postgresql/data/
  expose:
    - 5432
web:  
  build: .
  command: bundle exec unicorn -p 3000 -c ./config/unicorn.rb
  volumes:
    - .:/usr/src/app
  ports:
    - "3000:3000"
  links:
    - db
```

We can see the db service is using the `postgres:9.3` image (more about it on [postgres official repo](https://registry.hub.docker.com/_/postgres/)) and that this container exposes a service on its port 5432 (only accessible to the `web` container). It also mounts a volume:

```
  volumes:
    - ~/.docker-volumes/app-name/db/:/var/lib/postgresql/data/
```

This means that the container's `/var/lib/postgresql/data/` directory will have its contents mapped to the host's `~/.docker-volumes/app-name/db/` directory. This allows us to remove the container safely when not in use, knowing we'll be able to reuse an already setup database when needed. Before continuing, **change the volume path** to match your application's name and **create that path**, since Docker assumes it already exists (`mkdir -p ~/.docker-volumes/hello-world/db/`).

`web` is doing a `build .` and, as you may have guessed, this is the command using the `Dockerfile`. Our `Dockerfile` simply contains

```
FROM rails:onbuild 
```
 
which pulls and builds the `rails:onbuild` image. If you need to specify your Ruby version, please see [Choosing Ruby version](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#choosingrubyversion). As you may see in the [rails official repo](https://registry.hub.docker.com/_/rails/), the `Dockerfile` assumes it is on the same directory as the root of the Rails application.

Back to `fig.yml`:

```
volumes:  
    - .:/usr/src/app
```

is where the magic happens. This mounts our Rails application's code directory inside the Docker container at `/usr/src/app`. This is what enables us to change the code and see our changes happen live. The `ports: "3000:3000"` bit means our application's port 3000 will be mapped to our Docker host's port 3000, so we can access it on our browser.

Docker makes the link between services and lets them know of each other using hosts; this means we need to **change our `development` and `test` 
environments on `database.yml`** to use the db host and the default username:

```
development:  
  host: db
  username: postgres
```

```
test:  
  host: db
  username: postgres
```

You should use the `database.yml` provided in [whitesmith/fig-tree/rails-pg-fig-devenv](https://github.com/whitesmith/fig-tree/tree/master/rails-pg-fig-devenv) as reference.

## THE COMMANDS

Please make sure:

- 1. You have copied `fig.yml` and the `Dockerfile` to your application's folder.
- 2. You have adapted `fig.yml` to match your application's name.
- 3. You have created the database's volume directory on the host.
- 4. You have adapted `config/database.yml` to use the `db` container.

### FIRST TIME RUNNING

Our commands will be different when running for the first time.

First, we need to run our database.

```
fig up -d db
```
  
The `-d` flag means we're running it in detached mode, so we can continue to enter commands. If this your first time using the `postgres:9.3` image, this may take a while, since it will be downloaded; future projects that use this image won't need to download it again.

Now we need to setup the database.

```
fig run web rake db:setup
```
  
This will create an instance of the `web` container and run `rake db:setup` inside it. When the command is over, the container will stop running. If this is your first time using the `rails:onbuild` image, this will also take a while.

Now we just have to run

```
fig up
```
  
Since the `db` container was already running, this will simply start the `web` container.

**Voilà**! Open your browser at [http://localhost:3000](http://localhost:3000/)! **Note**: If you're using `boot2docker`, it will be available at [http://192.168.59.103:3000](http://192.168.59.103:3000/), but YMMV; please check your Docker host's IP by running `boot2docker ip`.

### SECOND TIME RUNNING

Just

```
fig up
```
  
and you're good to go. `fig stop` when you're done with your work for the day and `fig rm` for clean up; because we've persisted the database on the host, removing the container won't destroy the database unless you remove the folder. 

If you're completely done with your project, see [[1]](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#note-1) for details.

## THE END

We hope you find this helpful. Any feedback, questions or issues are welcome. Just drop by [whitesmith/fig-tree](https://github.com/whitesmith/fig-tree) and let us know.

This is an experiment; we are currently starting to use it on some of our projects and if it works, you can expect us to share more complex setups in the future.

## TL;DR

The juice:

- 1. [Install Docker](https://docs.docker.com/installation/).
- 2. [Install Fig](http://www.fig.sh/install.html).
- 3. `git clone git@github.com:whitesmith/rails-pg-fig-devenv.git`
- 4. Add the repo's `fig.yml` and `Dockerfile` to your RoR application's root.
- 5. Edit `fig.yml` to match your application's name (the `db` volume path) and create that folder structure.
- 6. `fig up -d db` (This downloads the PostgreSQL image, grab a beer.)
- 7. `fig run web rake db:setup`
- 8. `fig up` (This downloads the Ruby image, another beer.)
- 9. Open your browser at `[http://localhost:3000](http://localhost:3000/)` (if you're using `boot2docker` (aka OS X), replace `localhost` by the output of `boot2docker ip`, most likely [http://192.168.59.103:3000](http://192.168.59.103:3000/)).

The images downloaded in steps 4. and 6. will be reused by future projects, so don't worry: this will only happen the first time you run these commands on your machine (unless you use different versions).

## APPENDIX

### CHOOSING RUBY VERSION

If you want to specify the Ruby version used by your `web` container, an easier way to do so is simply copying the contents of [rails:onbuild  Dockerfile](https://github.com/docker-library/rails/blob/master/onbuild/Dockerfile) to your `Dockerfile`, removing the `ONBUILD` prefixing some of the Docker commands [[3]](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#note-3) (since those commands would only be triggered when using this as a parent image) and changing the `FROM ruby:X` to match your desired version. Remember that `X` is a Docker tag, which means you should check the [ruby official repo](https://registry.hub.docker.com/_/ruby/) first for the list of supported tags. Remember: you can always write your first [Dockerfile](https://docs.docker.com/reference/builder/)!

### CLEANING UP THE PROJECT

**Note 1**: `fig rm` removes your project containers. If 
you would like to completely remove the project, you should:

- 1. **Remove the `db` volume**. Remember creating `~/.docker-volumes/app-name/db/?` This is where your database data resides. If you're done with it, simply remove it. You'll have to `rake db:setup` the next 
time.
- 2. **Remove unused Docker images**: When you `fig up` for the first time, it downloads the necessary Docker images (Ruby and PostgreSQL). If you're using them for different projects, great, that means you found this useful! But if you're not using them in other projects and you're not thinking of doing `fig up` anytime soon, maybe you should remove them. Do `docker images` to list them and `docker rmi <image-id>` to remove them.

### UPDATE & REFRESH

**Note 2**: *Update-and-refresh* on non-Linux systems was already possible [using other approaches such as NFS](http://www.talkingquickly.co.uk/2014/06/rails-development-environment-with-vagrant-and-docker/). But following that approach while keeping Linux/Mac OS X dev envs as similar as possible would require us to either:

- Maintain the Vagrantfile/*Linux-setup-strategy* in sync or
- Make Linux users use the Mac OS X guys VM and make them install NFS, so they could all use the same Vagrantfile (not cool OS X guys, Linux guys don't need to use VMs).

While it would be possible, for instance, to reuse the setup scripts used by the Vagrantfile on Linux systems, [I](https://twitter.com/jlbribeiro) feel the [Docker 1.3](http://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/) release simply eliminates the need for heterogeneity amongst OSes (after all, that's Docker's philosophy). Instead, *It just works™*.

### ONBUILD REMOVAL

**Note 3**: A simple `sed -i 's/^ONBUILD //g' Dockerfile` does the trick.

---

Original source: [A RAILS DEVELOPMENT ENVIRONMENT USING DOCKER THROUGH FIG](http://blog.whitesmith.co/a-rails-development-environment-using-docker-through-fig/#note-1)