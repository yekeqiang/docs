# Using Docker in Production

---

##### Author: Jeremy Derr

---


I aggressively switched [Djed Studios](http://djed.com/) to Docker earlier this year and couldn’t be happier with the result. It has been prominently front-and-center in our culture of continuous improvement and “failing fast”. Initially, I recreated the exact, pre-existing development environment into a monolithic Docker container. I then slowly evolved the workflow such that we now do everything “The Docker Way” – we have 1 service per container and build our service up atomically from there. The very same containers that we use for development are, ultimately, placed onto our production servers.

## Python Onbuild

The official [python onbuild](https://github.com/docker-library/python/blob/master/2.7/onbuild/Dockerfile) containers proved invaluable. For a few reasons, though, I have forked them for our own purposes. You can see the base of [this fork on github](https://github.com/jcderr/python). The only notable change is that it is now based on ubuntu instead of debian.

Our private python-onbuild containers use this change to install a handful of apt packages that are integral to our site. In keeping with the purpose of onbuild containers, I wanted this container to be everything needed to build or run our app except for the actual python environment and code.

This gives us a really simple Dockerfile:

```
FROM jcderr/python:2.7.8-onbuild

RUN ln -s /usr/src/app /opt/app; mkdir /opt/logs
RUN mkdir -p /usr/src/app/static && python manage.py collectstatic --noinput --clear
VOLUME [ "/usr/src/app", "/var/run" ]
CMD [ "python", "manage.py", "runserver", "0.0.0.0:80"]
```

The ‘onbuild’ part of this construct is that the base image uses the ONBUILD declaration to grab our requirements.txt file and install a full python environment, and then injects our code into /usr/src/app. By doing this is two stages, building the python environment can utilize Docker’s build cache if requirements.txt hasn’t changed and results in a much faster build.

## Fig

Fig 1.0 was [released](https://blog.docker.com/2014/10/fig-1-0-boot2docker-compatibility-and-more/) simultaneously with [Docker 1.3](https://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/). This was a key point for us, as it allowed us to retire Soos, my custom-designed shell glue for our developers’ environments. I had reviewed Fig at the time and found it lacking, but the fine folks at Orchard (and at Docker, after the acquisition) have done an incredible job in rapid iteration on the concept and produced an outrageously amazing product. The plan is to integrate it fully into a future release of Docker itself and I wholeheartedly endorse this effort. Fig is that awesome.

A shadow of Soos still persists in our project, mostly as a wrapper for common tasks in Fig, and we have two figfiles, fig.yml and fig-production.yml. By default, all bin/soos calls use fig.yml, which stands up a db, redis, and a Django dev server on port 8000, with all relevant links.

If someone calls bin/soos --prod [action], however, it will use the production figfile. For instance, bin/soos --prod up will stand up a copy of the entire production stack locally, including Nginx with SSL, a celery worker, and so on.

## Production

We deploy entirely within AWS with CloudFormation, and all of our instances are CoreOS Stable machines. Each instance has EC2 metadata to define what environment it belongs to (prod, stage, or test) as well as its role (django, celery, etc).

### Sharing Secrets

Securing and sharing secrets is one of the hard problems in operations. We chose to store these secrets in an S3 bucket and allow access via IAM profiles. Each environment has a matching environment file that defines a handful of necessary values, such as DATABASE_URL, in shell syntax.

### systemd

systemd isn’t for everyone, but it’s the init system in CoreOS, it gets the job done, and it solves my specific problems quite well, so I’ve embraced it wholeheartedly in our stack. We have global units that target all units that match specific metadata (eg. ‘role=django’, ‘role=celery’).

I abuse subshells pretty aggressively, though. One of our units is a fork of the CoreOS team’s example ELB Presence Notifier, altered to use IAM profiles instead of requiring access keys in the environment.

```
[Unit]
Description=ELB Presence Notifier
After=http.service
BindsTo=http.service

[Service]
User=core
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker stop elb-presence
ExecStartPre=-/usr/bin/docker rm elb-presence
ExecStartPre=/usr/bin/docker pull jcderr/s3fetch
ExecStartPre=-/usr/bin/docker run --rm -v /home/core:/root jcderr/s3fetch python /opt/app/s3fetch.py ************* .docker.cfg /root/.dockercfg
ExecStartPre=-/bin/sh -c '/usr/bin/docker run --rm -v /home/core:/root jcderr/s3fetch python /opt/app/s3fetch.py ************* $(/usr/bin/etcdctl get /djed-environment) /root/$(/usr/bin/etcdctl get /djed-environment)'
ExecStartPre=/usr/bin/docker pull djedstudios/elb-presence
ExecStart=/bin/sh -c '/usr/bin/docker run --name elb-presence --env-file=/home/core/$(/usr/bin/etcdctl get /djed-environment) djedstudios/elb-presence'
ExecStop=/usr/bin/docker stop elb-presence

[X-Fleet]
Global=true
MachineMetadata="role=django"
```

This service adds the instance to an AWS Elastic Load Balancer and, when the linked http.service process dies, remove it again. Process termination is low hanging fruit. There’s no point in waiting for the AWS health checker to take its time removing the instance if we know the process isn’t even running.

As you can see, I have a key I fetch out of etcd to make this work. djed-environment holds the full path to the environment file we fetched from S3. Other units also rely on djed-version, which will be the tag name for the docker container that should be run.

With similarly constructed systemd units, I stand up the service like this:

- Fetch the app container
- Run a copy of the app to serve the website
- Run another copy to serve the API
- Both run uwsgi on a Unix socket
- Fetch and run the nginx container
- Nginx picks up the app and API sockets as upstreams
- Fetch and run the presence notifier

We’re a Django shop. The only difference between the app container and the api container is that we pass in a different DJANGO_SETTINGS_MODULE to the api container.

## Deploying Updates

To deploy a new version of the code, I simply update the etcd key /djed-version. We have a self-healing/self-updating service called Norm that watches this key and, upon a change, begins the update process.

### Norm

Norm is a short bash script that has two modes of operation: maintenance and update.

In Maintenance Mode, Norm watches the fleet service to make sure that all systemd units in the stack are healthy. If they fail, it works through a regimen of stopping, unloading, and then relaunching said services.

In Update mode, one machine is chosen at random to be the ‘responsible adult’ and the others are instructed to pull the targeted version of the container (as defined by the /djed-version key). Once each machine concludes the download, they inform the responsible adult by updating a key that includes their machine ID. Once the entire fleet has the new container, the responsible adult stops the service, performs all maintenance required in the upgrade (eg. database migrations, static file generation), and then brings the service back up.

In the near future, we’ll be moving to a database migration scheme that will allow this update to take place without downtime, but we’re not quite there yet.

---

Original source: [Using Docker in Production](https://continuously.me/2014/11/05/using-docker-in-production/)