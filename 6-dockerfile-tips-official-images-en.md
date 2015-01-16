# 6 Dockerfile Tips from the Official Images

---

Author: Adrian Mouat 

---

Following on from my [previous post on the Docker Official Images](http://container-solutions.com/2014/10/docker-official-images-good-bad-ugly/), in this post I’ll go through some tips and techniques for writing Dockerfiles that I learnt from the official images.


## 1. Prefer Debian

The majority of Dockerfiles for official images are based on Debian, either directly or through another image. The version is usually pegged to a given distribution, normally wheezy (at time of writing, the stable distribution), but several images use jessie (in testing) and even sid (unstable). The main advantage of the Debian image is the smaller size – it clocks in at around 85.1 MB compared to around 200 MB for Ubuntu. Specifying the exact distribution guards against your build breaking when the  distribution tagged latest is upgraded.

## 2. Establish Provenance

If your users need to be able to rely on and trust your image, you need to consider how to verify the authenticity of any software installed in that image. In the case of apt-getting from official Debian repositories, this is taken care of already. However, if you download files from the internet or install software from a third-party repository, you should verify the files by testing checksums or digital signatures. For example, the nginx Dockerfile does the following to verify the nginx package:


```
RUN apt-key adv --keyserver pgp.mit.edu --recv-keys 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
RUN echo "deb http://nginx.org/packages/mainline/debian/ wheezy nginx" >> /etc/apt/sources.list
 
ENV NGINX_VERSION 1.7.7-1~wheezy
 
RUN apt-get update && apt-get install -y nginx=${NGINX_VERSION}
```

Notice nginx is pegged to a specific version. This is a good idea as it helps to ensure the image tested by the maintainer is the same as the one built by the build system. It isn’t infallible however, as nginx itself is likely to pull in dependencies which may change over time (consider dependencies specified as >= to a given version).

You can do something similar for any files downloaded, by taking a cryptographic sum of the file and testing against a stored version. This is done in the [Redis Dockerfile](https://github.com/docker-library/redis/blob/master/2.8/Dockerfile). Also, some downloads will have a signature file which you can test with gpg, which again is commonly done in the official images.

Unfortunately several of the official images fail to do this correctly currently, or only validate some files, so be aware of this when looking at the official Dockerfiles.

## 3. Remove Build Dependencies

If you compile code from source during your build, it is likely your image is much larger than it needs to be. If possible, try to install the build tools, build the software and remove the build tools all in the same RUN instruction. This is awkward and annoying, but can save 100s of MBs. There is no point in deleting files in a separate instruction as they will already have been bundled into the image. For an example of how to do this, we can look at the Redis Dockerfile again:


```
RUN buildDeps='gcc libc6-dev make'; \
    set -x \
    && apt-get update && apt-get install -y $buildDeps --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /usr/src/redis \
    && curl -sSL "$REDIS_DOWNLOAD_URL" -o redis.tar.gz \
    && echo "$REDIS_DOWNLOAD_SHA1 *redis.tar.gz" | sha1sum -c - \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && rm redis.tar.gz \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && ln -s redis-server "$(dirname "$(which redis-server)")/redis-sentinel" \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

**gcc**, **libc** and **make** are installed, used and deleted in this one instruction. Also note the author has deleted the no longer needed tar.gz file source directory. Incidentally, this code also shows how to use sha1sum to verify the checksum of the Redis download.

## 4. Check Out gosu

The [gosu](https://github.com/tianon/gosu) utility is often used in scripts called from ENTRYPOINT instructions inside Dockerfiles for official images. It’s a very simple utility, similar to sudo, that runs a given instruction as a given user. The difference is that gosu avoids sudo‘s “strange and often annoying TTY and signal-forwarding behavior”.

Also check out the [official advice on writing entrypoint scripts](https://docs.docker.com/articles/dockerfile_best-practices/), which is followed by most official images

## 5. Consider the buildpack-deps Base Image

Several of the Docker “language-stack” images are based on the [buildpack-deps](https://registry.hub.docker.com/u/library/buildpack-deps/) base image, which installs various commonly required development headers and tools (such a source code management tools). If you are building a language-stack image, you may well be able to save some time by using this base image. It has come in for some criticism for adding unnecessary bloat to images which has lead to several repositories such as Node offering alternative slim packages based directly on Debian (the full Node image is 728MB compared to just 291.4MB for slim).  However, remember your users may require the same development libraries and are likely to have downloaded the base image already anyway.

## 6. Use a Range of Descriptive Tags

All of the official repositories offer a range of tags. As well as a **latest** tag, it is good form to offer a versioned tag that users can use without fear of the base image changing and breaking their containers. The official images take this further, often offering minimal **slim** images as discussed above as well as **onbuild** images which take care of automatically importing and compiling code. By tagging this image onbuild, the user is much less surprised when code is moved about and compiled when creating a child image.

---

Original source: [6 Dockerfile Tips from the Official Images](http://container-solutions.com/2014/11/6-dockerfile-tips-official-images/)