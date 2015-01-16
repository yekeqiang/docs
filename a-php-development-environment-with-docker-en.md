# A PHP development environment with Docker

---

##### Author: Geoffrey Bachelet

---

A lot of developers nowadays use Vagrant to manage their development environment through Virtual Machines. Vagrant is awesome, but VMs have a number of drawbacks (mainly, it uses a lot of resources). With the emergence of container technologies, and more particularily Docker, we have a simple solution to this problem.

## Disclaimer

Because of the way `boot2docker` works, the approach described in this post will not work out of the box. Extra care need to be taken when you want to share folders to a Docker container in a non-Linux environment, and the issue actually deserves a whole blog post that I will write later.

## What is a good development environment

First, we need to define what we consider a good development environment. For me, it has the following qualities:

1. Disposable. I must be able to trash the environment and spawn a new one at will.
2. Fast to start. When I want to work, I want to work now.
3. Easy to update. Things go very fast in our industry, and I must be able to update my development environment with new version of software easily.

Docker allows all of this, and a bit more. You can destroy a container and re-create it almost instantly, and updating the environment is just a matter of rebuilding the image(s) you’re using.

## What is a PHP development environment

Given the complexity of today’s web applications, a PHP development environment can be a lot of things. To keep things simple, we are going to limit ourselves. Our environment will be able to run a Symfony project through Nginx and PHP5-FPM, with MySQL. Running CLI commands inside a running container is a bit more complicated, so we’ll keep that for another post too.

## Pet vs Cattle

Another decision we have to make upfront is whether our development environment will be single or multi-container. There are advantages in both:

- single-containers are easier to pass around, and manipulate, because they are self-contained. Everything runs in the same container, much like a VM. It also means that every time you want to upgrade something (new version of PHP for example), you need to rebuild the whole container.
- multi-containers allows for greater modularity when adding components to your stack, because each container contains a part of the stack: web, php, mysql, etc. You can scale up each service individually, easily add services without having to rebuild everything, etc.

Because I’m lazy, and I need to keep some content for my book, we’ll only see the single-container approach in this post.

## Initializing a project

