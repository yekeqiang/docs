# Setting up a Ghost blog using Docker and Nginx, Part 1

---

Author: Carlos Barba

---

So you found out about the Ghost blogging platform and thought "**hey this is pretty awesome, I want that!**" but [Ghost pro](https://ghost.org/pricing/) wasn't for you and the open source setup was a bit more than you bargained for?

#### Fear no more!

After trying out a few ways of setting up ghost I've found that using Docker was by far the fastest, easiest and most extensible way to deploy the platform. And I'll be glad to show you how.

## Let's get started

I'll be using Ubuntu on a Digital Ocean Droplet but you are welcome to try this guide with your own setup.

#### TL;DR

I'm going to try to explain the setup and breakdown each command but you can skip all this and go straight to this [gist](https://gist.github.com/carloscheddar/bd645e5329851e2ad8e4) which has all the steps needed to get you started.

## Installing Docker

Remember that complicated setup you didn't want to go through? What if I told you that someone already went through it and decided to upload and share his progress. That's where Docker comes in, by installing it we can pull an image of an already working ghost blog into our servers. Yeah it's like magic, let's install it with this command.

```
curl -s https://get.docker.io/ubuntu/ | sudo sh
```

## Setting up Ghost
The ghost image we're going to use is provided by the Docker team and you can see what we're installing here.

### Making data persistent

Docker lets you create volumes which we use to share data with the host. This way if we mess up the blog container we're free to delete it and try again because our data is persistent. To use this method we first create the folder where the data should be saved. I'll be creating mine in the home directory but you are free to put it anywhere you want.

```
mkdir ~/ghost-data
```

Let's also add the `config.js` file to the `ghost-data` folder so that you can edit the blog's settings. You can download an example from the ghost github repo [here](https://github.com/TryGhost/Ghost/blob/master/config.example.js).

For this setup to work you must replace this line:

```
    host: '127.0.0.1',
```

with

```
    host: '0.0.0.0',
```

Now we need to tell docker where our data is and where we want our container to find it by using this command:

```
docker run -v /home/carloscheddar/ghost-data/:/ghost-override --name=ghost-storage ubuntu
```

Let's breakdown this command:

- `run`: Creates the docker container.
- `-v /home/carloscheddar/ghost-data/:/ghost-override`: This tells docker to bind our `ghost-data` folder into a `ghost-override` folder that the ghost image uses. This command needs to use the absolute path of `ghost-data` or docker will output errors.
- `--name=ghost-storage`: We name it `ghost-storage` for easy access later on.
- ubuntu: We use the ubuntu image for our storage container.

If you run this command you'll see that you have a `ghost-storage` container:

```
docker ps -a
```

### Creating the ghost container

If you didn't have problems creating the storage container we can now use the volume we created.

```
docker run -d --volumes-from ghost-storage --name=ghost -p 80:2368 dockerfile/ghost
```

Let's breakdown this command so you know what's happening:

- `run`: this tells docker to create a container and run it with these arguments.
- `-d`: This tells docker to run a **detached** container. This allows the container to run in the background.
- `--volumes-from ghost-storage`: We use the volume created by the previous command.
- `-p 80:2368`: So ghost runs on port **2368** and we're just telling docker to forward that port to the host's port **80**.
- `--name=ghost`: This argument lets you name the container, you con specify any name you want I'm using **ghost** for clarity.
- `dockerfile/ghost`: This is where the magic happens, we're telling docker to go to the [docker hub](https://registry.hub.docker.com/) and pull some image called **dockerfile/ghost**.

If at any point you encounter errors you can delete containers and try again by executing the remove command:

```
docker rm ghost-storage ghost
```

## What's next?

Now you have a fully functioning ghost blog with persistent storage. Since we forwarded port 80 when creating the container our server is already up and ready to receive requests. Type your ip into the browser and see how it works.

Now this setup may be enough for most cases but you may want something more robust that can handle multiple sites and that's where [Nginx](http://nginx.org/en/) comes in.

If you want to continue and set up Nginx you can continue this guide [here](http://carloscheddar.com/setting-up-a-ghost-blog-using-docker-and-nginx-part-2).

----

Original source: [Setting up a Ghost blog using Docker and Nginx, Part 1](http://carloscheddar.com/setting-up-a-ghost-blog-using-docker-and-nginx-part-1/)