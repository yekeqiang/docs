# How to compile Docker on Windows

---

Author: Ahmet Alp Balkan

---

After taking on my new role at the Azure Linux Team, my first assignment was to get the [Docker](http://docker.com/) command line interface working on Windows. This is an important piece in bringing Docker into the Windows ecosystem and part of our [partnership with Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/).

![alt](http://resource.docker.cn/compiling-docker-on-windows-1.png)


<blockquote class="twitter-tweet" lang="en"><p>Who wants <a href="https://twitter.com/docker">@Docker</a> for Windows? Got it cross-compiling tonight!&#10;&#10;🐳 cc:<a href="https://twitter.com/solomonstre">@solomonstre</a> <a href="https://twitter.com/hashtag/docker?src=hash">#docker</a> <a href="http://t.co/bGOn0jmX8m">pic.twitter.com/bGOn0jmX8m</a></p>&mdash; Ahmet Alp Balkan (@ahmetalpbalkan) <a href="https://twitter.com/ahmetalpbalkan/status/526978720657965056">October 28, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Today, I am proud to announce that the [first piece of Windows code is now merged into Docker](https://github.com/docker/docker/pull/9113), which means you can now compile the Docker client on Windows!

<blockquote class="twitter-tweet" lang="en"><p>Spectacular work by <a href="https://twitter.com/ahmetalpbalkan">@ahmetalpbalkan</a>, opening Microsoft&#39;s 1st <a href="https://twitter.com/docker">@docker</a> PR: <a href="https://t.co/xwmj5jPBeJ">https://t.co/xwmj5jPBeJ</a></p>&mdash; Solomon Hykes (@solomonstre) <a href="https://twitter.com/solomonstre/status/532465725261115392">November 12, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Before going any further, I would like to clarify several things:

## Clarifications

- At this point, the merged code is only the first step in getting it compiling and (barely) working on Windows. We have a set of known bugs (see [PR description](https://github.com/docker/docker/pull/9113)), which we are looking forward to fix and I am pretty sure there are more bugs beyond that. Making something designed for UNIX work on Windows seamlessly is not a trivial task. In the upcoming days, I will be focusing on resolving those bugs. We are also looking forward to seeing your contributions to make Docker’s Windows support better.

- Also, please note that this tutorial is not about running the Docker daemon on Windows or running Windows containers on Docker. None of these are possible today. This is just about getting client code compiled on Windows.

- I will not be talking about how the porting work was done, that will be topic of another blog post here on my blog, again. Stay tuned for that one!

- Please note that the docker.exe you are about to build is not a supported distribution by Microsoft or Docker. Please use it at your own risk. A more stable version of the Docker Windows CLI will be shipped later.

## Step 1: Install Go

Download the [Go MSI Installer](https://golang.org/doc/install#windows) from golang.org. This installation will add the `go` program to your PATH environment variable and you should be able to run the `go` command in `cmd.exe`. If that does not work, you may need a restart.

## Step 2: Check out the code

Assuming you have Git installed on your system, you need to clone the [docker/docker](https://github.com/docker/docker/) repository locally:

```
git clone https://github.com/docker/docker.git c:\gopath\src\github.com\docker\docker
```

## Step 3: Compile!

The rest is just as simple. Run `cmd.exe` and run the following commands in order:

```
set GOPATH=c:\gopath;c:\gopath\src\github.com\docker\docker\vendor
set DOCKER_CLIENTONLY=1
cd c:\gopath\src\github.com\docker\docker\docker
go build -v
```

If all goes well, you will end up with a lovely **`docker.exe`** on the directory you are at! ♥

![alt](http://resource.docker.cn/compiling-docker-on-windows-2.png)

(If you happen to run in problems about emulating TTY (linux terminal) in cmd.exe, you need to use ConEmu or install ANSICON. Please see the [pull request](https://github.com/docker/docker/pull/9113) description for detailed info.)

## Summary

This is just an intro to building the Docker CLI for Windows. Normally, these binaries are built in a Linux environment, inside a Docker container (yes, [Go](http://golang.org/) is crazy like that, it can cross compile Windows binaries on Linux)! However, for demonstration purposes we are detailing the procedure for building these Windows binaries on its own turf.

In the meantime, you can use this tutorial to build Docker on your Windows machine to test the changes. We are looking forward to your contributions on making the Docker’s Windows support even better!

<blockquote class="twitter-tweet" lang="en"><p>We promised <a href="https://twitter.com/docker">@docker</a> client for Windows. The first pull request from <a href="https://twitter.com/ahmetalpbalkan">@ahmetalpbalkan</a> is ready. Want to help test it? <a href="https://t.co/t7AM123Pdg">https://t.co/t7AM123Pdg</a></p>&mdash; Ross Gardler (@rgardler) <a href="https://twitter.com/rgardler/status/532736663886528512">November 13, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

If you happen to find a bug, please open an issue on [Docker repo] and mention me `cc: @ahmetalpbalkan` in the description.

I would like to acknowledge help of the Docker developer community on getting this work reviewed and merged to Docker. We are looking forward to make it more stable and shippable soon!

There is some Windows code now living in Docker! ♥

---

Original source: [How to compile Docker on Windows](https://ahmetalpbalkan.com/blog/compiling-docker-cli-on-windows/)
