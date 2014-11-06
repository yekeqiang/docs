# Orchestrate Docker locally with Makefiles: Example with Interactive Brokers in Go

---

##### Author: Guillaume J. Charmes 

---



![alt](http://resource.docker.cn/interactive-brokers.jpeg) 

![alt](http://resource.docker.cn/docker-logo.png)



## Docker Orchestration

In this article, I’ll talk about basic orchestration using names and links, it is mainly for local testing, for production use, the strategy would be different.

### Names and Links

The **names** and **links** within Docker are very useful. When starting a container with `--name`, you can then manipulate it via that name instead of the ID or the generated name. It is particularly useful when you need to play with that container in the future. (An alternative would be to inject the ID in a variable, but this is very limiting).

The **links** allow you to “inject” all the information from a given container into an other. This includes exposed ports and environment variables.

#### Naming Convention / Good practice

As a convention, to avoid name collision, I always name my container with the `_c` suffix. I do not use any prefix/suffix on images.

#### Example

Start a dummy container with a name, an exposed port and an environment variable then link it to an other one.

```
#!/bin/sh
# make sure the container name is free, remove it if needed
docker rm -f mycontainer_c
docker run -d -p 4242 -e foo=bar --name mycontainer_c ubuntu:14.04 sleep 100
docker run --link mycontainer_c:mcc ubuntu:14.04 env
> PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
> HOSTNAME=912b106bf5eb
> MCC_PORT=tcp://172.17.0.55:4242
> MCC_PORT_4242_TCP=tcp://172.17.0.55:4242
> MCC_PORT_4242_TCP_ADDR=172.17.0.55
> MCC_PORT_4242_TCP_PORT=4242
> MCC_PORT_4242_TCP_PROTO=tcp
> MCC_NAME=/naughty_mayer/mcc
> MCC_ENV_foo=bar
> HOME=/root
```

### Docker build

In order to generate a usable image, Docker provides an incredible tool: Dockerfiles. It allows you to “describe” your image.

#### Example with Go

main.go

```
package main
 
func main() {
    println("hello world")
}
```

Dockerfile

```
FROM    google/golang:stable
ADD     .    /src
CMD     /src/a.out
RUN     cd /src && go build -o a.out .
```

#### Issues

While this is very powerful, in “real life”, it is often not really useful as is. What about your Database, caching, queuing or any other services your application interacts with?
This is where the names, links and Makefile come to play.

### Makefiles

Makefiles are a powerful tool, even though not designed for this usecase, we can leverage its features to accomplish easy local orchestration.

#### Rules

The feature that interest us the most are the Makefile “**rules**”, or **dependencies**. It allows you to write easily a shell script with dependencies management.

#### Caching

One of the original purpose of the Makefile is to provide compilation caching. I.e., not recompile the whole project when you change one file. 
As we are using Docker for this, we will not use the caching to that level. However, we will use it to cache already built images.

#### Conventions

For consistency, all my Makefile look alike:

- Uppercase variables 
	- NAME
	- SERVICE_IMAGE
	- SERVICE_CONT
- `all`, `clean`, `re` rules (Yes, from Epitech)
- Often test rule is present
- `all` depends on build and build services
- Images and container names usually contain $(NAME)
- All dummy files for caching are part of .gitignore and .dockerignore

#### Example

- Without Docker - Watch whole directory for changes

```
NAME   = myproject
 
all    : build
 
.built : .
        go build .
        @touch .built
 
build  : .built
 
clean  :
        @$(RM) .built
 
re     : clean all
 
.PHONY : all build clean re
```

Calling `make` will invoque the all `rule`. all depends on `build` by convention. 

`build` depends on `.built`, that rule is a bit particular as it actually represent a file on disc. It is used as placeholder to know if we need to redo the build. In a “regular” Makefile scenario, we would have source file instead. 

If `.built` exists and didn’t change (*mtime*), then we do nothing. Otherwise, executre the rule. 

`.built` depends on `.`, meaning that if anything (*mtime*) changes in the local directory, the rule will be reexecuted (upon next *make* call). 

The `.built` rule, when finish creates the `.built` file. So next time make is called, nothing will happen unless something changed within the directory.

This is not really useful as is, but now, let’s take a look at the same thing with Docker.

- With Docker

```
NAME   = myproject
 
all    : build
 
.built : .
        docker build -t $(NAME) .
        @docker inspect -f '{{.Id}}' $(NAME) > .built
 
build  : .built
 
clean  :
        @$(RM) .built
 
re     : clean all
 
.PHONY : all build clean re
```

Very straight forward, we do the same thing expect we store the ID of the resulting image in the `.built` file.

Now that we know how to play with Makefile and Docker, let’s see an actual use case with some Orchestration

## Interactive Brokers

Interactive Brokers (IB) is a brokerage company that offers extremely low fees. Where “big” companies offers things like $8 a trade or when you are lucky $5 a trade, IB is often under $0.80 per trade.
It also has the advantage to come with an API so you can do everything programatically. However, it requires their gateway (java and GUI) to be up and running.

Someone wrote a pure go library that interacts with the IB API: [http://github.com/gofinance/ib](http://github.com/gofinance/ib), however, the tests requires to have Java and for some reason, I did not manage to have them work on my machine.
This is where the **dockerize all the things** come in place: let’s Dockerize it, so anyone can use it. Without Java and without hassle.

### Dockerize

#### Main repository

It is incredibly complex to Dockerize something, especially with Go:

```
FROM            google/golang:stable
MAINTAINER      Guillaume J. Charmes <guillaume@charmes.net>
 
ADD             .         /src
```

That’s it. We are dealing with a library, so nothing will get executed. We will call the `go test` from the Makefile as they have some dependencies and can’t be ran stand alone.

#### Test server

The tests depends on the test server to be running, so let’s Dockerize it as well.

First let’s take a look at the original script to spawn the server:

```
#!/bin/bash
 
rm -rf tws
mkdir tws
pushd tws
jar -xf ../unixmacosx-943.2a.jar
pushd IBJts
java -cp ../../ibcontroller-2.12.0.jar:jts.jar:total.2013.jar -Xmx512M -XX:MaxPermSize=128M ibcontroller.IBGatewayController ../../ibcontroller-2.12.0.ini &
popd
popd
```

In order to make it work, I had to replace `jar -xf` by `unzip` and in order to avoid the Gateway to complain about missing X11, I run it within `xvfb`. My new script “docker ready” look like this:

```
#!/bin/bash
 
rm -rf tws
mkdir tws
cd tws
unzip ../unixmacosx-943.2a.jar
cd IBJts
socat TCP-LISTEN:4003,fork TCP:127.0.0.1:4002&
xvfb-run java -cp ../../ibcontroller-2.12.0.jar:jts.jar:total.2013.jar -Xmx512M -XX:MaxPermSize=128M ibcontroller.IBGatewayController ../../ibcontroller-2.12.0.ini
```

It is very similar. You will note the `socat` hack. For some reason, I did not manage to have the gateway listen on the outside (even when setting it to 0.0.0.0). So I create a simple proxy on an other port.

The Dockerfile then look like this:

```
FROM            ubuntu:14.04
MAINTAINER      Guillaume J. Charmes <guillaume@charmes.net>
 
RUN             apt-get update
RUN             apt-get install -y unzip socat xvfb gsettings-desktop-schemas openjdk-7-jre && rm -rf /var/lib/apt/lists/*
ENV             JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
 
EXPOSE          4003
 
ADD             .         /src
 
CMD             cd /src && ./ibgwdocker
```

Now that we have both the main code and the test server dockerize, we need to orchestrate.

#### Makefile

The Makefile rule system makes a perfect orchestration tool. In order to run tests, we first need to have the code dokerize (i.e. build the Dockerfile) as well as the test server. We then need the test server to be up and running and only then we can run the tests.

```
NAME            =       ib
GATEWAY_IMAGE   =       $(NAME)_gateway_test
GATEWAY_CONT    =       $(GATEWAY_IMAGE)_c
TEST_CONT       =       $(NAME)_test_c
 
all             :       test
 
.build_gw_id    :       testserver
                docker build -t $(GATEWAY_IMAGE) testserver
                @docker inspect -f '{{.Id}}' $(GATEWAY_IMAGE) > .build_gw_id
 
.gateway_id     :       .build_gw_id
                -@docker rm -f $(GATEWAY_CONT) > /dev/null 2> /dev/null || true
                -@docker rm -f $(GATEWAY_CONT)_tmp > /dev/null 2> /dev/null || true
                docker run --name $(GATEWAY_CONT) -d $(GATEWAY_IMAGE)
                @echo Wait for Gateway to be started
                @sleep 1
                @docker run --link $(GATEWAY_CONT):gw --rm --name $(GATEWAY_CONT)_tmp -t ubuntu:14.04 \
                        bash -c 'for i in {1..60}; do \
                                        echo | nc $$GW_PORT_4003_TCP_ADDR 4002 && exit 0 || (echo -n ..; sleep 1); \
                                done; \
                                echo; \
                                echo Waiting for Gateway timed out; exit 1'
                @echo
                @docker inspect -f '{{.Id}}' $(GATEWAY_IMAGE) > .gateway_id
 
gateway         :       .gateway_id
 
.build_id       :       .
                docker build -t $(NAME) .
                @docker inspect -f '{{.Id}}' $(GATEWAY_IMAGE) > .build_id
 
build           :       .build_id
 
test            :       gateway build
                -@docker rm -f $(TEST_CONT) > /dev/null 2> /dev/null || true
                docker run --link $(GATEWAY_CONT):gw --name $(TEST_CONT) -t $(NAME) bash -c 'cd /src && go test $(TESTFLAGS) -gw $$GW_PORT_4003_TCP_ADDR:4003'
 
clean           :
                -@docker rm -f $(GATEWAY_CONT) > /dev/null 2> /dev/null || true
                -@docker rm -f $(GATEWAY_CONT)_tmp > /dev/null 2> /dev/null || true
                -@docker rm -f $(TEST_CONT) > /dev/null 2> /dev/null || true
 
clean_all       :       clean
                -@rm -f .build_id .build_gw_id .gateway_id
 
re              :       clean_all all
 
 
.PHONY          :       all gateway buld test clean clean_all re
```

That is maybe a bit much at once, let’s decompose.

The goal here is to run the tests, so I made `all` to depend on `test` instead of `build` because there is no binary to build.

The `test` rule depend on the repository to be build and the gateway (testserver) to be up and running. It depends on the `build` and `gateway` rules.

`gateway` depends on `.gateway_id`, which mean that when the file `.gateway_id` exists, the rule will not be executed. If it does not exists, then call the `.gateway_id` rule, which depends on `.build_gw_id`. This means that `gateway` represent the test server runtime, but in order to run, it needs to be build before hand. 

`.build_gw_id` depends on `testserver`. So if anything changes within `testserver`, then the rule will be reexecuted. 

That rule effectively build the `testserver` image. Once built, then the `.gateway_id` rule is executed. This actually starts the test server. You will not the small shell hack that tried to connect to the port 4002. When in succeed, it means the gateway is up and running. 

We run the test server with: `docker run --name $(GATEWAY_CONT) -d $(GATEWAY_IMAGE)`, so we give it a fixed name.

Once the rule finishes, we have a test server built and up and running, now the `build` rule gets executed. 

This one is pretty straight forward: it uses `.build_id` has caching file, when it does not exists, it builds the repository image. 

When both the rules are satisfied (i.e. cached or executed), then the `test` rule starts:

```
docker run --link $(GATEWAY_CONT):gw --name $(TEST_CONT) -t $(NAME) bash -c 'cd /src && go test $(TESTFLAGS) -gw $$GW_PORT_4003_TCP_ADDR:4003'
```

We link the container with the test server one, so inside we will have access to the `$GW_PORT_4003_TCP_ADDR`, which is the actual private ip of the test server container. We give `-t` only for interactive purpose: it allows for ctrl-c to interrupt.

At this point, we have a container with the ip of the running test server, so we start the tests and give the test server IP as parameter.
See [https://github.com/gofinance/ib/pull/4](https://github.com/gofinance/ib/pull/4) for the full Pull-Request which includes the Makefile, the Dockerfiles and the code changes to accept variable test server IP.

## Other use cases

This is very useful for lot of situation, for example:
- Redis
- Postgres, Mongo (or any database)
- Elastic Search

If you need multiple services, simply add more rules. The makefile will allow you to easily describe your cascade dependencies and cache everything.

---

Original source: [Orchestrate Docker locally with Makefiles: Example with Interactive Brokers in Go](http://blog.charmes.net/2014/11/orchestrate-docker-locally-with.html)