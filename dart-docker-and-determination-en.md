# Dart, Docker, and Determination

---

##### Author: Matthew Butler

---

![alt](http://resource.docker.cn/docker-dart.png)

Dart recently released official [Docker Images for Dart](http://news.dartlang.org/2014/09/docker-images-for-dart-now-available.html). Docker is not something I’ve looked closely at in the past however I thought that this would make a good excuse to do so. If you’re not sure what Docker is, I’d recommend first checking out the [official description](https://www.docker.com/whatisdocker/).


I have a number of components for my stack that I’d like to containerize. Nginx to serve static content and act as a reverse proxy, a Mongo database and of course my Dart server-side code. Looking at it I immediately know that Nginx really can’t be containerized until after my Dart code is in its own container. This is because inter-container communication is much easier than communication from a container to the host system. Secondly I know that the MongoDB is pretty well isolated and would probably be easiest to get running, especially with the [Official MongoDB repository](https://registry.hub.docker.com/_/mongo/).

Getting the container installed and running was extremely easy. But it did leave me with one problem, how do I import my existing data, and how do I persist my database after it has been imported? One of the most difficult things to keep in mind when using Docker containers initially was that they are completely ephemeral. By default they do not persist anything saved to disk between shutdowns. Looking at the README information for the Docker repository only mentions that the mongo port is exposed. It does not mention anything about the database location. Looking closer at the [Mongo Dockerfile](https://github.com/docker-library/mongo/blob/master/2.7/Dockerfile#L23) I do see that it sets up a volume for /data/db. So now I know where I can map my local file system to, in order to persist the data.

So I run my Mongo image with a command similar to this:

```
sudo docker run --name mongodb -P -v /path/to/mongo_data/db:/data/db -d mongo
```

This does the job of naming my container mongodb for easier access, and mounts a local path for the database to store information into. True I should probably have used a [Data Volume container](https://docs.docker.com/userguide/dockervolumes/), but this was quicker and easier to get set-up as a start with Docker. Unfortunately this still doesn’t inject my existing data into the container, so that’s my next step.

I have a full [mongodump](http://docs.mongodb.org/manual/reference/program/mongodump/) of my database already backed up. I could have locally installed a MongoClient to connect to the docker container to restore the database, however that seemed to defeat the purpose of containerizing my mongo database in the first place. So instead I decide to create another container from the mongo image. I use the following to create the new container, mounting a directly which contains my mongo `dump/` directory.

```
sudo docker run -it --name mongo_client -v /path/to/mongo_data:/data --link mongodb:mongodb mongo /bin/bash
```

That’s a hefty number of arguments, but lets me run a mongo container interactively putting me into a bash shell. It also loads my directory with the mongo `dump/` directory into `/data` within my container. Now within the container I can run [mongorestore](http://docs.mongodb.org/manual/reference/program/mongorestore/) specifying the host as `mongodb` (the alias for the mongodb named container passed in the link argument). Docker automagically resolves the IPs and all that fun stuff for me to allow the easy connection into my main mongo server container and imports all the documents into the database properly.

As a final test, I modify my local script to access the mongo database via the docker IP for that container (Note the -P option when I ran the docker container which exposes all ports to the host as well. Without that, it would expose the ports to other containers but not the host). A quick check verifies it is able to view all the data and access the database as expected. Additionally, checking in `/path/to/mongo_data/db` reveals that the database documents are being persisted to the host file system correctly as well.

A side note, this did not all happen on my first try. I initially tried mounting just the /data directory in the mongodb container assuming that any subdirectories would also persist to the local storage, however that was not the case, and though it created the db subdirectory on the host system, it was empty and data did not persist between restarts until I used the method above when running my container.

Next task, is to containerize my Dart server side code. First I make some modifications to my code, such as pointing mongo to connect to the mongodb hostname I set on that container earlier. Additionally, trying to use the dart-runtime image, I need to make some minor modifications to the server-side code like the filename, and port it binds to. I create the very basic Dockerfile and try to build… and I almost immediately receive an error that it can’t find one of the dependencies my Dart app uses. In this case it specifies the directory on my local machine where it resides.

The dependency in question is a path dependency for my project which is provided via an absolute path. Unfortunately the [dart-runtime](https://registry.hub.docker.com/u/google/dart-runtime/) first copies the current app to the docker container and then tries to run `pub get` within the container. Since the path dependency is not copied over, it can’t find it. Looking through the [Dockerfile documentation](https://docs.docker.com/reference/builder/#add) I find that:

> The <src> path must be inside the context of the build; you cannot ADD ../something /something, because the first step of a docker build is to send the context directory (and subdirectories) to the docker daemon.

This basically means any files I want to add to a container must be a sub-directory of where the Dockerfile is located. That also means I won’t be able to use the dart-runtime image, and I will need to go up a level to the [google/dart](https://registry.hub.docker.com/u/google/dart/) image. First step is to look at the [Dockerfile used by the dart-runtime](https://github.com/dart-lang/dart_docker/blob/master/runtime/Dockerfile) image, as this will be close to what I want to achieve without rolling my own too much. I notice it includes a dart_run.sh script which also includes options for debugging and passing additional DartVM options. So I decide I’ll include that as well to keep things consistent with the dart-runtime script. There’s also an explanation in the google/dart image as to why pub get needs to be run twice.

Keeping all of this in mind, I create a new directory for building my container. Within it I copy my path dependency into a subdirectory, and I copy my main app into another subdirectory. I modify the `pubspec.yaml` to look for the path dependency relatively as opposed to an absolute path. I then copy the dart_run.sh into the root of the new directory and add in a Dockerfile. My directory layout looks something like this:

```
app_container /
app_container / dart_run.sh
app_container / Dockerfile
app_container / my_app /
app_container / my_dependency /
```

I also slightly modify dart_run.sh to point into into the my_app directory instead of trying to run the default /app/bin/server.dart location. My Dockerfile remains close to that used by the dart-runtime image, however I add several commands to add the pubspec.yaml and pubspec.lock for both my_app and for my_dependency. I then use the [WORKDIR](https://docs.docker.com/reference/builder/#workdir) command to enter both directories and run pub get first on the dependency then on the main app. Then add the rest of the files for each directory. My final Dockerfile looks something like this:

```
FROM google/dart

# From google/dart-runtime
ADD dart_run.sh /dart_runtime/
RUN chmod 755 /dart_runtime/dart_run.sh
RUN CHOWN root:root /dart_runtime/dart_run.sh

WORKDIR /app
ADD my_dependency/pubspec.yaml /app/my_dependency/
ADD my_dependency/pubspec.lock /app/my_dependency/
ADD my_app/pubspec.yaml /app/my_app/
ADD my_app/pubspec.lock /app/my_app/

WORKDIR /app/my_dependency
RUN pub get

WORKDIR /app/my_app
RUN pub get

WORKDIR /app
ADD . /app

WORKDIR /app/my_dependency
RUN pub get

WORKDIR /app/my_app
RUN pub get

CMD []
ENTRYPOINT ["/dart_runtime/dart_run.sh"]
EXPOSE 8080 8181 5858
```

Once this is in place I’m finally able to build my image and run a container! It’s possible a couple of the steps may be a little redundant or more than strictly required, for instance I’m not sure if I really need to run pub get on both main app and dependency twice. But better safe than sorry. When running the container I make sure to link in the mongodb container, and verify that the database connectivity is in place.

My last step is to get nginx up and running in a container. Fortunately, once again, there is an [official nginx image](https://registry.hub.docker.com/_/nginx/) to use. So I pull that down. The documentation on this image in the repository is a little more detailed than that for the mongodb image. In particular they mention that `/usr/share/nginx/html` is a volume which can be mounted for static content. They also indicate that `/etc/nginx/nginx.conf` can be mounted to easily provide a custom configuration without the need to rebuild the container each time you update it. Both of these are features I’ll really need, that way I can update the static files anytime I want without needing to rebuild the container, and I can add additional sites for nginx to act as a reverse proxy relatively easily.

My run command looks along these lines, which links my web directory for the client-side files, my nginx.conf which contains the information for proxying requests to the dart server-side for non-static content, and which links the two containers together.

```
sudo docker run --name nginx-con -P -v /path/to/static/web:/user/local/nginx/html:ro -v /path/to/nginx.conf:/etc/nginx.conf:ro --link my_app:myapp -d nginx
```

I changed the `my_app` container name to be referenced as myapp in order to ensure there were no issues with using myapp as the URL from the nginx.conf file for proxying. I also ensured to pass the -P option to expose the ports on the host machine as well. My proxy_pass in my nginx.conf can now look a something like this:

```
location /my_app/ {
    root   html;
    rewrite ^/my_app/(.*) /$1 break;
    try_files $uri @proxy
}

location @proxy {
    proxy_pass http://myapp:8080;
    proxy_http_version 1.1;
}
```

With that, I now have 3 docker containers running, each linked together as appropriate but also isolated services which can easily be reproduced, and updated independent of each other. My dev machine is also much cleaner without services that aren’t needed outside of development running. Where appropriate, persistence is achieved on the local filesystem as opposed to within the container(s), and configuration updates are readily possible without rebuilding the container (with the exception of the Dart container).

---

Original source: [Dart, Docker, and Determination](http://blog.butlermatt.me/2014/09/dart-docker-and-determination/)