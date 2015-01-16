# Streamline SDLC with Docker using VisualOps 

---

##### Author: VisualOps


---

Docker brings tremendous value to many aspects of IT engineering. A major one is SDLC. During our recent brainstorming session, two ideas came up around this topic:

> Is it possible to deploy the same multi-node application on my laptop, which is identical to the prod in the cloud?

> How could I clone a running cluster from the cloud to my laptop?

In a hypervisor world, these scenarios are impractical, since running a bunch of VMs on a single laptop is a minefield. With Docker, this no longer seems to be an issue; however the problem with Docker is that a multi-container environment requires extra components for the service discovery, orchestration, etc. One will face quite some difficulty to get things setup and make sure the environment is consistent with the production. So, we decided to make a tool that just does it, without any manual work needed.

After the meeting, our Technical Project Lead — Thibault, built a quick prototype. Everyone in the team was thoroughly impressed by the demo, therefore we focused our resources and efforts into polishing and releasing this CLI tool.

## How it works

visualops is the command-line interface for VisualOps. By leveraging the Docker module, it allows you to deploy the same stack, or clone running apps to your laptop, for dev/testing/debugging purposes.

Let’s start with a [prebuilt stack of Wordpress and MySQL](http://store.visualops.io/#!docker-wordpress-us-east-1):

![alt](http://resource.docker.cn/docker-wordpress-dual-node.png)


### 1. Setup

```
$ pip install visualops
```

and login

```
$ visualops login
Enter usename or email:test
Your Password:
Login Success!
```

### 2. Launch the stack on your laptop

First, you need to pull the stack. A YAML file will be generated:

```
$ visualops stack pull stack-19defd22
Pulling stack-19defd22 from remote ….
stack-19defd22 is saved to /root/stack-19defd22.yaml
Done!
$ ls -l
total 4
-rw-r—r— 1 root root 554 Oct 15 08:40 stack-19defd22.yaml
```

To launch the stack on your laptop:

```
$ visualops stack run stack-stack-be82fee9 -l
Deploying stack-stack-be82fee9.yaml ……
Enter app name (use ‘-’ for None) [docker-wordpress-us-east]:
…
create app docker-wordpress-us-east succeed!
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED …
9359ccecf6d6 visualops/mysql “/bin/bash /opt/run. 5 seconds ago …
7627f39c6e4d visualops/wp “/bin/bash /opt/run. 5 seconds ago … 
9da651202d13 visualops/wp “/bin/bash /opt/run. 5 seconds ago …
```

#### Done!

### 3. Clone a running app from the cloud to the laptop

```
$ visualops app clone app-42c79a93
Show remote app info….
Pulling app-42c79a93 from remote ….
> app-42c79a93 is saved to /root/app-42c79a93.yaml
Done!
Enter app name (use ‘-’ for None) [elasticsearch]:
…
create app elasticsearch succeed!
```

#### Done!

## What’s Next?

Our new tool currently supports all Linux distributions supported by Docker and compatible with VisualOps. Ubuntu, CentOS, Amazon Linux and RedHat Linux (latest version) are officially supported.

The next version will support Mac OS X by [boot2docker](http://boot2docker.io/).

Happy Hacking!

---

Original source: [Streamline SDLC with Docker using VisualOps](https://medium.com/@visualops/streamline-sdlc-with-docker-345da157e374)