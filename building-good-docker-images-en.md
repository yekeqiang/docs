# Building good Docker images

---

##### Author: Jonathan Bergknoff

---

The [docker registry](https://registry.hub.docker.com/) is bursting at the seams. At the time of this writing, a search for "node" gets just under 1000 hits. How does one choose?

## What constitutes a good docker image?

This is a subjective matter, but I have some criteria for a docker image that I consider good:

- **working**. Some examples:

	- an Android SDK image should be able to compile a project without first applying updates to the container.
	- a MySQL container should expose a way to bootstrap the server with a database and user.

- **minimal**. The beauty of containers is the ability to sandbox an application (if not for security, then to avoid clutter on the host file system). Whereas I could install node.js on my host system or pollute it with a Java Development Kit, I would rather pay a slight premium in disk space or performance to keep them cordoned off from the rest of my files. With that said, it is obviously preferable that these penalties be as small as possible. The docker image should serve its purpose, having exactly what's necessary for it to function but nothing else. Following this principle, the image is more extensible and has fewer things that can break.

- **whitebox**. In the case of docker images, this means having a published `Dockerfile`. That way I can evaluate what went into creating the image and tinker with it if I want to.

Unfortunately the docker registry does not make it easy to discover "good" images or even to judge any particular image. It's often a matter of `docker pull <...>` and then wondering why the 10 megabyte `node` binary needs 10 file system layers and, ultimately, a 700 megabyte virtual environment.

## Building good docker images

Because there is no consensus on "good" docker images, and because the barrier to entry for adding images to the docker registry is very low, the situation is straight out of [xkcd #927](http://xkcd.com/927/): everybody just does his or her own thing. The introduction of ["official" language-specific docker development environments](https://github.com/docker-library/official-images) is a good start. I was happy to see that some of my pet practices (listed below) showed up in those images. However, the "thousand node images" situation probably won't improve much until the docker registry works on its discovery and evaluation mechanisms.

With that said, here are the Dockerfile practices which I've settled on as best. I am no expert (I don't think anybody is at this early point in docker's lifetime), so discussion and feedback are welcome.

- ### Base images off of `debian`

At the time of this writing, `ubuntu:14.04` is 195 MB while `debian:wheezy` is 85 MB, but the extra hundred megabytes of Ubuntu doesn't buy you anything of value (that I'm aware of). In some extreme cases, it may even be possible to base your image off of 2 MB `busybox`. This is probably only practical with a statically linked binary. An example of a `busybox-based` docker image is `progrium/logspout` ([link](https://github.com/progrium/logspout)) which clocks in at a respectable 14 MB.

- ### Don't install build tools without good reason


Build tools take up a lot of space, and building from source is often slow. If you're just installing somebody else's software, it's usually not necessary to build from source and it should be avoided. For instance, it is not necessary to install python, gcc, etc. to get the latest version of node.js up and running on a Debian host. There is a binary tarball available on the [node.js downloads page](http://nodejs.org/download/). Similarly, redis can be installed through the package manager.

There are at least a few good reasons to have build tools:

   * you need a specific version (e.g. redis is pretty old in the Debian repositories).
   * you need to compile with specific options.
   * you will need to `npm install` (or equivalent) some modules which compile to binary.

In the second case, think really hard about whether you should be doing that. In the third case, I suggest installing the build tools in another "npm installer" image, based on the minimal node.js image.

### Don't leave temporary files lying around

The following `Dockerfile` results in an image size of 109 MB:

```
FROM debian:wheezy
RUN apt-get update && apt-get install -y wget
RUN wget http://cachefly.cachefly.net/10mb.test
RUN rm 10mb.test
```

On the other hand, this seemingly-equivalent `Dockerfile` results in an image size of 99 MB:

```
FROM debian:wheezy
RUN apt-get update && apt-get install -y wget
RUN wget http://cachefly.cachefly.net/10mb.test && rm 10mb.test
```

Thus it seems that if you leave a file on disk between steps in your `Dockerfile`, the space will not be reclaimed when you delete the file. It is also often possible to avoid a temporary file entirely, just piping output between commands. For instance,

```
wget -O - http://nodejs.org/dist/v0.10.32/node-v0.10.32-linux-x64.tar.gz | tar zxf -
```

will extract the tarball without putting it on the file system.

### Clean up after the package manager

If you run `apt-get update` in setting up your container, it populates `/var/lib/apt/lists/` with data that's not needed once the image is finalized. You can safely clear out that directory to save a few megabytes.

This `Dockerfile` generates a 99 MB image:

```
FROM debian:wheezy
RUN apt-get update && apt-get install -y wget
```

while this one generates a 90 MB image:

```
FROM debian:wheezy
RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*
```

### Pin package versions

While a docker image is immutable (and that's great), a `Dockerfile` is not guaranteed to produce the same output when run at different times. The problem, of course, is external state, and we have little control over it. It's best to minimize the impact of external state on your `Dockerfile` to the extent that it's possible. One simple way to do that is to pin package versions when updating through a package manager. Here's an example of how to do that:

```
# apt-get update
# apt-cache showpkg redis-server
Package: redis-server
Versions:
2:2.4.14-1
...

# apt-get install redis-server=2:2.4.14-1
```

We can hope, but there is no guarantee, that the package repositories will still serve this version a year from now. However, it's undeniably valuable to explicitly show what version of the software your image depends on.

### Combine commands

If you have a sequence of related commands, it is best to chain them into one `RUN` command. This makes for a more meaningful build cache (logically grouped steps are lumped into one cache step) and keeps the number of file system layers down (I consider this generally desirable but I don't know that it's objectively better).

Backslashes `\` help you out here for readability:

```
RUN apt-get update && \
    apt-get install -y \
        wget=1.13.4-3+deb7u1 \
        ca-certificates=20130119 \
        ...
```

### Use environment variables to avoid repeating yourself

This is a trick I picked up from reading the `Dockerfile` ([link](https://github.com/docker-library/node/blob/51b1dd1984e287189106884c453ca506737eed78/0.10/Dockerfile)) of the "official" node.js docker image. As an aside, this `Dockerfile` is great. My only criticism is that it sits on top of a huge `buildpack-deps` ([link](https://github.com/docker-library/buildpack-deps/blob/69f0b516b5515939bef6170f1e82362174143d13/wheezy/Dockerfile)) image, with all sorts of things I don't want or need.

You can define environment variables with `ENV` and then reference them in subsequent `RUN` commands. Below, I've paraphrased an excerpt from the linked `Dockerfile`:

```
ENV NODE_VERSION 0.10.32

RUN curl -SLO "http://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz" \
    && tar -xzf "node-v$NODE_VERSION-linux-x64.tar.gz" -C /usr/local --strip-components=1 \
    && rm "node-v$NODE_VERSION-linux-x64.tar.gz"
```

This article is being discussed further in [this Hacker News post](https://news.ycombinator.com/item?id=8483102).

---

Original source: [Building good Docker images](http://jonathan.bergknoff.com/journal/building-good-docker-images)