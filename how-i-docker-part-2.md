# How I Docker, Part 2

---

##### Author: Barett Clark

---

Yesterday I laid out a high level overview of [how I use Docker](http://cookingco.de/2014/10/27/how-i-docker/). I neglected to point out 1 thing — the environment variables. I exposed the `POSTGRES_USER` and `POSTGRES_PASSWORD`, but didn’t explicitly call out what to do with those in the context of the Rails app.

*database.yml*

```
default: &default
  adapter: postgresql
  encoding: unicode
  pool: 5
  host: <%= ENV['POSTGRES_PORT_5432_TCP_ADDR'] || 'db' %>
  username: <%= ENV['POSTGRES_USER'] || 'developer' %>
  password: <%= ENV['POSTGRES_PASSWORD'] || 'password' %>
 
development:
  <<: *default
  database: rails_4_1_6_development
 
test:
  <<: *default
  database: rails_4_1_6_test
 
production:
  <<: *default
  database: rails_4_1_6_production
```

Docker exposes some variables for you based on the containers and their names. You can also specify your own with that `-e` switch.

Alright. With that little bit cleared up, let’s talk about how I actually use all this now.

## How I Actually Use All This Now

Remember the intent of this exercise was to come up with a way to easily share an app with it’s dependencies in some nice tidy package such that nobody else needs to understand how to line up all those dependencies and deal with the maintenance.

Docker is great, but one of the things that I liked about Vagrant was how easily provisioning and reprovisioning happens. But I don’t really need all the overhead of some big complicated thing that requires 3 separate licenses and all that. We have boot2docker, which handles the virtualization on OSX with VirtualBox. It’s maybe not the most amazing software, but it hasn’t left a bad taste in my mouth yet.

I like the simplicity of Docker, and I like the simplicity of just using the tooling that Docker provides us.

So to simplify the workflow (using Docker and Boot2Docker) I wrote a bunch of rake tasks. Feel free to comment or submit updates.

```
namespace :boot2docker do
  desc 'Start VM from any states'
  task :start do
    %x(boot2docker start > /dev/null 2>&1)
  end
 
  desc 'Setup shell'
  task :shellinit => [:start] do
    # NOTE: this does not set the env, so we have to do something different
    # %x($(boot2docker shellinit > /dev/null 2>&1))
    env = %x(boot2docker shellinit 2> /dev/null)
    env.split(/\n/).each do |line|
      env, setting = line.gsub(/^.* /, '').split(/\=/)
      ENV[env] = setting
    end
  end
 
  desc 'Gracefully shutdown the VM'
  task :stop do
    %x(boot2docker stop)
  end
 
  desc 'Install nsenter'
  task :install_nsenter => [:shellinit] do
    system("boot2docker ssh '[ -f /var/lib/boot2docker/nsenter ] || docker run --rm -v /var/lib/boot2docker/:/target jpetazzo/nsenter'")
  end
 
  desc 'SSH into the given container'
  task :ssh, [:container] => :install_nsenter do |t, args|
    container_hash = %x(docker ps | grep #{args[:container]} | awk '{ print $1 }')
    puts "SSH into container: #{container_hash}"
    exec "boot2docker ssh -t sudo /var/lib/boot2docker/docker-enter #{container_hash}"
  end
end
 
namespace :docker do
  namespace :maintenance do
    desc 'Stop running containers'
    task :stop, [:container] => 'boot2docker:shellinit' do |t, args|
      # rake docker:maintenance:stop[foo]
      if args[:container]
        puts "Stopping #{args[:container]}"
        %x(docker ps -a | grep #{args[:container]} | awk '{print $1}' | xargs docker stop)
      else
        puts "Stopping ALL"
        %x(docker stop $(docker ps -aq))
      end
    end
 
    desc 'Remove one or more containers'
    task :rm, [:container] => [:stop] do |t, args|
      if args[:container]
        puts "Removing #{args[:container]}"
        %x(docker ps -a | grep #{args[:container]} | awk '{print $1}' | xargs docker rm)
      else
        puts "Removing ALL"
        %x(docker rm $(docker ps -aq))
      end
    end
 
    desc 'Remove one or more images'
    task :rmi, [:container] => [:stop, :rm] do |t, args|
      %x(docker rmi $(docker images -a | grep "^<none>" | awk '{print $3}') > /dev/null 2>&1)
    end
  end
 
  namespace :fig do
    desc 'Use fig to build the Dockerfile image'
    task :build => ['boot2docker:shellinit'] do
      ENV['FIG_PROJECT_NAME'] = File.dirname(__FILE__).split(/\//).last.downcase
      system "fig build"
    end
 
    desc 'Use fig to run the app as described in fig.yml'
    # TODO: clean out old images
    task :up => :build do
      system "fig up"
    end
  end
 
  namespace :info do
    desc 'List all the container (docker ps -a)'
    task :ps => 'boot2docker:shellinit' do
      system("docker ps -a")
    end
 
    desc 'List all the images on localhost (docker images)'
    task :images => 'boot2docker:shellinit' do
      system("docker images")
    end
 
    # rake docker:hostname[rails-basic-app,3000]
    # open $(rake docker:info:hostname[railsbasic_web_1,3000])
    desc 'Give the hostname and port for a container'
    task :hostname, [:container, :port] => 'boot2docker:shellinit' do |t, args|
      require 'json'
      docker_host = ENV['DOCKER_HOST'].match(/\/(\d*.\d*.\d*.\d*):/).to_a.last
      json        = JSON.parse(%x(docker inspect #{args[:container]})).first
      if json["NetworkSettings"]["Ports"]
        port        = args[:port]
        host_port   = json["NetworkSettings"]["Ports"]["#{port}/tcp"].first["HostPort"]
        puts "http://#{docker_host}:#{host_port}"
      else
        puts "No docker container instance found for #{args[:container]}"
      end
    end
  end
end
 
namespace :git do
  desc 'Stash files'
  task :stash do
    %x(git stash save)
  end
 
  desc 'Pull the freshest code'
  task :pull, [:remote, :branch] do |t, args|
    # NOTE: when used as a dependency this is pretty sloppy
    remote = args[:remote] || 'origin'
    branch = args[:branch] || 'master'
    %x(git pull --rebase #{remote} #{branch})
  end
end
 
task :default => 'docker:fig:up'
```

## The Rakefile

So what does this buy us, this giant pile of rake tasks? This gives us a thin wrapper around some pretty basic functionality. Rake also has the ability to chain things together, so you can ask to clean up all the old instances and it will make sure they’re all stopped and that boot2docker is even running.

*It makes is so that we can pass this around and have a simplified workflow*.

Sometimes boot2docker loses it’s mind and quits on you, and you don’t even have to care about that anymore because the rake tasks will make sure boot2docker is running for you.

Here are all the rake tasks that I currently have in place:

```
$ rake -T
rake boot2docker:install_nsenter # Install nsenter
rake boot2docker:shellinit # Setup shell
rake boot2docker:ssh[container] # SSH into the given container
rake boot2docker:start # Start VM from any states
rake boot2docker:stop # Gracefully shutdown the VM
rake docker:fig:build # Use fig to build the Dockerfile image
rake docker:fig:up # Use fig to run the app as described in fig.yml
rake docker:info:hostname[container,port] # Give the hostname and port for a container
rake docker:info:images # List all the images on localhost (docker images)
rake docker:info:ps # List all the container (docker ps -a)
rake docker:maintenance:rm[container] # Remove one or more containers
rake docker:maintenance:rmi[container] # Remove one or more images
rake docker:maintenance:stop[container] # Stop running containers
rake git:pull[remote,branch] # Pull the freshest code
rake git:stash # Stash files
```

## Generalized Yet Specific

So we have `Dockerfiles` and environment variables that we can pass into containers. We have `boot2docker` to handle the virtualization for us on OSX. We have `rake` to manage `docker` and `boot2docker`. But what about apps and their dependencies?

## Fig

We need a DSL that is a little bit generalized, but not as generalized as Chef or Puppet. We just need to be able to tell Docker how to do all those things that were in my little container startup script. Enter Fig.

Whereas before we had *dockerup.sh*

```
docker run -d \
  --name rails-basic-postgres \
  -e POSTGRES_USER=docker -e POSTGRES_PASSWORD=docker \
  barrettclark/postgis
 
docker build -t="barrettclark/rails-basic:devel" .
 
docker run -d -P \
  --name rails-basic-app \
  --link rails-basic-postgres:postgres \
  -e POSTGRES_USER=docker -e POSTGRES_PASSWORD=docker \
  -v /Users/barrettclark/temp/rails_basic/project:/rails \
  barrettclark/rails-basic:devel
```

We now have *fig.yml*

```
db:
  image: barrettclark/postgis
  ports:
    - "5432"
  environment:
    - POSTGRES_USER=docker
    - POSTGRES_PASSWORD=docker
web:
  build: .
  environment:
    - POSTGRES_USER=docker
    - POSTGRES_PASSWORD=docker
  volumes:
    - ./project:/rails
  links:
    - db
  ports:
    - "3000"
```

You’ll note in the `Rakefile` that the default rake task is `docker:fig:up`, which will first run the `docker:fig:build` task (and also make sure boot2docker is running and the environment is set).

## Putting It All Together

So let’s run this app, shall we?

```
$ rake
db uses an image, skipping
Building web...
---> d90ac1ef8a16
Step 1 : RUN apt-get update
---> Using cache
---> 8e44373a6c4e
Step 2 : RUN apt-get upgrade -y
---> Using cache
---> 7c8a817248f8
Step 3 : RUN apt-get -y --fix-missing install libpq-dev nodejs
---> Using cache
---> 0fd7db2ace6d
Step 4 : RUN gem install bundler --no-ri --no-rdoc
---> Using cache
---> d653711e99ef
Step 5 : ADD start.sh start.sh
---> Using cache
---> b5298535c564
Step 6 : RUN chmod +x /start.sh
---> Using cache
---> 51f55dde7927
Step 7 : ADD project/Gemfile Gemfile
---> Using cache
---> 49f2911395c8
Step 8 : ADD project/Gemfile Gemfile.lock
---> Using cache
---> f1f6b6cf4863
Step 9 : RUN bundle install
---> Using cache
---> 73271fb83da5
Step 10 : RUN rm /Gemfile*
---> Using cache
---> be0105be5b00
Step 11 : ADD project/ /rails
---> ff9da0621df5
Removing intermediate container b18ed235b2a5
Step 12 : WORKDIR /rails
---> Running in 045c488e22c0
---> 4c85897ee553
Removing intermediate container 045c488e22c0
Step 13 : EXPOSE 3000
---> Running in 0909d1f73082
---> 43df967060e7
Removing intermediate container 0909d1f73082
Step 14 : CMD /start.sh
---> Running in 8c515adc0285
---> 69d3e7832b58
Removing intermediate container 8c515adc0285
Successfully built 69d3e7832b58
Creating railsbasic_db_1...
Creating railsbasic_web_1...
Attaching to railsbasic_db_1, railsbasic_web_1
web_1 | *** STARTING RAILS APP ***
web_1 | rm: cannot remove /rails/tmp/pids/server.pid': No such file or directory
db_1 | 2014-10-28 15:00:14 UTC LOG: database system was shut down at 2014-10-17 16:06:21 UTC
db_1 | 2014-10-28 15:00:14 UTC LOG: autovacuum launcher started
db_1 | 2014-10-28 15:00:14 UTC LOG: database system is ready to accept connections
web_1 | Don't run Bundler as root. Bundler can ask for sudo if it is needed, and
web_1 | installing your bundle as root will break this application for all non-root
web_1 | users on this machine.
web_1 | Fetching gem metadata from https://rubygems.org/..........
web_1 | Using rake 10.3.2
web_1 | Using i18n 0.6.11
web_1 | Using json 1.8.1
web_1 | Using minitest 5.4.2
web_1 | Using thread_safe 0.3.4
web_1 | Using tzinfo 1.2.2
web_1 | Using activesupport 4.1.6
web_1 | Using builder 3.2.2
web_1 | Using erubis 2.7.0
web_1 | Using actionview 4.1.6
web_1 | Using rack 1.5.2
web_1 | Using rack-test 0.6.2
web_1 | Using actionpack 4.1.6
web_1 | Installing mime-types 2.3
web_1 | Using mail 2.6.1
web_1 | Using actionmailer 4.1.6
web_1 | Using activemodel 4.1.6
web_1 | Using arel 5.0.1.20140414130214
web_1 | Using activerecord 4.1.6
web_1 | Using coffee-script-source 1.8.0
web_1 | Installing execjs 2.2.1
web_1 | Using coffee-script 2.3.0
web_1 | Using thor 0.19.1
web_1 | Using railties 4.1.6
web_1 | Using coffee-rails 4.0.1
web_1 | Using hike 1.2.3
web_1 | Using multi_json 1.10.1
web_1 | Installing jbuilder 2.1.3
web_1 | Using jquery-rails 3.1.2
web_1 | Using pg 0.17.1
web_1 | Using bundler 1.7.4
web_1 | Using tilt 1.4.1
web_1 | Using sprockets 2.11.0
web_1 | Installing sprockets-rails 2.1.4
web_1 | Using rails 4.1.6
web_1 | Using rdoc 4.1.2
web_1 | Using sass 3.2.19
web_1 | Using sass-rails 4.0.3
web_1 | Using sdoc 0.4.1
web_1 | Using spring 1.1.3
web_1 | Using turbolinks 2.4.0
web_1 | Using uglifier 2.5.3
web_1 | Your bundle is complete!
web_1 | Usebundle show [gemname]` to see where a bundled gem is installed.
web_1 | == 20141004114903 CreateGrilledThings: migrating ==============================
web_1 | -- create_table(:grilled_things)
web_1 | -> 0.0099s
web_1 | == 20141004114903 CreateGrilledThings: migrated (0.0100s) =====================
web_1 |
web_1 | [2014-10-28 15:00:43] INFO WEBrick 1.3.1
web_1 | [2014-10-28 15:00:43] INFO ruby 2.1.2 (2014-05-08) [x86_64-linux]
web_1 | [2014-10-28 15:00:43] INFO WEBrick::HTTPServer#start: pid=25 port=3000
```

Then you can ask docker about the instance:

```
$ rake docker:info:hostname[railsbasic_web_1,3000]

http://192.168.59.103:49154
```

Even cooler, you can open that up in your browser with

```
open $(rake docker:info:hostname[railsbasic_web_1,3000])
```

And using a little bit on linux magic (nsenter) you can SSH into the running container:

```
$ rake boot2docker:ssh[railsbasic_web_1]
SSH into container: f52a7f422ec2
root@f52a7f422ec2:/#
```

## Summary

So that’s where my head is at the moment. Docker + boot2docker + rake + fig. That gives you a flexible system that is still specific enough. You can run multiple apps, and you can have multiple apps that have dependencies in each other. I’ve had a little more trouble getting this running with a Scala stack, so YMMV.

Again, I’m not saying that any of this is particularly correct. It’s just what has evolved for me in my particular team’s use case. I would love to hear feedback on what you think works best for you and your needs. I would also like to thank [Clifton King](https://twitter.com/cliftonk), who gave me a couple of docker-related tips along the way.

---

Original source: [How I Docker, Part 2](http://cookingco.de/2014/10/28/how-i-docker-part-2/)