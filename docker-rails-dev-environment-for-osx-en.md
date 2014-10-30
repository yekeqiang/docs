# Rails Development environment for OS X using Docker

---

##### Author: Andrew Allen

---

## Intro

I'm not going to go into the details of what Docker is in this post. There are already a lot of great explanations out there of what it is and why you might want to be using it. When I was looking to set up a virtual development environment hosting a Rails app with Docker, I had trouble finding a good guide that covered setup from from to back, so I'm documenting my findings here. If you have suggestions or improvements, please let me know in the comments.

I’ll be using a Rails app as an example, but this process can work equally well with any other technology stack.

We'll assume a common configuration for a moderately complex Rails application and we'll throw in background processing using Sidekiq to make things interesting. We'll be breaking our app out into the following containers:

- Web server
- Postgres DB
- Redis Instance
- Sidekiq process

## Dependencies

Let’s go ahead and get our dependencies out of the way.

### Install VirtualBox

At the base we need VirtualBox to manage our virtual operating system. I use homebrew and cask to install it for simplicity and easy updates. You can also find traditional installers on their site.

```
$ brew tap caskroom/cask
$ brew install brew-cask
```
```
$ brew cask install virtualbox
```

### Install Docker

Next, we need to install Docker. Luckily, Docker provides a very nice binary for installing it on OS X called boot2docker. Unfortunately, the homebrew package doesn't seem to be well-maintained. Instead, you can find a link to download it on their [installation page](https://docs.docker.com/installation/mac/).

After running the installer, we need to get boot2docker initialized:

```
 $ boot2docker init
 $ boot2docker start
 $ $(boot2docker shellinit)
```

After running `boot2docker start`, you'll see a list of environment variables including `DOCKER_HOST`. You should add these to your ~/.bashrc file so that Docker will be available in new terminal sessions.

### Install fig

Fig is a python utility that orchestrates multi-container projects. It has been made an official Docker project so you can expect continued updates and expanded functionality in its future. Even though it’s written in python, it will work for any type of project, including Rails in our case.

To install it, run the following curl command:

```
curl -L https://github.com/docker/fig/releases/download/1.0.0/fig-`uname -s`-`uname -m` > /usr/local/bin/fig; chmod +x /usr/local/bin/fig  
```

