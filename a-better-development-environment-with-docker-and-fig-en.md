# A Better Development Environment with Docker and Fig

---

##### Author: Cameron Maske

---

Local development environments can be a nightmare. Have you ever run into a scenario where something works on your machine but not on your colleagues'? Having to spend time debugging platform-specific problems is a frustrating productivity sink.

Before we explore how to set up our stress free development environment, let's consider what key elements it should have.

- Isolated. No other project should affect it and it shouldn't affect any other projects. For example, if one project uses Python 2.7, that shouldn't affect another project which uses Python 3.4.
- Repeatable. Write it once and have it work each time. I don't want to have to spend time tweaking it for each user.
- Be as close to production as possible.

All that sounds great. But how do we achieve this?

Enter Docker. If you are unfamiliar with what Docker is, Solomon Hykes (founder and CTO of Docker) gives a great [introduction talk](http://youtu.be/Q5POuMHxW-0?t=2m18s).

Docker lends itself perfectly to our 3 requirements above. Docker was built to solve the problem of "run it once, run it everywhere". Docker containers are isolated, portable and repeatable.

But building, booting and managing containers can take a fair few complex shell commands.

The missing link is Fig. Fig describes itsself as a tool to create [fast, isolated development environments](https://github.com/docker/fig/) using Docker.

Fig moves all the configuration required to orchestra Docker into a simple clear `fig.yml` file. It handles all the work of building and running containers, forwarding their ports, sharing volumes, and linking them.

Let's explore Fig by example, and let's make it challenging.

I want a project with TWO databases, `Postgres 9.1` and `ElasticSearch 1.1`. I want `Redis 2.8.3` for caching, and I'll be running my main site through a Python-powered Flask app.

Before we set this up in Fig, imagine setting this up locally. How long would it take?

Thought experiment over; let's dive into an example. All code featured is available in this [GitHub repo](https://github.com/TrackMaven/using-fig).

## An example

Let's start with the `fig.yml`.

```
web:
    build: .
    command: python app.py
    volumes:
        - ./code
    ports:
        - "5000:5000"
    links:
        - es
        - db
        - redis
es:
    image: dockerfile/elasticsearch
    volumes:
        - elasticsearch:/data
    ports:
        - "9200:9200"
        - "9300:9300"
db:
    image: orchardup/postgres
    ports:
        - "5432:5432"
    environment:
        POSTGRESQL_DB:test_db
redis:
    image: trackmaven/redis
    ports:
        - "6379:6379"
```

In less then 30 lines of yaml, I've declared our stack.

With [Fig installed](http://www.fig.sh/install.html) we can start our stack by running `fig up`.

Click to watch the [demo](http://resource.docker.cn/slight-zesty-glass-frog.webm) 。

Let's break down what our simple command above did into 2 stages.

Building stage.

```
$ docker pull orchardup/postgres
$ docker pull trackmaven/redis
$ docker pull dockerfile/elasticsearch
$ docker build -t demo_web -rm=True web
```

Fig goes through each service, then, if needed, builds an image.

Building is based off a [Dockerfile](http://docs.docker.com/reference/builder/) that contains all the required steps to produce our desired image.

```
# Our base image is Ubuntu 13.04
FROM stackbrew/ubuntu:raring
# Install any updates
RUN apt-get -y update
# Ensure python is installed and postgres drivers.
RUN apt-get install -y python python-pip python-psycopg2

WORKDIR /code
ADD requirements.txt requirements.txt
RUN pip install -r requirements.txt
```

In our example project, our [Dockerfile](https://github.com/TrackMaven/using-fig/blob/master/web/Dockerfile) builds on top of an Ubuntu base. It ensures Python and pip are installed before installing our project-specific Python packages from requirements.txt.

Alternatively, you can point services towards a pre-built image. This can exist either locally, and be referenced by a tag or a partial image ID, or remotely, on Docker's [public registry](https://registry.hub.docker.com/).

Building a container off a Dockerfile works best for a service that is dependent on code within your project. In this case, the core logic for our Python-powered app is in our source control. In contrast, images work best for services that don't rely as heavily on project-specific code (e.g. the datastores).

Once Fig is satisfied all the required images are built or pulled, it's then time to run the services...

Running stage.

```
$ docker run --rm --name demo_es_1 -v "elasticsearch:/data" -p "9200:9200" -p "9300:9300" dockerfile/elasticsearch
$ docker run --rm --name demo_db_1 -p "5432:5432" -e "POSTGRESQL_DB:test_db" orchardup/postgres
$ docker run --rm --name demo_redis_1 -p "6379:6379" trackmaven/redis
$ docker run --rm --name demo_web_1 -v "web:/code" -p "5000:5000" --link es:demo_es_1 --link db:demo_db_1 --link redis:demo_redis_1 demo_web python app.py
```

### Equivalent docker commands preformed by Fig

Each service has a variety of run time options, including...

- `volumes`: Allows you to share folders between your host machine and the services' docker containers. In our example, the `web` folder is shared, so any code changes to the contained files are immediately updated on the running container.
- `ports`: Exposes ports between the host (you) and the container (service).
- `environment`: Allows you to set environment variables for containers. In our example, this allows us to configure the name of our database, [due to a clever run script](https://github.com/orchardup/docker-postgresql/blob/master/run).
- `links`: [Allows inter-service communication](http://www.fig.sh/env.html). In our example, `web` needs to know the ip and ports for the `redis`, `db` and `es` services. Fig ensures those boot up first and then injects a set of environment variables to web which include the IP address and various ports of the linked services.

To stop the running services you kill them with `Ctrl+C` or run `fig stop` in another terminal window.

Behind the scenes, Fig has used Docker to build and then run containers for each service. Running `fig up` again will restart the previously created containers whose volume changes (files, folders, etc) persist between boots. This means that if you have added data to one of the data stores it will still be present between restarts. You can remove the containers associated with services, and thus any volume changes, by running `fig remove [SERVICE]`.

After a successful build of a service Fig won't attempt to rebuild that service on the next up. That means if you want to make any changes to a built service's `Dockerfile`, you'll need to tell Fig to `rebuild` that image by using `fig build [SERVICE]`.

Fig also gives you the ability to run one-off commands in services using `fig run [SERVICE] [COMMAND]`, e.g. `fig run web python`.

### That's it!

For more about what Fig can do, I'd suggest checking out the [documentation](http://www.fig.sh/index.html).

## Tips we've learnt along the way.

- Try to use images when you can. Each layer is pulled in parallel, resulting in a quick download. An image is also frozen. Without careful version pinning, re-building images can change over time. Try to move as many non-project specific steps out of your Dockerfiles into new base images. For example, in our web service, the python and pip installation steps are good canditates to be contained in base images.

- Across teams, try to all use the same version of Docker and Fig. Both are moving fast and not insusceptible to version differences.

---

Original source: [A Better Development Environment with Docker and Fig](http://engineroom.trackmaven.com/blog/a-better-development-environment-with-docker-and-fig/)