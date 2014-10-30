## How I Docker

---

##### Author: Barett Clark

---
 
I have spent the past several months playing with Docker off and on. I have to say that I think I really like it.

I love the idea of really small, specific, containers. I also love the idea of building up a toolkit of things that you use.

## Some Background

I work at Sabre Labs, and we get to explore and experiment with trends and technologies that we think could make a difference in the travel world. This means that we spin up a lot of projects in a lot of different technologies and languages. So it is really nice to have a toolkit of things that we’ve used and done. That also helps with knowledge sharing.

About a year ago we set upon a path to attempt to make DevOps easier in a cross-functional team. We have a designer, a front-end developer, and a couple of back-end or full-stack developers. DevOps can be very time consuming when you’re working in a Java stack one day and a Rails stack the next.

## Vagrant

First we looked at Vagrant, using Chef to setup the dependencies. It worked, generally. It’s expensive. It’s messy.

The licenses are frustrating. You pay for Vagrant, which is fine. If you want to use VMWare, which we did, you pay for that. Also fine. If you want to use Vagrant with VMWare you pay for a third license. That feels weird.

Chef. I’ve used Chef before in previous engagements, and it’s worked ok. It’s better than nothing (or shell scripts) for sure. My frustration with Chef is how messy the recipes are. The dependencies in particular. We didn’t use Chef Server, so we would just copy in all the recipes that we needed, and then their dependencies. Look at the dependency chain for git (on Ubuntu). Before you’re done you’ve brought in Windows dependencies. And also the Yum package manager. Why? That makes me crazy.

I know that Vagrant supports Puppet, and I just never got around to looking at Puppet. I know that Vagrant also supports Docker now, and I can’t get it to dance. But I don’t really want to. I think that’s overly complicated.

## Docker

What’s so great about Docker then? It’s complicated in different ways. Or can be complicated.

Like I said at the top, I love how simple and concise the containers are. I also like the DSL. It’s specific, and it lets me do what feels natural on that particular environment. I don’t need to overly generalize. I don’t need a DSL that can work on all *nix distros. I just want to say what I need and make it so.

## Teach Me How To Docker

Look around at blog posts on how to Docker. I’ve noticed that a great many of them fall into the “install ALLTHETHINGS” camp. You build a single container that includes the database and the app server. Bleh. Why bother with containerization if you’re just going to stick everything in a single container?

Small. Simple. Concise. That’s how I Docker.

So, without all of that as the background, here is a simple Docker setup for a simple Rails app.

*Dockerfile*:

```
FROM barrettclark/ruby-2.1.2

RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get -y --fix-missing install libpq-dev nodejs

RUN gem install bundler --no-ri --no-rdoc

ADD start.sh start.sh
RUN chmod +x /start.sh
ADD project/Gemfile Gemfile
ADD project/Gemfile Gemfile.lock
RUN bundle install
RUN rm /Gemfile*

ADD project/ /rails
WORKDIR /rails

EXPOSE 3000

CMD /start.sh
```

You see here that I have a base container image called `barrettclark/ruby-2.1.2`. Docker containers can inherit from other containers. This is where you can end up with a bit of a Russian Doll situation because that container takes a base image and puts Ruby 2.1.2 on it. You see it then runs some basic apt-get commands. We are using Postgres for this Rails app, so we need those header files for the pg gem, hence libpq-dev.

Next we add some files. Docker caches each step to see if it needs to be rebuilt. That’s why we don’t just copy in the whole project yet. Once we’ve handled the gem dependencies, then we can copy over the app. Rails will run on port 3000, and because we know it’s the only thing running in that container, and also in this machine for that matter, we can just expose that port and call it good.

`CMD /start.sh` tells the container to run that file when the container is started (unless you tell it to do something else when you start it). You can see that it does some housekeeping, database setup, and fires up the app.

*start.sh*

```
echo "*** STARTING RAILS APP ***"
rm /rails/tmp/pids/server.pid
bundle install
rake db:create
rake db:migrate
rake db:seed
bundle exec rake log:clear
rails s
```

Build the container: docker `build -t="barrettclark/rails-basic:devel"` . Great. Now you’ve got everything you need to run a simple Rails app in Docker.

## Run All The Things

I usually write a little shell script to help with this step. We need 2 containers, and we want them eventually linked together.

```
docker run -d \
--name rails-basic-postgres \
-e POSTGRES_USER=docker -e POSTGRES_PASSWORD=docker \
barrettclark/postgis

docker run -d -P \
--name rails-basic-app \
--link rails-basic-postgres:postgres \
-e POSTGRES_USER=docker -e POSTGRES_PASSWORD=docker \
-v /Users/barrettclark/temp/rails_basic/project:/rails \
barrettclark/rails-basic:devel /bin/bash -l -c "/start.sh"
```

What’s all that? The first docker run spins up a Postgres container named ‘rails-basic-postgres’ and sets some environment variables. If you don’t have the given image on your localhost, Docker will go try to fetch it from Dockerhub. The ‘-d’ switch tells Docker to daemonize the container — run it in the background. Next we run the Rails container and link that Postgres container to it.

Remember how I said the whole point of all this was to be able to give this service (or whatever) to everyone else in a cross-functional team. I don’t want them to have to know how to line up all these dependencies. This allows me to handle that for them. **The real kicker here is that you can mount a directory on the localhost inside the container**. Yes! The caveat is that the localhost directory MUST live in /Users. A symlink is not sufficient. Now this allows a designer to play with the javascript or markup on his or her computer in his or her preferred editor, and nobody has to know where the code is actually running.

Well, that last piece isn’t entirely true. Remember how I sort of waved my hands at just exposing port 3000 and calling it good? The `-P` Docker option allows us to expose ports, and Docker will map them to a port on the localhost. If you want to be specific you can do that as well with a different switch.

Run `docker ps -a` to see all the instances running and what ports they’ve been mapped to.

I’m not saying that any of this is particularly correct. It’s just what has evolved and makes the most sense to me. The next blog post will go into a little more detail on how I actually use all of this, and some tooling that I’ve written to make it a little easier.

---

Original source: [How I Docker](http://cookingco.de/2014/10/27/how-i-docker/)