> *Note that this will install Fig 1.0, which is the most recent version as of this writing (10/24/14). Check out their [installation page](http://www.fig.sh/install.html) for the most up to date instructions.*

## Adding Fig & Docker to our Sample App

We'll need to make a few changes to our app's configuration to make it compatible with Docker. Luckily, the changes are pretty minimal.

### The Dockerfile

We'll start with the Dockerfile, which describes how to build our virtual environment to host a Rails app. I've added comments to describe what's going on. For more information on the commands and Dockerfiles in general, check out their [reference page:](https://docs.docker.com/reference/builder/).

```
# Choose a Ruby 2.1.2 image as our starting point
FROM ruby:2.1.2

# Run updates
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

# Set up working directory
RUN mkdir /myapp  
WORKDIR /myapp

# Set up gems
ADD Gemfile /myapp/Gemfile  
ADD Gemfile.lock /myapp/Gemfile.lock  
RUN bundle install

# Install foreman
RUN gem install foreman

# Finally, add the rest of our app's files
ADD . /myapp
```
  
### fig.yml

Fig lets us define our containers and how they should be linked together with a simple yaml file.

```
db:  
  image: postgres
  ports:
    - "5432"

redis:  
  image: redis
  ports:
    - "6379"

web:  
  build: .
  command: foreman s
  volumes:
    - .:/myapp
  ports:
    - "3000:3000"
  links:
    - db
    - redis
```

Several interesting and slightly magical things are happening here:

1. The db and redis containers are pulling their image from the central Docker registry at [http://hub.docker.com](http://hub.docker.com).

2. The web container, on the other hand, is building it's image using the Dockerfile we just created. This is denoted by the . which means, "look in the current directory for a Dockerfile".

3. The db and redis containers expose the default Postgres and Redis ports while the web container maps all incoming calls on port 3000 to its internal port 3000, where we'll be running the Rails server.

4. The web container will be mounting the current Rails directory at the path /myapp. This will allow us to make changes in OS X using whatever editor and workflow we're used to, while syncing those changes to the virtualized container.

5. Now for the real magic: the web container will be linked to the db and redis containers. This means we'll be able to make calls to these separate containers over our virutal OS's internal network. I'll cover the details of setting up these connections next.

For further reference on the fig.yml syntax, check out: [http://www.fig.sh/yml.html](http://www.fig.sh/yml.html).

### database.yml

```
default: &default  
  adapter: postgresql
  encoding: unicode
  username: postgres
  host: <%= ENV['BLOG_DB_1_PORT_5432_TCP_ADDR'] %>
  port: <%= ENV['BLOG_DB_1_PORT_5432_TCP_PORT'] %>
  pool: 5

development:  
  <<: *default
  database: demo_development

test:  
  <<: *default
  database: demo_test
```

Notice the environment variables. These are coming from Fig and are shared among linked instances. To get a list of the environment variables your service is exposing, you can run `fig run SERVICE env`, so to see the database environment variables, we could run `fig run db env`.

[http://www.fig.sh/env.html](http://www.fig.sh/env.html)

```
$ fig run web env
```

### Sidekiq initializer

```
if ENV[‘BLOG_REDIS_1_PORT']  
    redis_url = "redis://#{ENV[‘BLOG_REDIS_1_PORT_6379_TCP_ADDR']}:#{ENV[‘BLOG_REDIS_1_PORT_6379_TCP_PORT']}"

  Sidekiq.configure_server do |config|
    config.redis = {
        namespace: "sidekiq",
        url: redis_url
    }
  end

  Sidekiq.configure_client do |config|
    config.redis = {
        namespace: "sidekiq",
        url: redis_url
    }
  end
end  
```

Same deal with the environment variables. To see what the redis service is exposing, we run fig run redis env.

### Javascript Runtime

```
gem 'therubyracer',  platforms: :ruby  
```

## Running your new development environment

First, we'll build any containers that are defined with Dockerfiles, in this case, our web container.

```
$ fig build
```

We can use fig run to run the normal Rails setup commands:

```
$ fig run web rake db:create
```
```
$ fig run web rake db:migrate
```

> *Note that the first time we run something that requires a container defined by an image, it will take a few minutes to download and run that image*

Finally, we can use fig up to bring up the whole environment.

```
$ fig up
```

We can now visit our site by using boot2docker's ip address. We can find this by running boot2docker ip.

## Working with Fig

Fig provides a really nice interface to interact with our dockerized processes. For more info on what else you can do with fig, check out their [documentation page](http://www.fig.sh/cli.html).

One quick tip here: you can use `fig run SERVICE /bin/sh` to get a quick shell into your container. Great for debugging and sanity checking what's actually inside the container. You can use a similar approach to get a Rails console running: `fig run web rails c`.

## Drawbacks

This approach of running your Rails development environment inside a virutal environment isn't without it's drawbacks, however.

The main issue is speed. There's a pretty heavy virtualization tax, even if you devote most of your machine's resources to VirtualBox. Additionally, if you have a very asset-heavy Rails app, you may see considerably slow load times. I had to disable asset debugging on one particularly asset-heavy app to get remotely asdf which means I need to have my machine prepared to run the Rails server locally whenever I need to work on the frontend. Not exactly ideal.

I'd love to hear any suggestions on improving load time in cases like this, because that's the one thing stopping me from using this approach full time.

PS: If you found this helpful, you should [follow me on Twitter](http://twitter.com/allenan_) for more content like this.

---

Original source: [Rails Development environment for OS X using Docker](http://allenan.com/docker-rails-dev-environment-for-osx/)