# Why developers love Docker (Part II)

---

Author: Felix Crisan

---

## Docker 101

Docker relies for its execution environment, on features in the host’s kernel – LXC. But it also needs filesystem support in the so called UFS (Union File System).

![alt](http://resource.docker.cn/docker-filesystems-multilayer-update.png)

Early versions of Docker relied on AuFS, but more recent releases have adopted a ‘neutral’ approach, with a preference for devicemapper. This use of a copy-on-write FS is actually what makes Docker look like Git (and Docker Hub like GitHub).

Docker’s two main concepts are Containers and Images, where Containers are, well, containers running the application and Images are the Containers-at-Rest (i.e. saved container state). To make an analogy with class concepts in OOP, Images are class definitions, while Containers are class instances at run-time.

All in all there are just 28 commands that the tool called Docker (self-entitled “self-sufficient runtime” for Linux containers) understands, but they wrap all the mentioned capabilities – including the control (such as start/stop a container) and the meta-management (such as push/pull to/from the repository).

For instance starting a container running Redis is as simple as*:

```
docker pull dockerfile/redis

docker run -d --name redis -p 6379:6379 dockerfile/redis 
```

(*note that there are other Redis ‘templates’ as well)

This will allow one to subsequently use

```
<container_host_ip>:6379
```

to connect to the Redis server. To get a feeling of what Docker is like just visit [https://www.docker.com/tryit/](https://www.docker.com/tryit/) to access a browser based interactive tutorial.

The latest version of Ubuntu Linux (14.04 LTS) comes with activated Docker support. It still needs to be installed, though that’s just an ‘apt-get install docker.io’.

## Docker Benefits

The roles that have most to benefit from Docker are those at the two ends of the development cycle (developer and dev-ops/sysadmins), but it also improves the lives of everyone else in between (like QA staff).

Docker’s most appreciated benefits are:

- **application-centric**: the incremental improvement that Docker brings on LXC is focus on deployment of application vs deployment of machines

- **portability**: once published, any image will yield the exact same result wherever it runs

- **versioning**: much like Git, allows commit/push and pulls on existing images, verifying differences from one commit to the other

- **automation**: there are tools that allow a machine, once running, to reach a specific state

- **sharing**: through the use of Docker Hub anyone can build on existing machines or make available their images for others

- **reusability**: an image can be ‘fork’-ed for two different purposes.

Stay tuned for the last article of the [Docker series](http://blog.bigstep.com/tag/docker/).  Meanwhile, if you have any questions, do let us know and we’ll do our best to answer them.

This is a guest post written by Felix Crisan.

---

Felix Crisan – CTO of [Netopia](https://www.mobilpay.ro/public/en) (company behind mobilPay and web2sms services), has more than 15 years of experience in IT, payments and telecom. He went from startups to corporate and then back to startup life, building architectures for IBM and HP and as well as games like Moorhuhn. From employee to entrepreneur, his passion has always been the technology and programming, lately being quite a Big Data aficionado.

Follow Felix on [Twitter](https://twitter.com/fixone) and [LinkedIn](https://www.linkedin.com/in/fixone).

---

Original source: [Why developers love Docker (Part II)](http://blog.bigstep.com/big-data-performance/developers-love-docker/)

