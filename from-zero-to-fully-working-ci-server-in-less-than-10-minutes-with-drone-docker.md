#From zero to fully working CI server in less than 10 minutes with Drone & Docker

What is Drone? The official description is: "Drone is a Continuous Integration platform built on Docker".

What is Docker? Again, official description is :"Docker is an open-source project to easily create lightweight, portable, self-sufficient containers from any application".

10 minutes. This is all you need. In fact, there is an extra buffer in those 10 minutes to get your configuration working.

10 minutes is by order of magnitude less time than it would take you to get a Jenkins server up & running. I have never been a fan of Jenkins, and probably won't ever be, but let's keep this for another post, maybe.

REQUIREMENTS

We will assume that you have a Ubuntu 13.04 (64 bit) server with a routable IP or address. On Digital Ocean, which I love (referral here, thanks!), you can even have a VPS pre-built with Docker, 0.8 as of time of writing.

You can also use a Ubuntu 12.04 64 bit box in theory for Drone.

Your application needs to be hosted on GitHub. BitBucket coming soon apparently, but for now, this works only with GitHub which is fine by me. I love GitHub.

STEP 0: INSTALL DOCKER (OPTIONAL, SKIP IF DOCKER IS ALREADY INSTALLED)

On your Ubuntu machine, run this:

1
curl -s https://get.docker.io/ubuntu/ | sudo sh
You can verify the installation with:

1
sudo docker run -i -t ubuntu /bin/bash
STEP 1: INSTALL DRONE

This is very simple to install!

You gotta do:

1
2
wget http://downloads.drone.io/latest/drone.deb
sudo dpkg -i drone.deb
The README also adds "sudo start drone" but it was already started for me.

To finish the installation, navigate to http://my-server-ip-or-addr:80/install and follow the steps in the wizard (account creation). Once logged in, keep that browser tab open.

STEP 2: REGISTER A NEW APPLICATION ON GITHUB

You now have Drone up and running, but we need to configure GitHub access so that it can setup hooks to build your code after code push or pull requests. First step for that is to register a new application on GitHub. It is done right here.

Pick up a name you, it could be "my Drone server" or whatever makes sense to you
Set the homepage URL to http://my-server-ip-or-addr/
The description is up to you, just like the name
The authorization callback URL should be set to http://my-server-ip-or-addr/auth/login/github
STEP 3: GET YOUR GITHUB CLIENT ID & SECRET CONFIGURED IN DRONE

After the registering your new app on GitHub, you were given a client ID and secret, copy them in the "GitHub OAuth Consumer Key and Secret" section of http://my-server-ip-or-addr/account/admin/settings

STEP 4: LINK YOUR PROJECT

Click on the "New Repository" button (which will get you to http://my-server-ip-or-addr/new/github.com), click "Link Now", accept and then once on the "Repository Setup" page of Drone, you get to fill the repository details (GitHub owner + repository name).

Boom! Your project is almost ready to be built. You need only one final step now...

STEP 5: CONFIGURE YOUR .DRONE.YML

The .drone.yml file is the configuration file for the build steps, services, notifications, etc.

Here is a simple example for a Ruby on Rails project:

1
2
3
4
5
6
7
8
9
10
11
12
13
image: ruby2.0.0
script:
  - cp config/database.drone.yml config/database.yml
  - bundle install
  - psql -c 'create database test;' -U postgres -h 127.0.0.1
  - bundle exec rake db:schema:load
  - bundle exec rspec spec
services:
  - postgres
notify:
  email:
    recipients:
      - email@example.com
You are wondering what that database.drone.yml file contains? See this gist.

There is a variety of images for Go, Python, Haskell, PHP, Scala, Node, etc. Drone provides us with official images but you are not restricted to only those.

There is also a bunch of services available to you.

All the details on how to configure your .drone.yml file can be found in the README and drone.io's documentation.

When you are done, commit, push and watch the build. Rinse & repeat until your .drone.yml is working just fine.

For your information, I noticed that the first build takes a few minutes.

EXTRA, EXTRA!

Here a few things that you might want to consider that Drone offers you:

Email notifications (SMTP settings at the bottom of http://my-server-ip-or-addr/account/admin/settings)
SSL is available to you and you should really consider it (at the top of http://my-server-ip-or-addr/account/admin/settings)
HipChat notifications
Continuous deployment is also available to you
CONCLUSION

That's it, it just works. Should you use that instead of Jenkins, Codeship or other hosted CI as a Service? It's up to you. This project was just publicly released a week ago so I am not sure that I would use that for my job's code just yet, but I will pay close attention to this.

Let me know if you like it or not!

PS: if you love Docker and Heroku-like PaaS, you might also be interested by my post about getting Dokku (you're very own Heroku-like setup) installed.


http://jipiboily.com/2014/from-zero-to-fully-working-ci-server-in-less-than-10-minutes-with-drone-docker
