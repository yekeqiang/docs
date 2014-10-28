# Give wings to the whale: biicode with docker

---

##### Author: Biicode

---

We have uploaded a docker image with biicode pre-installed. So, you can quickly try biicode thanks to docker!

Don’t you know what’s docker? Let’s start with a short explanation about it.

## What’s docker

[Docker](https://www.docker.com/whatisdocker/) is an open platform for developers and sysadmins to build, ship, and run distributed applications. Consisting of Docker Engine, a portable, lightweight runtime and packaging tool, and Docker Hub, a cloud service for sharing applications and automating workflows, Docker enables apps to be quickly assembled from components and eliminates the friction between development, QA, and production environments. As a result, IT can ship faster and run the same app, unchanged, on laptops, data center VMs, and any cloud.


## How to run the biicode image

First of all, you need to have [docker installed](https://docs.docker.com/installation/#installation) . Then, run the following command to download and run a docker container with the ubuntu 14.04 image with biicode pre-installed.


```
sudo docker run -t -i davidsanfal/ubuntu14_biicode:latest /bin/bash
 
Unable to find image 'davidsanfal/ubuntu14_biicode:latest' locally
Pulling repository davidsanfal/ubuntu14_biicode
eb94c93951d5: Download complete 
511136ea3c5a: Download complete 
b18d0a2076a1: Download complete 
67b66f26d423: Download complete
...
root@dc7e87dcfc2b:~#
```

Now, try the biicode installation.


```
root@dc7e87dcfc2b:~#  bii
ERROR: None or bad command. Type "bii --help" for available commands
```

And this is all you need to use **biicode with docker**!

## How to use biicode inside the docker container

Now, let’s make an [example with SQLite](http://docs.biicode.com/c++/examples/sqlite.html).

First, open and build the [examples/sqlite_basic](http://www.biicode.com/examples/sqlite_basic) block.


```
root@dc7e87dcfc2b:~# bii init sql
Successfully initialized biicode project sql
root@dc7e87dcfc2b:~# cd sql
root@dc7e87dcfc2b:~/sql# bii open examples/sqlite_basic
INFO: Processing changes...
Opened examples/sqlite_basic
root@dc7e87dcfc2b:~/sql# bii cpp:build
INFO: Processing changes...
Running: cmake  -G "Unix Makefiles" -Wno-dev  ../cmake
 
(...)
 
[100%] Built target examples_sqlite_basic_main
```

Now, **run the example**.


```
root@dc7e87dcfc2b:~/sql# bin/examples_sqlite_basic_main    
Opened database successfully
 
SELECT, List Veggies
 
STORE = Veggies
NAME = Spinach
NUMBER = 3
 
STORE = Veggies
NAME = Onion
NUMBER = 1
 
 
SELECT, List Drinks
 
STORE = Drinks
NAME = Coffee
NUMBER = 7
 
 
SELECT, Updated Lists:
 
STORE = Drinks
NAME = Coffee
NUMBER = 2
 
STORE = Veggies
NAME = Onion
NUMBER = 1

Closed database successfully
```

If you want to know how to upload an image to docker-hub or anything, you can see the [docker-hub docs](https://docs.docker.com/introduction/#docker-hub) or visit our [forum](http://forum.biicode.com/) and ask any questions. feel free to upload your own images!

---

Original source: [Give wings to the whale: biicode with docker](http://blog.biicode.com/biicode-docker/)

