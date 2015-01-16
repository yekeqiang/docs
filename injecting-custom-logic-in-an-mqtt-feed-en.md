# Injecting Custom Logic in an MQTT feed

---

Author: Kyle Roche

---

I'm going to explore adding custom logic to a live [MQTT](http://mqtt.org/) feed. At 2lemetry, we often hear from customers that they have the need to evaluate a sensor value or inject some custom lookup into a sensor payload before it hits the subscriber. ThingFabric enables this type of inline logic in its [Rules Engine](https://2lemetry.atlassian.net/wiki/display/KB/How+to+use+the+Rules+Engine).

In this post, I'm going to inject logic into a sensor payload using a NodeJS script in a Docker Container and a basic SQL based rule on the ThingFabric MQTT platform.

## Use Case

Consider the following use case that we see fairly often in IoT applications. There is a machine in the field that lacks context of it's meaning in the application stack. For example, the machine only sends some sort of hardware identifier and the back end or subscribing systems only have context based on some other logical / non-hardware identifier.

In these cases, we typically get requests for customers to add lookups or third party logic into the MQTT message stream. Using the rules engine, we can filter out specific MQTT messages (or topics), route the messages to a HTTP based system then publish the responses back into the MQTT stream.

Subscribers now have the option to listen to only transformed messages. In addition, we can also create new rules on the transformed messages creating a sort of logical sequencing of events.

## Tools used in this tutorial:

- [Docker](https://www.docker.com/whatisdocker/): Platform to build, ship, and run applications. I started with this tutorial series.
- [Elastic Beanstalk](http://aws.amazon.com/elasticbeanstalk/): A simple service for deploying and scaling web applications. It also supports hosting of Docker containers, which we'll use in this post
- ThingFabric MQTT service: [ThingFabric](https://app.thingfabric.com/) is a sensor enablement platform for the Internet of Things. Everything we're doing here can be done w/ the free Hacker Tier account

## Our Docker Container

If you don't already have Docker installed, read through the tutorial for your operating system to install it. I'm using Docker on Mac OS X ([Boot2Docker](http://docs.docker.com/installation/mac/)).

Find a new working directory for the example. Let's first create the NodeJS application. It'll be a basic service that returns a JSON object in response to a POST request and a string in response to a GET request. Copy the contents of the following gist into a file called **index.js**.

Copy the contents of the **Package.json** gist to a file named the same.

As you can see, there's nothing really complicated happening here. We have a response to the GET request and a response to the POST request inside a basic Express application.

For Docker to run this application we need to instruct it on how to start the container and environment. Create a file called **Dockerfile** and add the contents from that gist (also below).

```
FROM    centos:centos6
 
# Enable EPEL for Node.js
RUN     rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
# Install Node.js and npm
RUN     yum install -y npm
 
# Bundle app source
COPY . /src
# Install app dependencies
RUN cd /src; npm install
 
EXPOSE  8080
CMD ["node", "/src/index.js"]
```

```
{
  "name": "node-boilerplate",
  "private": true,
  "version": "0.0.1",
  "description": "Node.js Hello world app on CentOS using docker",
  "author": "Kyle Roche",
  "dependencies": {
    "express": "3.2.4"
  }
}
```

```
var express = require('express');
 
var PORT = 8080;
 
var app = express();
app.post('/', function(req, res) {
	res.send('{"test":"newvalue"}');
});
app.get('/', function(req, res) {
	res.send('working');
});
 
app.listen(PORT);
console.log('Running on localhost:' + PORT);
```

## Deploying to Beanstalk

We're going to host our container on AWS' Elastic Beanstalk service. This isn't necessarily a tutorial on Beanstalk, so I'm going to skip the basics here. If you are new to Beanstalk, follow the [Getting Started](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.Walkthrough.html) guide.

## Creating the MQTT Rule

The ThingFabric Rule's Engine facilitates HTTP forwarding to allow for 3rd party logic to be injected into the data stream. SQL syntax is used to create filters for inbound data and remove noise. For more information on the SQL Rule syntax visit the 2lemetry Knowledge Base.

We're going to create the rule from the command line using our default MQTT domain. The rule will pick up anything from **domain/hello** and forward it to the HTTP service on Elastic Beanstalk. It will asynchronously wait for the response from the service and then forward that to the **responseTopic** we provide in the rule template.

Create a text file in a working directory. Copy the contents from the gist below to a file called **samplerule.txt**.

```
{
    "query":"SELECT * FROM '[domain]/hello'",
    "integrations": [
        {
                "type":"http",
                "url":"http://kyleroche.elasticbeanstalk.com",
                "method":"POST",
                "responseTopic":"[domain]/hello3"
        }
    ]
}
```

From the command line, execute the following command:

```
curl -u [username]:[secret] -X POST --data "@samplerule.txt" http://q.m2m.io/3/sql
```

## Testing the service end to end

Let's use the ThingFabric Portal's web based MQTT client to test our new service integration. Open your ThingFabric project and click on the **Tools** tab. By default, the wildcard (domain/#) topic will be selected. Click **Subscribe**.

![alt](http://resource.docker.cn/screen-shot-2014-12-02-at-10-10-49-am.png)

I entered **{"test":"value"}** in the Message field for the Publish dialog. Make sure you leave the [domain]/hello topic as well. Click **Publish**. You should see a few messages in your stream.

![alt](http://resource.docker.cn/screen-shot-2014-12-02-at-10-11-33-am.png)

First, we have a message in the [domain]/$SYS/rules special topic. This message was sent by ThingFabric with some debug information about the HTTP post it ran on our behalf. We can see the URL, the body, headers, response code, etc. And, we'll also see the response body that was returned by our Beanstalk application.

![alt](http://resource.docker.cn/screen-shot-2014-12-02-at-10-11-09-am.png)


## Summary

In this post, we were able to inject content from a 3rd party services into our MQTT stream. Subscribers now have the option to listen only on the resulting topic receiving only the transformed payload.

---

Original source: [Injecting Custom Logic in an MQTT feed](http://blog.kyleroche.com/2014/12/02/injecting-custom-logic-in-an-mqtt-feed/)