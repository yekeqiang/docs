# Updating your Docker for Shellshock

---

##### Author: Brian Christner

---

What a week. Starting my workday early Thursday (25 September) morning I came across a Tweet from someone I follow that he spent the entire night updating his Linux systems. Hmmm this doesn't look good. After quickly getting up to speed with the [Shellshock Bug](http://www.troyhunt.com/2014/09/everything-you-need-to-know-about.html) it was time to make a plan.

## Reviewing the Landscape

I quickly evalutated all my webservers spread around the world and made a plan to update. Updating my webservers was relatively uneventful and fast but required quite sometime to ssh to each one and run the commands.

Since my last article about launching a [Dockerized Blog](http://brianchristner.io/migrating-from-wordpress-to-dockerized-ghost/) solution this adds a new aspect to updating Docker for this Bash patch as I need to update the base Docker image.

I have 3 Docker containers: 

- 1 x NGINX Proxy 
- 2 x Nodejs + Ghost CMS

## How to update Docker Images & Containers

What I did is deploy a new container for both the NGINX Proxy & Ghost CMS. I started both containers in interactive mode with /bin/bash enabled. I then updated Bash inside the containers, Committed the Container as a new image, and then deployed the new image.

So let's take a look at the process step-by-step:

1. Deploy a new Docker container for both NGINX and Ghost 
NGINX -> 

```
docker run -i -p 80:80 -v \ /var/run/docker.sock:/tmp/docker.sock \
--name proxy jwilder/nginx-proxy /bin/bash
```

Once the container is running and you are attached run: 

```
apt-get update 
apt-get install --only-upgrade bash
```

2. Now repeat the process for the Ghost containers 
Ghost ->

```
docker run -i -p 49157:2368 -v \ /var/docker/directory_name:/ghost-override \
-e VIRTUAL_HOST=www.example.com \ dockerfile/ghost /bin/bash apt-get update apt-get install --only-upgrade bash
```

3. Next let's commit the containers as new Docker Images

``` 
docker commit -m"Updated Bash" -a="Brian" \ proxy jwilder/nginx-proxy:v2
```

4. Repeat the process for Ghost image by changing the image and container names in the command.

5. Stop the running containers for proxy and Ghost 

```
docker stop proxy ghost1 ghost2
```

6. Deploy the newly created proxy container 

```
docker run -i -p 80:80 -v \ /var/run/docker.sock:/tmp/docker.sock \ --name proxy jwilder/nginx-proxy:v2 forego start -r
```

7. Same for the Ghost image 

```
docker run -i -p 49157:2368 -v \ /var/docker/directory_name:/ghost-override \ -e VIRTUAL_HOST=www.example.com \ dockerfile/ghost bash /ghost-start
```

Happy Dockering!

---

Original source: [Updating your Docker for Shellshock](http://brianchristner.io/updating-your-docker-containers-for-shellshock/)