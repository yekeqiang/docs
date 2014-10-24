## A Simple Way to Dockerize Applications

---

##### Author: Jason Wilder

---

Dockerizing an application is the process of converting an application to run within a Docker container. While dockering most applications is straightforward, there are a few problems that need to be worked around each time.

Two common problems that occur during dockerization are:

- Making an application use environment variables when it relies on configuration files
- Sending application logs to STDOUT/STDERR when it defaults to files in the container’s file system

This post introduces a new tool: `dockerize` that simplifies these two common dockerization issues.

## The Problems

### Configuration

Many applications use configuration files to control how they work. Different runtime environments have different values for various sections of a file. For example, database connection details for a development environment would be different than a production environment. Similarly, API keys and other sensitive details would be different across environments.

There are a few ways to handle these environmental differences with docker containers:

- Embed all environment details in the image and use a control environment variable to indicate which file to use at run time. (e.g. APP_CONFIG=/etc/dev.config)
- Use volumes to bind mount the configuration data at run time
- Use wrapper scripts that modify configuration data with tools like sed that environment variable

Embedding all environment details is not ideal because environmental changes should not require a rebuild of an image. It’s also less secure since sensitive data such as API keys, login credentials, etc.. for all environments are stored in the image. Comprimising a development environment could leak production details. Having these kinds of details in any image should really be avoided.

Using volumes keep these details out of the image but makes deployment more complicated since you can no longer just deploy the image. You must also coordinate configuration file changes along with the image.

Injecting environment variables into custom files is not always trivial as well. You can sometimes craft a sed command or write some custom scripts to it but it’s repetitive work. This does produce an image that works well in a docker ecosystem though.

### Logging

Docker containers that log to STDOUT and STDERR are easier to troubleshoot, monitor and integrate into a [centralized logging](http://jasonwilder.com/blog/2012/01/03/centralized-logging/) system. Logs can be acessed directly with the `docker logs` command and through the docker logs API calls. There are also many tools that can automatically pull docker logs and ship them off if they log to STDOUT and STDERR.

Unfortunately, many applications log to one or more files on the file system by default. While this can usually be [worked around](http://jasonwilder.com/blog/2014/03/17/docker-log-management-using-fluentd/), it’s tedious to figure out the nuances of each applications logging configuration.

---

## Using Dockerize

[dockerize](http://github.com/jwilder/dockerize) is a small Golang application that simplifies the dockerization process by:

1. Generating configuration files using templates and the containers environment variables at startup
2. Tailing arbitrary log files to STDOUT and STDERR
3. Starting a process to run within the container

### An Example

To demonstrate how it works, we’ll walk through dockerizing a generic nginx container with `dockerize`. We start with:


```
FROM ubuntu:14.04

# Install Nginx.
RUN echo "deb http://ppa.launchpad.net/nginx/stable/ubuntu trusty main" > /etc/apt/sources.list.d/nginx-stable-trusty.list
RUN echo "deb-src http://ppa.launchpad.net/nginx/stable/ubuntu trusty main" >> /etc/apt/sources.list.d/nginx-stable-trusty.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C300EE8C
RUN apt-get update
RUN apt-get install -y nginx

RUN echo "daemon off;" >> /etc/nginx/nginx.conf

EXPOSE 80

CMD nginx
```

Next we’ll install `dockerize` and run `nginx` through it:


```
FROM ubuntu:14.04

# Install Nginx.
RUN echo "deb http://ppa.launchpad.net/nginx/stable/ubuntu trusty main" > /etc/apt/sources.list.d/nginx-stable-trusty.list
RUN echo "deb-src http://ppa.launchpad.net/nginx/stable/ubuntu trusty main" >> /etc/apt/sources.list.d/nginx-stable-trusty.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C300EE8C
RUN apt-get update
RUN apt-get install -y wget nginx

RUN echo "daemon off;" >> /etc/nginx/nginx.conf

RUN wget https://github.com/jwilder/dockerize/releases/download/v0.0.1/dockerize-linux-amd64-v0.0.1.tar.gz
RUN tar -C /usr/local/bin -xvzf dockerize-linux-amd64-v0.0.1.tar.gz

ADD dockerize /usr/local/bin/dockerize

EXPOSE 80

CMD dockerize nginx
```

Nginx logs to two different log files under `/var/log/nginx` by default. It would be nice have the nginx access and error log streamed to the console if your run this container interactively or if you `docker logs nginx` so you can see what’s happening.

We can fix that by passing `-stdout <file>` and `-stderr <file>` as command-line options. These can also be passed multiple times if there are several files to tail.

```
CMD dockerize -stdout /var/log/nginx/access.log -stderr /var/log/nginx/error.log nginx
```

Now when you run the container, nginx logs are available via `docker logs nginx`.

To demonstrate the templating, we’ll make this a into a more generic proxy server than can be configured using environment variables. We’ll define the environment variable `PROXY_URL` to be a URL of a site to proxy.

```
PROXY_URL="http://jasonwilder.com"
```

When the container is started with this variable, `dockerize` will use it to generate an nginx server location path.

Here’s the template:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.html index.htm;

    # Make site accessible from http://localhost/
    server_name localhost;

    location / {
      access_log off;
      proxy_pass {{ .Env.PROXY_URL }};
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Then our final Dockerfile would look like:

```
FROM ubuntu:14.04

# Install Nginx.
RUN echo "deb http://ppa.launchpad.net/nginx/stable/ubuntu trusty main" > /etc/apt/sources.list.d/nginx-stable-trusty.list
RUN echo "deb-src http://ppa.launchpad.net/nginx/stable/ubuntu trusty main" >> /etc/apt/sources.list.d/nginx-stable-trusty.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C300EE8C
RUN apt-get update
RUN apt-get install -y wget nginx

RUN echo "daemon off;" >> /etc/nginx/nginx.conf

RUN wget https://github.com/jwilder/dockerize/releases/download/v0.0.1/dockerize-linux-amd64-v0.0.1.tar.gz
RUN tar -C /usr/local/bin -xvzf dockerize-linux-amd64-v0.0.1.tar.gz

ADD default.tmpl /etc/nginx/sites-available/default.tmpl

EXPOSE 80

CMD dockerize -template /etc/nginx/sites-available/default.tmpl:/etc/nginx/sites-available/default -stdout /var/log/nginx/access.log -stderr /var/log/nginx/error.log nginx
```

The `-template <src>:<dest>` options indicates that the template at `/etc/nginx/sites-available/default.tmpl` should be generated and written to `/etc/nginx/sites-available/default`. Multiple templates can be specified as well.

Run this container with:

```
$ docker run -p 80:80 -e PROXY_URL="http://jasonwilder.com" --name nginx -d nginx
```

You can then access `http://localhost` and it will proxy to this site.

This is a simplistic example but it can easily be extended using the embedded `split` function and `range` statement to handle multiple proxy values or other options. There are also a few other [template functions](https://github.com/jwilder/dockerize#using-templates) available.

---

## Conclusion

While this example is somewhat simplistic, many applications need some shims to make them run well within docker. `dockerize` is a generic utility to help with this process.

You can find the code at [jwilder/dockerize](http://github.com/jwilder/dockerize).

---

Original source: [A Simple Way to Dockerize Applications](http://jasonwilder.com/blog/2014/10/13/a-simple-way-to-dockerize-applications/)