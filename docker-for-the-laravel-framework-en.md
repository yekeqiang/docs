# Docker for the Laravel framework 

---

##### Author: Dylan Lindgren

---

There's a lot of buzz about [Docker](https://www.docker.com/) at the moment, and rightly so. It's a huge leap forward in the world of app containerisation as far as usability is concerned, bringing it out of giant data centers of the likes of Google and Facebook, and into the hands of the masses of developers and sysadmins.

Docker uses native Linux kernel features to containerise processes, which is akin to putting them in their own little silo where they can't speak with the other processes on the system. Docker also has features for deployment, management, and automated building of these containers. Containerisation leads to a similar level of isolation as a virtual machine, without needing to run it on top of a [hypervisor](http://en.wikipedia.org/wiki/Hypervisor) and taking the 10-15% performance hit doing so involves. Watch the talk the Docker founder and CTO gives below for a quick overview of Docker. You can also check out [Docker.com](https://www.docker.com/) if you want a bit more background.


As most of the applications I develop are in Laravel, I wanted to see how I could make use of Docker to have a local development environment which entirely mirrors my production one. As you're only deploying the application itself and its dependencies (like Nginx), the risks to security and stability are greatly reduced compared to an entire virtual machine. You still however get the time savings of deploying your development environment into production.

There's a few articles on the internet about combining Laravel and Docker. All of the ones I've read take the concepts the author learned from using [Vagrant](https://www.vagrantup.com/) and place them straight into Docker, that is, running all the processes inside a single container. For a number of reasons this means you're [missing out](https://blog.docker.com/2014/06/why-you-dont-need-to-run-sshd-in-docker/) on some of Docker's real advantages. We want to have one container for one process, and we will link each container (i.e. process) to a "data-only" container where all our app's files will be stored. Let's give it a go!

## Preparing your development environment

Docker uses containerisation technology implementations exclusive to Linux (e.g. namespaces, [cgroups](http://en.wikipedia.org/wiki/Cgroups), [UnionFS](http://en.wikipedia.org/wiki/UnionFS)), so if we're developing on OS X or Windows we need to run it within a virtual machine. The Docker software package for non-Linux operating systems is called *[Boot2Docker](https://github.com/boot2docker/boot2docker)*.

As at time of writing, the Boot2Docker installers don't support mapping directories on the host inside the virtual machine they create, meaning we can't use data on our host inside our Docker containers. This is a must, luckily however this feature is currently in development and has made its way into the GitHub repo. I built an ISO image which we will download as part of the installation process that will give us this functionality.

### Docker on OS X

Start by downloading and installing the [Docker for OS X Installer](http://docs.docker.com/installation/mac/). After that's complete download my [boot2docker.iso](https://drive.google.com/file/d/0B1Iz2vYLOLiHZExrQUNqeGZXZkU/view?usp=sharing) and place it inside your `~/.boot2docker/` directory (replacing the `boot2docker.iso` that's already in there).

Now, lets open a terminal window and run a few commands. The first one will initialise our Docker virtual machine.

```
$ boot2docker init
```

Then we will map our `/Users/dylan/myapp` directory inside it (where we're going to keep our app's data) as `/data`, using the `boot2docker` ssh command.

```
$ VBoxManage sharedfolder add boot2docker-vm --name myapp --hostpath /Users/dylan/myapp
$ boot2docker ssh 'sudo mkdir /data'
$ boot2docker ssh 'sudo mount -t vboxsf -o "defaults,uid=33,gid=33,rw" myapp /data'
```

Lets check and see if the directory was mapped properly.

```
$ boot2docker ssh 'ls -l /data'
otal 0  
drwxr-xr-x    1 root     root           136 Sep 25 08:34 logs  
drwxr-xr-x    1 root     root           612 Sep 26 10:57 www 
```
 
Yep!

Lastly we will forward port 80 on our host to the virtual machine's port `80`.

```
$ VBoxManage modifyvm boot2docker-vm --natpf1 "web,tcp,,80,,80"
```

Now we're ready to start the virtual machine. Run the below commands.

```
$ boot2docker up
$ export DOCKER_HOST=tcp://$(boot2docker ip 2>/dev/null):2375
```

You only need to run that second command if you're prompted to set your `DOCKER_HOST` environment variable (but running it regardless won't hurt).

If at any point you want to stop boot2docker you can run boot2docker down.

### Docker on Windows

Start by downloading and installing the [Docker for OS X Installer](http://docs.docker.com/installation/mac/). After that's done download my [boot2docker.iso](https://drive.google.com/file/d/0B1Iz2vYLOLiHZExrQUNqeGZXZkU/view?usp=sharing) and place it inside your `C:\Program Files\Boot2Docker` for Windows directory (replacing the `boot2docker.iso` that's already in there).

Run the program `Boot2Docker Start` from your start menu. This will create the Docker virtual machine based on our ISO file. Once it finishes loading you'll be dropped into a terminal session on the virtual machine. We want to do a bit of configuration so lets stop the virtual machine entering the command `sudo poweroff`.

Open a new command prompt window and cd to the directory `C:\Program Files\Oracle\VirtualBox`.

The first thing we want to configure is to map the `C:\Users\dylan\myapp` directory into the virtual machine as `/data`.

```
> VBoxManage sharedfolder add boot2docker-vm --name myapp --hostpath C:\Users\dylan\myapp
```

We also want to forward port `80` on our host to the virtual machine's port `80`.

```
> VBoxManage modifyvm boot2docker-vm --natpf1 "web,tcp,,80,,80"
```

You can now close the command prompt.

Open `Boot2Docker Start` again. Lets finish mapping our host folder and check to make sure it worked.

```
$ boot2docker ssh 'sudo mkdir /data'
$ boot2docker ssh 'sudo mount -t vboxsf -o "defaults,uid=33,gid=33,rw" myapp /data'
$ boot2docker ssh 'ls -l /data'
otal 0  
drwxr-xr-x    1 root     root           136 Sep 25 08:34 logs  
drwxr-xr-x    1 root     root           612 Sep 26 10:57 www  
```

Yep!

You can shutdown the virtual machine at any time by running the `sudo poweroff` command in this window.

### Docker on Linux

If you're developing on Linux, you've got it easy because you don't need to bother with any virtual machines, and port forwarding or host directory mapping to it. You can run Docker natively! Just follow the instructions appropriate for your distribution in the [Docker documentation](http://docs.docker.com/installation/#installation).

## An Overview

To get the a core Laravel app up and running we not only need a web server that can process PHP, we also need to be able to run the PHP command line applications composer and artisan. There's more processes you will probably use (e.g. [Bower](http://bower.io/)) but this should be a good baseline for getting started using Laravel with Docker. Each of these processes has their own container.

Here's a list of the Docker images we will use:

- **[dylanlindgren/docker-laravel-data](https://github.com/dylanlindgren/docker-laravel-data)** - This image will be used to create our ["data-only" container](http://www.offermann.us/2013/12/tiny-docker-pieces-loosely-joined.html), which will be used to give access to our app's files to our other containers.
- **[dylanlindgren/docker-laravel-composer](https://github.com/dylanlindgren/docker-laravel-composer)** - This image will be used to create a container that allows us to run composer commands
- **[dylanlindgren/docker-laravel-artisan](https://github.com/dylanlindgren/docker-laravel-artisan)** - This image will be used to create a container that allows us to run artisan commands
- **[dylanlindgren/docker-laravel-phpfpm](https://github.com/dylanlindgren/docker-laravel-phpfpm)** - PHP-FPM for processing our PHP files.
- **[dylanlindgren/docker-laravel-nginx](https://github.com/dylanlindgren/docker-laravel-nginx)** - An Nginx web server. This container will link to the PHP-FPM one above.
- **NEW! [dylanlindgren/docker-laravel-bower](https://github.com/dylanlindgren/docker-laravel-bower)** - [Bower](http://bower.io/)

Having separate containers for artisan and composer is a real advantage for us, as we can choose to only push the `docker-laravel-data`, `docker-laravel-nginx`, and `docker-laravel-phpfpm` containers to production when we go live without the possibility of breaking anything!

I made a flowchart which visualises how all the containers fit together, along with where they get their data from, and where it's mounted inside the containers.

![alt](http://resource.docker.cn/docker-and-laravel-flowchart.png)

You can see that all the containers are getting their `/data` folder from the `docker-laravel-data` container, which in-turn is getting its `/data` folder from the host directory `~/myapp.` Inside this `~/myapp` folder we will have two directories:

- `www` - Contains our applications files (e.g. `public/index.php`)
- `logs` - Access and error log files for Nginx

I've published all the images listed above on [Docker Hub](https://hub.docker.com/) so they're easy to download. Run the below command to pull them all into your boot2docker virtual machine.

```
$ docker pull dylanlindgren/docker-laravel-data && \
  docker pull dylanlindgren/docker-laravel-composer && \
  docker pull dylanlindgren/docker-laravel-artisan && \
  docker pull dylanlindgren/docker-laravel-phpfpm && \
  docker pull dylanlindgren/docker-laravel-nginx && \
  docker pull dylanlindgren/docker-laravel-bower
```

The images are also linked above on GitHub, and can be built using the docker build command, however that's outside the scope of this tutorial.

## Docker & Laravel In Practice

I use a late-2013 MacBook Pro for development, so all the instructions below are tailored for an OS X environment. It should be pretty easy to change a few paths here and there to work with Linux or Windows however.

### Creating a data-only container

Create the folders `~/myapp/www` and `~/myapp/logs` on your host. The `~/myapp` folder will be mapped to our "data-only" container and all the other containers will get access to your app's data through this container.

If you already have a Laravel app, put all its files in the `~/myapp/www` folder. Otherwise we'll create the Laravel app later.

Lets create our "data-only" Docker container, and map the directory we just created to `/data` inside the container:

```
docker run --name myapp-data -v /Users/dylan/myapp:/data:rw dylanlindgren/docker-laravel-data  
```

### Running composer commands

To run `composer` commands you would run the Docker container like so:

```
docker run --privileged=true --volumes-from myapp-data --rm dylanlindgren/docker-laravel-composer *your composer commands here*  
```

Woah! That's a really long command! Who wants to be typing all that just to run a simple `composer dump-autoload`? Bash aliases to the rescue! Simply edit your `.bashrc` file and add the below.

```
alias myapp-composer="docker run --privileged=true --volumes-from myapp-data --rm dylanlindgren/docker-laravel-composer" 
```
 
Restart your terminal session. You can now run `composer` commands by running `myapp-composer`!

If you're creating a new Laravel app, run the below command to use Composer to download Laravel and it's dependencies:

```
myapp-composer create-project laravel/laravel /data/www --prefer-dist  
```

Don't forget to give the appropriate permissions to Laravel's `app/storage` folder, otherwise you may get errors later on.

### Running `artisan` commands

`artisan` commands are run in the same way as we run `composer` commands.

```
docker run --privileged=true --volumes-from myapp-data --rm dylanlindgren/docker-laravel-artisan *your artisan commands here*  
```

So lets add another line to our `.bashrc` file

```
alias myapp-artisan="docker run --privileged=true --volumes-from myapp-data --rm dylanlindgren/docker-laravel-artisan"  
```

Restart your terminal session once again. You can then run artisan commands by running myapp-artisan.

### Serving Laravel

Both Nginx and PHP-FPM are seperate processes, and thus we will put each one in its own container for the reasons previously explained.

First, lets create the PHP-FPM container. Note the use of the -d switch which means the process will be run in the background. The PHP-FPM process doesn't run and then exit like the `composer` and `artisan` commands do - it just keeps running. So we want to run it as a daemon in the background so we can do other things, like launch our Nginx container!

So let's run PHP-FPM:

```
docker run --privileged=true --name myapp-php --volumes-from myapp-data -d dylanlindgren/docker-laravel-phpfpm 
```
 
We will use the `--link` switch when we create our Nginx container to link it to this PHP-FPM container and allow them to communicate over IP (port 9000 to be precise).

Lets run Nginx:

```
docker run --privileged=true --name myapp-web --volumes-from myapp-data -p 80:80 --link myapp-php:fpm -d dylanlindgren/docker-laravel-nginx  
```

And finally if we open our web browser and go to `http://localhost` we will see our Laravel welcome page!

![alt](http://resource.docker.cn/we-have-arrived.png)

## Conclusion

Hopefully from running through this tutorial with me you can really see not only how easy Docker is, but how it could be a huge game changer for Laravel development.

If you have any feedback or questions, feel free to leave a comment below, or you can contact me on Twitter with [@dylanlindgren](https://twitter.com/dylanlindgren) or email with [dylan.lindgren@gmail.com](mailto:dylan.lindgren@gmail.com).

---

Original source: [Docker for the Laravel framework](http://www.dylanlindgren.com/docker-for-the-laravel-framework/)