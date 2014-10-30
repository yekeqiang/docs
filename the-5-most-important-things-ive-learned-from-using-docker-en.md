# The 5 Most Important Things I’ve Learned From Using Docker

---

##### Author:  John Regan 

---

![alt](http://resource.docker.cn/5things.jpg)


## Introduction

I’ve been using Docker for a little over a year, on several platforms – local Linux installs as well as cloud providers – and in that time I’ve learned a decent amount about how to manage my own images, build images flexible enough for any platform, and even a bit about writing my own, not-intended-for-Docker-at-all programs.

I’ve tried to consolidate these into five usable, digestible items, to keep in mind when starting a new Docker (or non-Docker!) project.

## 1. I need to be really, really specific and slightly paranoid when making images

I try to not run my applications as root. One nice thing about most distros: they usually create a system user when you install any type of service. For example, nearly every distro creates some type of `http`, `apache`, or `www-data` user when Apache is installed.

I decided to make an image for ejabberd, from source – and part of my build script would create an `xmpp` system user. And like many people, I used Docker’s handy Automated Builds service, and set my image to automatically rebuild when the base ubuntu image was updated.

I had an error in my image though: instead of saying `FROM ubuntu:12.04` I simply said `FROM ubuntu` – so one day, my image auto-updated and switched to Ubuntu 14.04, which added a new default system user. This caused my
application’s UID to increase by one. I pulled my latest ejabberd
build and it failed to start since the ejabberd user was unable to read
files (since I use a volume to store ejabberd’s files).

This has led to me to do two things now:

1. I use a specific distro version tag for any image I build.
2. I write start-up scripts for every application.

Those start-up scripts normally launch as root and do a few things:

- They make sure needed config files exist in the first place – you never know when they’ll be replaced by some empty volume!
- They chown the configuration and data files to the user I’ll run my app as.

This has probably saved me countless hours of frustration. I’m very specific with what my images are based off of, and I don’t just use my main application as the image’s `ENTRYPOINT` – I write some type of script to make sure the
environment is sane.

## 2. I have no way of knowing what capabilities somebody’s system will have

Until recently, I ran strictly the latest and greatest version of Docker on Ubuntu Linux. However, after trying out some cloud providers I’ve learned a few things:

- Somebody else might not be running the latest version of Docker.
- Somebody else might not have every capability of Docker made available to them.
- Somebody else might not have root access to the system running Docker.

This has drastically changed how I build my Docker images – I no longer write instructions like, “you have to launch a container using `--volumes-from`” or “this requires a linked container named `DB`” – I don’t know what another user might be using to run my images! I try and go out of my way to make my images as flexible as possible. If my image needs a MySQL database, I’ll work with a linked container, or an environment variable telling me where to connect to, and so on.

This adds a lot of work up front, but I think it’s very worth it. Now I can do all sorts of kooky things, like run an ambassador MySQL container that connects to an actual MySQL database, and use the link option to use a specific hostname when connecting. It’s pretty neat!

## 3. Dockerfiles might be a pain at first, but I love ‘em now

There are two ways to create a Docker image. On one hand I can spawn a container and build it interactively. Once my container is running the way I’d like it to, I can tag that container and use it as an image.

This is an easy way to build a container. I’ll be prompted when a package has an interactive step for setup. I can just edit configuration files with my preferred text editor. It’s a very simple process.

Dockerfiles on the other hand, can be something of a pain. What happens when a package prompts me for something? How do I edit files non-interactively? It’s a tricky process; there’s no way to make a part of it interactive – everything has to be entirely automated.

I’ve found the benefits outweigh the cons, though. I can’t count how many times I finally have a program running just the way I like, but can’t remember exactly what I did. A Dockerfile on the other hand? That shows exactly what I did. It’s all right there; even better, I can place it under version control. Yes, it requires more work up front to build an image, but it’s the only way I build Docker images now.

## 4. Spawning processes requires care – whether I use Docker or not!

It’s not unusual for an application to spawn a child process; I do it in my own programs all the time. On most systems I can spawn a process and read its output, check its exit code, whatever I need to do – and let the init system handle cleaning up when the process exits. For years, I’ve been writing programs this way without so much as a second thought.

In many cases, a Docker container will not be running an init system – and any spawned processes will hang around as zombie processes consuming resources. I’ve learned how to properly monitor and destroy a process within my own programs, just in case somebody ever decides to run my program in their own Docker image. You never know when somebody will take a program you’ve written and do something new with it!

## 5. One task per container doesn’t have to mean one process per container

This is a thought that’s met with some controversy. Go into the #docker
channel on Freenode and ask for recommendations on process supervision inside of containers; you’ll get a variety of different responses. Nearly everybody agrees that containers should perform one task, but there’s a lot of disagreement on whether that means one process or not.

I’ve come to the conclusion that using a process supervisor (like supervisord, runit, or s6) is completely acceptable as long as I determine what my task is, and only run services needed for that task. This is especially useful for web applications, where I often need specific rewrite rules for Apache or Nginx.

For example, if I have a web application that requires PHP-FPM, Nginx, Cron, and MySQL – I’ll run PHP-FPM+Nginx+Cron in one container, but keep MySQL in another container. I also make sure that if a key process exits or crashes, the process supervisor follows suit and quits as well. This preserves the normal Docker behavior of quitting when the main process quits.

## Conclusion

I sincerely hope my lessons learned will be useful to those of you getting into Docker (and those of you already pretty deep into it, too). Working with containers has changed how I build software, and working with different Docker platforms has changed how I build my containers.

If you have anything you’ve learned from using Docker, I’d love to hear about it from you! Use the comment section below to share your stories and tips, and any other feedback you may have, and thanks for reading!

---

Original source: [The 5 Most Important Things I’ve Learned From Using Docker](http://blog.tutum.co/2014/10/28/the-5-most-important-things-ive-learned-from-using-docker/)

