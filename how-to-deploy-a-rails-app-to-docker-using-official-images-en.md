# HOW TO DEPLOY A RAILS APP WITH DOCKER USING OFFICIAL IMAGES

---

Author: Lucas Carlson

---

![alt](http://resource.docker.cn/docker-ruby.png)

The Docker [official images](https://github.com/docker-library/official-images) have created a canonical way to build Docker images for any web application.

In [another post](http://www.centurylinklabs.com/heroku-on-docker/), I wrote about the [building](https://github.com/CenturyLinkLabs/building) tool that uses Heroku Buildpacks to create Docker containers. Though the `building` tool made sense before Docker added the `ONBUILD` command to Dockerfiles, using a more pure Docker system gives you a few benefits.

1. You get more control of the images being built around your code because the operating systems are just Dockerfiles you can modify yourself if you don’t like them
2. You get to pick versions easier because there are native Docker tags for different versions of languages and frameworks
3. You get the benefit of community support as enhancements are contributed into the Docker community, you can take advantage of them easier if you are using Docker standards

So how do you create a Rails app and Dockerize it? Easy.

Let’s create a new Rails app:

```
$ rails new foobar
$ cd foobar
$ echo 'FROM rails:onbuild' > Dockerfile
$ docker build -t my-rails-app .
$ docker run -d -p 3000:3000 my-rails-app
$ curl 0.0.0.0:3000
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby on Rails: Welcome aboard</title>
    …
```

What does this do? To find out let’s examine the [rails:onbuild Dockerfile](https://github.com/docker-library/rails/blob/master/onbuild/Dockerfile):


```
FROM ruby:2.1.5

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

ONBUILD COPY Gemfile /usr/src/app/
ONBUILD COPY Gemfile.lock /usr/src/app/
ONBUILD RUN bundle install

ONBUILD COPY . /usr/src/app

RUN apt-get update && \
  apt-get install -y nodejs --no-install-recommends && \
  rm -rf /var/lib/apt/lists/*
RUN apt-get update && \
  apt-get install -y \
    mysql-client postgresql-client \
    sqlite3 --no-install-recommends && \
  rm -rf /var/lib/apt/lists/*

EXPOSE 3000
CMD ["rails", "server"]
```

So now it is clear which version of Ruby we are using, where the source code files are going, and how it runs our application.

What if you are on a Mac but need the Gemfile.lock to be tied to the Linux versions of libraries? There is a clever command you can run to do this:

```
$ docker run --rm -v "$(pwd)":/usr/src/app -w /usr/src/app ruby:2.1.5 bundle install
```

This starts a temporary container with a filesystem tied to your local directory so that the `Gemfile.lock` file that is generated will be written to your local filesystem, not just to your container’s filesystem. Neat!

How is using ONBUILD better than using the `building` tool? For one, you can customize this a lot easier. For example, if you want to run your Rails app using `thin` instead of `webrick`, you can overwrite the `CMD` in your app’s Dockerfile after adding the `thin` gem to your Gemfile:

```
$ echo 'CMD ["thin", "start"]' >> Dockerfile
$ docker build -t my-rails-app .
```

This kind of flexibility is what I love about using Docker.

---

Original source: [HOW TO DEPLOY A RAILS APP WITH DOCKER USING OFFICIAL IMAGES](http://www.centurylinklabs.com/how-to-deploy-a-rails-app-to-docker-using-official-images/)