First thing is to initialize a new Symfony project. The recommended way to do this is to use [composer](http://getcomposer.org/) with the `create-project` command. We could install composer on our workstation, but that would be too easy, so instead we’re going to use it through docker too.

Luckily, I published an article on how to do just that: [make docker commands](http://geoffrey.io/making-docker-commands.html) (Ok I’m cheating, I initially wrote it as part of this very article, then decided that it would make a good standalone).

Anyway, go read it, then create yourself a `composer` alias if don’t have one already:

```
$ alias composer="docker run -i -t -v \$PWD:/srv ubermuda/composer"
```

You can now initialize a Symfony project like a boss:

```
$ composer create-project symfony/framework-standard-edition SomeProject
```

Awesome. Give yourself a high-five, get a cup of coffee or whatever is your liquid drug of choice, and get ready for the real work.

## The container

Building a self-sufficient container to run a standard Symfony project is pretty easy. All we have to do is to install our usual stack of nginx, php5-fpm and mysql-server, throw in a pre-made nginx vhost configuration, tweak some config files, and we’re good to go.

All source code for this container is available at GitHub in the [ubermuda/docker-symfony](https://github.com/ubermuda/docker-symfony) repository. The Dockerfile is the configuration file used by Docker to build an image, let’s review it:

```
FROM debian:wheezy

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -y
RUN apt-get install -y nginx php5-fpm php5-mysqlnd php5-cli mysql-server supervisor

RUN sed -e 's/;daemonize = yes/daemonize = no/' -i /etc/php5/fpm/php-fpm.conf
RUN sed -e 's/;listen\.owner/listen.owner/' -i /etc/php5/fpm/pool.d/www.conf
RUN sed -e 's/;listen\.group/listen.group/' -i /etc/php5/fpm/pool.d/www.conf
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

ADD vhost.conf /etc/nginx/sites-available/default
ADD supervisor.conf /etc/supervisor/conf.d/supervisor.conf
ADD init.sh /init.sh

EXPOSE 80 3306

VOLUME ["/srv"]
WORKDIR /srv

CMD ["/usr/bin/supervisord"]
```

We start with extending the debian:wheezy base image, then proceed to configure nginx and php5-fpm with a series of sed commands:

```
RUN sed -e 's/;daemonize = yes/daemonize = no/' -i /etc/php5/fpm/php-fpm.conf
RUN sed -e 's/;listen\.owner/listen.owner/' -i /etc/php5/fpm/pool.d/www.conf
RUN sed -e 's/;listen\.group/listen.group/' -i /etc/php5/fpm/pool.d/www.conf
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
```

We do two things here. First, configuring php5-fpm and nginx to run in the foreground so supervisord can keep track of them later. Then we configure php5-fpm to run with the user as the web server, to avoid a few issues with file permissions.

With that taken care of, we can install a couple of configuration files. First, nginx’ vhost configuration file, `vhost.conf`:

```
server {
    listen 80;

    server_name _;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    root /srv/web;
    index app_dev.php;

    location / {
        try_files $uri $uri/ /app_dev.php?$query_string;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        include fastcgi_params;
    }
}
```

We set the `server_name` to `_` because we don’t need any (it’s a bit like perl’s `$_` placeholder var), and configure the document root to be /srv/web, so we’ll want to deploy the application to `/srv` later. The rest is standard nginx + php5-fpm configuration.

We need supervisord (or any other process manager, but I like supervisord better), because a container can only run one process at a time. Luckily, this process can be a process manager that will spawn all the processes we need! Here’s the configuration for supervisord, with a small twist:

```
[supervisord]
nodaemon=true

[program:nginx]
command=/usr/sbin/nginx

[program:php5-fpm]
command=/usr/sbin/php5-fpm

[program:mysql]
command=/usr/bin/mysqld_safe

[program:init]
command=/init.sh
autorestart=false
redirect_stderr=true
redirect_stdout=/srv/app/logs/init.log
```

What we do here is define all our services, plus a special `program:init` process that is not an actual service, but rather a hackish way to run an init script.

The problem with the init script is it often requires some services to be running already. For example, you might want to initialize a few database tables, but you need MySQL running for that. One possible solution could be to start MySQL during the init script, initialize the tables, then stop MySQL to avoid interfering with supervisord’s process management and then run supervisord.

Such a script would look like something like this:

```
/etc/init.d/mysql start
app/console doctrine:schema:update --force
/etc/init.d/mysql stop

exec /usr/bin/supervisord
```

It’s a bit ugly. So intead of doing that, we tell supervisor to run our init script and to never restart it.

Here’s our actual `init.sh` script:


```
#!/bin/bash

RET=1

while [[ RET -ne 0 ]]; do
    sleep 1;
    mysql -e 'exit' > /dev/null 2>&1; RET=$?
done

DB_NAME=${DB_NAME:-symfony}

mysqladmin -u root create $DB_NAME

if [ -n "$INIT" ]; then
    /srv/$INIT
fi
```

It starts by waiting for MySQL to start, then create a database taking the name from the `DB_NAME` environment variable, defaulting to `symfony`. Then it looks for a script to run in the `INIT` environment variable and tries to run it. We’ll see at the end of this article how to use these variables.

## Building and running the image

With everything in place, we now need to build our Symfony Docker image using `docker build`:

```
$ cd docker-symfony
$ docker build -t symfony .
```

And you can now use it to run your Symfony project:

```
$ cd SomeProject
$ docker run -i -t -P -v $PWD:/srv symfony
```

That’s a bunch of options, let’s review what each does:

- `-i` enables interactive mode, that is, `STDIO`s are attached to your current terminal. This is useful to receive logs and send signals to the process running inside the container.
- `-t` creates a virtual `TTY` for the container. Basically, it’s `-i`’s best friend and you’ll often use (or not use) them together.
- `-P` tells the Docker daemon to publish all exposed ports. In our case, that will be the port 80.
- `-v $PWD:/srv` mounts the current directory into the container’s /srv directory. Mounting a directory makes its content available at the target mount point.

Now you you might remember the `DB_NAME` and `INIT` environment variables we talked about earlier, and wonder what they’re for. They are used to customized your environment. Basically, you can set environment variables in your containers with `docker run`’s `-e` option, and they will be catched up by the init script.

So if you want to name your database `some_project_dev`, you’d run the container like this:

```
$ docker run -i -t -P -v $PWD:/srv -e DB_NAME=some_project_dev symfony
```

The `INIT` variable is even more powerful since it allows you to specify a script to be run at startup. For example, you could have a `bin/setup` script that runs `composer install` and setups the database schema:

```
#!/bin/bash
composer install
app/console doctrine:schema:update --force
```

And have it run using the -e option again:

```
$ docker run -i -t -P \
	-v $PWD:/srv \
	-e DB_NAME=some_project_dev \
	-e INIT=bin/setup
```

Note that you can use -e multiple times in `docker run`, so that’s pretty cool. Also your init script needs to be executable (using `chmod +x`).

We can now check that everything works as expected by `curl`ing the container. First, we need to retrieve the public port that Docker mapped to the container’s port 80, let’s use docker port for that:

```
$ docker port $(docker ps -aql 1) 80
0.0.0.0:49153
```

The `docker ps -aql 1` command is a handy shortcut to retrieve the last container’s id. In our example, Docker mapped the container’s port 80 to port 49153, let’s curl this!

```
$ curl http://localhost:49153
You are not allowed to access this file. Check app_dev.php for more information.
```

We get Symfony’s default error message for when you’re trying to access the dev controller not from localhost. This is perfectly normal, since we’re not curling from inside the container. You can safely remove those lines from the `web/app_dev.php` front controller:

```
// This check prevents access to debug front controllers that are deployed by accident to production servers.
// Feel free to remove this, extend it, or make something more sophisticated.
if (isset($_SERVER['HTTP_CLIENT_IP'])
    || isset($_SERVER['HTTP_X_FORWARDED_FOR'])
    || !(in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', 'fe80::1', '::1')) || php_sapi_name() === 'cli-server')
) {
    header('HTTP/1.0 403 Forbidden');
    exit('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
}
```

These are the lines that prevent access to the dev controller from anywhere else than localhost. You can now retry our curl and check that it works, or even point your browser at [http://localhost:49153/:](http://localhost:49153/)

![alt](http://resource.docker.cn/result.png)

It works!

Ok that was easy. We can now start environments very quickly, and update them easily, but there’s still a lot of place for improvement. Next time, we’ll see how to run commands inside a running container, stay tuned!

---

Original source: [A PHP development environment with Docker](http://geoffrey.io/a-php-development-environment-with-docker.html)

