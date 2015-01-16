# How to Automate Docker Deployments

---

Author:  Caleb Sotelo

---

For months I've been using [Drone](https://github.com/drone/drone) to [continuously build and deploy Docker applications](http://paislee.io/how-to-automate-docker-deployments/paislee.io/how-to-build-and-deploy-docker-images-with-drone/). This has achieved the desired effect of drastically reducing the time required to get code into production. But it wasn't until recently that I finally got a handle on my deploy script, which was silently leaking old, untagged Docker images onto my hard drives. **Below, I share a method for cleanly upgrading a running container, in the form of a bash script suitable for basic automated deployment setups**.

## Docker Images vs. Containers

First, let's cover some important points about Docker that will help explain the script below. In Dockerland, there are **images** and there are **containers**. The two are closely related, but distinct. For me, grasping this dichotomy has clarified Docker immensely.

## What's an Image?

An image is an inert, immutable, file that's essentially a snapshot of a container. Images are created with the [build](http://docs.docker.com/reference/commandline/cli/#build) command, and they'll produce a container when started with [run](https://docs.docker.com/reference/run/). Images are stored in a Docker registry such as [registry.hub.docker.com](https://registry.hub.docker.com/). Because they can become quite large, images are designed to be composed of layers of other images, allowing a miminal amount of data to be sent when transferring images over the network.

Local images can be listed by running docker images:

```
REPOSITORY                TAG                 IMAGE ID            CREATED             VIRTUAL SIZE  
ubuntu                    13.10               5e019ab7bf6d        2 months ago        180 MB  
ubuntu                    14.04               99ec81b80c55        2 months ago        266 MB  
ubuntu                    latest              99ec81b80c55        2 months ago        266 MB  
ubuntu                    trusty              99ec81b80c55        2 months ago        266 MB  
<none>                    <none>              4ab0d9120985        3 months ago        486.5 MB  
...
```

**Some things to note**:

1. IMAGE ID is the first 12 characters of the true identifier for an image. You can create many tags of a given image, but their IDs will all be the same (as above).
2. VIRTUAL SIZE is virtual because its adding up the sizes of all the distinct underlying layers. This means that the sum of all the values in that column is probably much larger than the disk space used by all of those images.
3. The value in the REPOSITORY column comes from the `-t` flag of the `docker build` command, or from `docker tag`-ing an existing image. You're free to tag images using a nomenclature that makes sense to you, but know that docker will use the tag as the registry location in a docker push or docker pull.
4. The full form of a tag is `[REGISTRYHOST/][USERNAME/]NAME[:TAG]`. For `ubuntu` above, REGISTRYHOST is inferred to be `registry.hub.docker.com`. So if you plan on storing your image called `my-application` in a registry at `docker.example.com`, you should tag that image `docker.example.com/my-application`.
5. The TAG column is just the [:TAG] part of the *full* tag. This is unfortunate terminology.
6. The `latest` tag will always refer to the newest version of an image when specified in a `docker pull`. To avoid breaking this feature for a particular registry image, never add the `latest` tag manually.
7. `docker pull`-ing the `:lates`t tag of an image will add at least two images to your local image list: one with the `latest` tag, and one for each original tag of the latest image, e.g. `14.04` and `trysty` above.
8. You can have untagged images only identifiable by their IMAGE IDs. These will get the `<none>` TAG and REPOSITORY. It's easy to forget about them.

More info on images is available from the [Docker docs](https://docs.docker.com/terms/image/).

### What's a container?

To use a programming metaphor, if an image is a class, then a container is an instance of a class—a runtime object. Containers are hopefully why you're using Docker; they're lightweight and portable encapsulations of an environment in which to run applications.

View local running containers with `docker ps`:

```
CONTAINER ID        IMAGE                               COMMAND                CREATED             STATUS              PORTS                    NAMES  
f2ff1af05450        samalba/docker-registry:latest      /bin/sh -c 'exec doc   4 months ago        Up 12 weeks         0.0.0.0:5000->5000/tcp   docker-registry  
```

Here I'm running a dockerized version of the docker registry, so that I have a private place to store my images. Again, some things to note:

1. Like IMAGE ID, CONTAINER ID is the true identifier for the container. It has the same form, but it identifies a different kind of object.
2. `docker ps` only outputs running containers. You can view *stopped* containers with `docker ps -a`.
3. NAMES can be used to identify a started container via the `--name` flag.

### How to avoid image and container buildup?

One of my early frustrations with Docker was the **seemingly constant buildup of untagged images and stopped containers**. On a handful of occassions this buildup resulted in maxed out hard drives slowing down my laptop or halting my automated build pipeline. Talk about "containers everywhere"!

We can remove all untagged images by combining `docker rmi` with the recent `dangling=true` query:

```
docker images -q --filter "dangling=true" | xargs docker rmi  
```

Docker won't be able to remove images that are behind existing containers, so you may have to remove stopped containers with `docker rm` first:

```
docker rm `docker ps --no-trunc -aq` 
```
 
These are [known pain points](https://github.com/docker/docker/issues/928) with Docker, and may be addressed in future releases. However, with a clear understanding of images and containers, these situations can be avoided with a couple of practices:

1. Always remove a useless, stopped container with `docker rm [CONTAINER_ID]`.
2. Always remove the image behind a useless, stopped container with `docker rmi [IMAGE_ID]`.

These are the practices built into the upgrade script below.

## Deployment Script

The following script is what I use to upgrade a running Docker container to a newer version. This could go in the deploy part of a `drone.yml`:

```
#!/bin/bash
docker pull docker.example.com/my-application:latest  
docker stop my-application  
docker rm my-application  
docker rmi docker.example.com/my-application:current  
docker tag docker.example.com/my-application:latest docker.example.com/my-application:current  
docker run -d --name my-application docker.example.com/my-application:latest 
```
 
Let's step through line by line.

### 1. Pull latest image

```
docker pull docker.example.com/my-application:latest 
```
 
We assume that a more recent image has been built and pushed to a registry. If the image hasn't been tagged, this will result in two new images in the `docker images` list: one with `<none>` tag, and one with `latest`. They'll both share the same IMAGE ID. If there's a previous version of this image tagged with `latest`, it will be untagged.

### 2. Stop the running container

```
docker stop my-application 
```
 
Stop the running instance of `my-application`. This assumes that we gave the image that name last time it was run.

### 3. Remove stopped container

```
docker rm my-application  
```

Now that the container is stopped, it's safe to remove it. This is cleanup step #1. Also, without this step, we wouldn't be able to give the name `my-application` to a different container.

4. Remove image behind stopped container

```
docker rmi docker.example.com/my-application:current 
```
 
Now Docker will let us remove the *image* behind the container we just stopped and removed. This assumes we've previously tagged it with `current`. We need a tag other than `latest` to be able to differentiate between the version of the app we're getting rid of, and the one we're bringing in.

### 5. Tag the newly downloaded image


```
docker tag docker.example.com/my-application:latest docker.example.com/my-application:current  
```

Now that the `current` tag is nonexistent, we tag the downloaded image with `current`, so that we can identify it next time around. Until step 1 of our next upgrade, the `current` and `latest` tags will have the same IMAGE ID.

### 6. Run the new container


```
docker run -d --name my-application docker.example.com/my-application:latest 
```
 
Run the image, being sure to `--name` it `my-application`.

That's it! A possible improvement would be to keep the previous version around to support a quick rollback script. I'm interested in hearing how others are doing automated Docker deployments, and wonder how tools like [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) implement container upgrades under the hood.

---
Original source: [How to Automate Docker Deployments](http://paislee.io/how-to-automate-docker-deployments/)

