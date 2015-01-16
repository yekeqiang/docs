# Docker, Weave, Raspberry Pi and a bit of Networked Cloud Computing!

---

Author: Alexander Grendel

---

I have been playing around with [Weave](https://github.com/zettio/weave) and [Docker](https://www.docker.com/) the past few months and recently wanted to see if I could bind the [cloud platform](http://cloudstore.interoute.com/main/WhatInterouteVDC) we operate at work with my Raspberry Pi I use at home. Ideally this Pi is not bound to my home at all and be used wherever I can get it to connect to the internet.

## The Network Problem

Networking is still one of the main hurdles to get things done. We are so accustomed to the arcane ways of traditional networking it is hard for people to see how technology can bring down these barriers. Operating systems and networks are a means to an end: It is all about the applications. Process isolation —pioneered for modern UN*X in Sun Solaris (zones), FreeBSD (jails) and later OpenVZ / Linux-vserver — paved the way for Docker to emerge and with the development in namespace/cgroups in the linux kernel this is now a relatively mature technology. Add Weave to the mix and the world is suddenly upside down!

What I want is the ability to connect my personal dropbox-like server (application B) in the cloud to connect with my home NAS storage solution (application A) to make all files available and serve some additional content from another server in the cloud as well (application C).

## The Setup

To prove the point I set out to deploy Weave on my Pi and connect it to my existing Docker/Weave setup I have in the Interoute Virtual Data Centre. Interoute has a integrated network / IaaS platform that is ideally suited for these type of scenarios as the private interconnectivity between cloud zones is automated and comes for free!

The setup roughly looks like the diagram below:

![alt](http://resource.docker.cn/docker-raspberry-1.jpeg)

2 interconnected virtual machines in the cloud and an on-premise device (at home in this example)
Traditionally you would set up some IPSEC or SSL VPN tunneling to link the environments but that involves annoying configuration steps just to get a private interconnect. Luckily the cloud zones are already securely interconnected on layer-3. The real problem starts when you need to route traffic to get to a remote host or have more than one gateway to direct traffic to public and internal networks (like on server 2). Even for me, I have been in the hosting and networking business professionally for 20 years, this means I have to think and work on it to make sure the routing is set up correctly. A waste of time and an endless source of frustration.

## A solution using Docker and Weave

Docker allows to create micro-services or full fledged linux distributions to run in containers and there is an explosion of management options emerging to deal with them at scale that I won’t cover here.

Weave is a tool that manages virtual networking for Docker containers deployed across multiple hosts. We are using 2 main features here;

- The ability to put containers on various hosts and attach them to the same subnet (virtual layer-2 broadcast domain effectively)
- Automatic path discovery for containers connected to the Weave network.

After setting up weave the solution suddenly looks like this:

![alt](http://resource.docker.cn/docker-raspberry-2.jpeg)

We have all applications connected and no routing configuration needed!
There is some magic happening on the path discovery from server 1 to server 3: As long as there is at least one path between servers Weave will automatically forward packets. Weave will learn MAC addresses from neighbors and learns next-hop for packets that way, see this.

The great thing about the setup is that I can take my Pi anywhere and as long as I have internet that allows (stateful outboud) tcp/udp on port 6783 it will always connect to the cloud setup.

## The ingredients

The tricky part was the Raspberry Pi to run Docker and Weave, as it requires recent kernel and userland to work properly. After some experimentation with Ubuntu on an odroid-u3 and Arch Linux and Raspbian on a Raspberry Pi and I settled for:

- Raspberry Pi model B
- Raspbian Wheezy upgraded to Jessie and 3.16 kernel (I could not find a proper Jessie image for the Pi)
- Minimum 8GB SD card to not run out of space while upgrading and please make this the fastest you can find, you have been warned!
- a bunch of cloud virtual machines running Debian Jessie (or whatever else that runs docker+weave)

## Set up the Raspberry Pi

- Download [Raspbian Wheezy](http://www.raspberrypi.org/downloads/) and write it to your SD card.

- after booting the Pi change all references from Wheezy → Jessie in `/etc/apt/sources.list` and run:


```
# sudo apt-get update
# sudo apt-get -y upgrade
# sudo apt-get -y dist-upgrade
```

Sit back and fall asleep as this will take long. Make sure to occasionally press y and continue sleeping (Press No for changes to dphys-swapfile as it will create a 1gb swapfile & raspbian uses a mostly unused 100MB swapfile instead).

Make sure to install kernel 3.16, libpcap-dev, Mercurial, Docker (we need docker to build Weave) and Go on the Pi:

```
# sudo apt-get install linux-image-rpi
# sudo apt-get install libpcap-dev
# sudo apt-get install mercurial
# sudo apt-get install golang-go
# sudo apt-get install docker.io
```

Jessie uses systemd. Make sure docker is running:

```
# sudo systemctl start docker
```

Now change `/boot/config.txt` to include (vmlinuz-3.16.0.4-rpi in my case):

```
kernel=/boot/vmlinuz-<version>-rpi
```

Download the Weave source code (as per Weave docs) and build it:

```
$ cd ~ 
$ mkdir go
$ export GOPATH=~/go 
$ export PATH=$PATH:$GOPATH/bin 
$ cd $GOPATH
$ WEAVE=github.com/zettio/weave
$ git clone https://$WEAVE
$ cd src/$WEAVE
$ make
```

This should get you set up.

## Configuring the 3 servers with Docker and Weave

Install Docker on the cloud servers:

```
# sudo -i
# apt-get install docker.io
```

Install the Weave script:

```
# wget -O /usr/local/bin/weave \
 https://raw.githubusercontent.com/zettio/weave/master/weave
# chmod a+x /usr/local/bin/weave
```

Once all hosts run docker, do this on the first host:

```
# weave launch -password <mypassword>
```

Any additional hosts can be added by running the same command plus an ip address of an already configured host like this:


```
# weave launch -password <mypassword> <ip address of first host or ANY existing weave host> 
```

Only requirement is that firewalls are configured to allow TCP and UDP connections to port 6783 to the weave hosts.

Now for the Raspberry Pi:

```
# weave run 10.0.1.1/24 -t -i --name ApplicationA \         resin/rpi-raspbian:jessie /bin/bash
```

And on the cloud VMs:


```
# weave run 10.0.1.2/24 -t -i --name ApplicationB ubuntu /bin/bash # weave run 10.0.1.3/24 -t -i --name ApplicationC ubuntu /bin/bash
```

To attach and test:

```
# docker attach <name>
```

When you have attached the vm you can do a ping to the other hosts in the 10.0.1.0/24 network. Even the container on server 3 is connected to the container on server 1 even though there is no direct connection between them: All traffic is automatically routed via server 2!

![alt](http://resource.docker.cn/docker-raspberry-3.png)

*Weave status shows topology on server2 (public ip addresses for server 1 and 2 blacked out)*

Installing the actual applications (owncloud and samba primarily in my case) inside docker I leave as an exercise to the reader, have fun!

## Links

- [Weave](https://github.com/zettio/weave)
- [Docker](http://docker.io/)
- [Raspbian](http://www.raspbian.org/)
- [Debian](http://www.raspbian.org/)
- [Interoute VDC](http://cloudstore.interoute.com/)

## Troubleshooting

I ran into issues along the way with using older kernels and other distro (not affecting the final configuration):

- Arch Linux would not compile weave due to missing libpcap while the headers were there. No patience to figure out dependencies.

- In the /usr/local/bin/weave script there is a line that sets MTU for the bridge, this was not supported on older kernels/drivers and you will get:

> RTNETLINK: Operation not supported

> I modified the script to set MTU at the start of bridge creation as that was supported for my setup, but ran into other problems later anyway.

- kernel for most common distributions that are shipped as images for raspberry / arm are not recent enough to properly run docker and/or miss cgroups kernel configs. I settled for Raspbian and linux-image-rpi (linux-image-3.16.0.4-rpi at the moment) package.

## Thoughts/Risks

- With weave you are potentially bridging security domains which could have implications on your security policies.

- I have not looked in detail at the security Weave implements with the -password option. No comment on the security as a whole of this setup. I will dig into that and update as needed.

- *disclaimer: docker is not officially supported on ARM and has potential issues! This is an experimental setup (for now)*.

---

Original source: [Docker, Weave, Raspberry Pi and a bit of Networked Cloud Computing!](https://medium.com/@ALGrendel/docker-weave-a-little-cloud-and-a-raspberry-pi-381f73a4376d)