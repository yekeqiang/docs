# oVirt Dockerized: Part 2

---

##### Author: Moran Goldboim 

---

![alt](http://resource.docker.cn/ovirt-logo.png)

Yesterday in [oVirt Dockerized: Part 1](http://community.redhat.com/blog/2014/10/ovirt-dockerized/), we learned how to run oVirt on a virtual machine or server, and the "fun part" of making oVirt go. In Part 2, we dive deeper into running oVirt with a remote database and as a standalone container, interactive configuration of an oVirt container, and how to clean up your environment.

## Run oVirt with a Remote Database

In this scenario, the oVirt container is connected to the data containers and are linked within the same namespace

```
cd oVirt-Dockerized/Run && sudo make ovirt-run
```

![alt](http://resource.docker.cn/ovirt-dockerized-newpt2.png)


## Run oVirt as a Standalone Container

Here, there is one container with the engine and a database.

```
cd oVirt-Dockerized/Run && sudo make ovirt-SA-run
```

The above commands will use the images on the [mgoldboi/ovirt*](https://hub.docker.com/u/mgoldboi/) Docker registry. If you want to build or rebuild your own images, you can fork or clone the [oVirt-dockerzied](https://github.com/mgoldboi/oVirt-Dockerized) repo and examine the [Build/Makefile options](https://github.com/mgoldboi/oVirt-Dockerized/blob/master/Build/Makefile). Some examples include:

### Interactive Configuration of oVirt Container

```
cd oVirt-Dockerized/Build && sudo make ovirt-manual
```

Build your custom auto-configuration by changing the answer file values.

```
cd oVirt-Dockerized/Build && vim DockerFiles/ovirt-SA/ovirt-engine-35-localdb.conf
```

You can start by changing the password:

```
 OVESETUP_CONFIG/adminPassword=str:ovirt
```

Or you can configure your hostname fqdn:

```
OVESETUP_CONFIG/fqdn=str:localhost
```

Once you are done with such changes, you can build it:

```
sudo make ovirt-SA
```

You can watch the build process proceed and oVirt being configured with your custom values. This will create an `ovirt-sa-configured-3.5.0` package in your local repo. You can then list the images you have using

```
sudo docker images
```
and finally make it run:

```
cd oVirt-Dockerized/Run && sudo make ovirt-SA-run
```

### Clean Your Environment

To stop and remove your oVirt containers, run

```
make clean
```

Remember, this command doesn’t touch images; those can be removed with

```
docker rmi image_name
```

### About Moran Goldboim

Moran Goldboim is a Supervisor of Software Engineering at Red Hat, based in Ra'anana, Israel.

---

Original source: [oVirt Dockerized: Part 2](http://community.redhat.com/blog/2014/10/ovirt-dockerized-part-2/)