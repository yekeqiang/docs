# How To Deploy A Docker Application Into Production On Amazon AWS

---

##### Author: Mark van Holsteijn

---

Docker reached production status a few months ago. But having the container technology alone is not enough. You need a complete platform infrastructure before you can deploy your docker application in production. Amazon AWS offers exactly that: a production quality platform that offers capacity provisioning, load balancing, scaling, and application health monitoring for Docker applications.

In this blog, you will learn how to deploy a Docker application to production in five easy steps.

For demonstration purposes, you are going to use the node.js application that was build for CloudFoundry and [used to demonstrate Deis in a previous post](http://blog.xebia.com/2014/06/18/deploying-a-node-js-app-to-docker-on-coreos-using-deis/). A truly useful app of which the sources are available on [github](https://github.com/mvanholsteijn/sample_nodejs_cf).


## 1. Create A Dockerfile

First thing you need to do is to create a Dockerfile to create an image. This is quite simple: you install the node.js and npm packages, copy the source files and install the javascript modules.

```
# DOCKER-VERSION 1.0
FROM    ubuntu:latest
#
# Install nodejs npm
#
RUN apt-get update
RUN apt-get install -y nodejs npm
#
# add application sources
#
COPY . /app
RUN cd /app; npm install
#
# Expose the default port
#
EXPOSE  5000
#
# Start command
#
CMD ["nodejs", "/app/web.js"]
```

## 2. Test Your Docker Application

Now you can create the Docker image and test it.

```
$ docker build -t sample-nodejs-cf .
$ docker run -d -p 5000:5000 sample-nodejs-cf
```

Point your browser at [http://localhost:5000](http://localhost:5000/), click the 'start' button and Presto!

## 3. Zip The Sources

Now you know that the instance works, you zip the source files. The image will be build on Amazon AWS based on your Dockerfile.

```
$ zip -r /tmp/sample-nodejs-cf-srcs.zip .
```

## 4. Deploy Docker Application To Amazon AWS

Now you [install](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [configure](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) the amazon AWS command line interface (CLI) and deploy the docker source files to elastic beanstalk.  You can do this all manually, but here you use the script [deploy-to-aws.sh](https://github.com/mvanholsteijn/sample_nodejs_cf/blob/master/deploy-to-aws.sh) that I created.

```
$ deploy-to-aws.sh \
         sample-nodejs-cf \
         /tmp/sample-nodejs-cf-srcs.zip \
         demo-env
```

After about 8-10 minutes your application is running. The output should look like this..

```
INFO: creating application sample-nodejs-cf
INFO: Creating environment demo-env for sample-nodejs-cf
INFO: Uploading sample-nodejs-cf-srcs.zip for sample-nodejs-cf, version 1412948762.
upload: ./sample-nodejs-cf-srcs.zip to s3://elasticbeanstalk-us-east-1-233211978703/1412948762-sample-nodejs-cf-srcs.zip
INFO: Creating version 1412948762 of application sample-nodejs-cf
INFO: demo-env in status Launching, waiting to get to Ready..
...
INFO: demo-env in status Launching, waiting to get to Ready..
INFO: Updating environment demo-env with version 1412948762 of sample-nodejs-cf
INFO: demo-env in status Updating, waiting to get to Ready..
...
INFO: demo-env in status Updating, waiting to get to Ready..
INFO: Version 1412948762 of sample-nodejs-cf deployed in environment
INFO: current status is Ready, goto http://demo-env-vm2tqi3qk4.elasticbeanstalk.com
```

## 5. Test Your Docker Application On The Internet!

Your application is now available on the Internet. Browse to the designated URL and click on start. When you increase the number of instances at Amazon, they will appear in the application. When you deploy a new version of the application, you can observe how new versions of the application  appear without any errors on the client application.

For more information, goto [Amazon Elastic Beanstalk adds Docker support](http://aws.amazon.com/about-aws/whats-new/2014/04/23/aws-elastic-beanstalk-adds-docker-support/), and [Dockerizing a Node.js Web App](https://docs.docker.com/examples/nodejs_web_app/).

---

Original source: [How To Deploy A Docker Application Into Production On Amazon AWS](http://blog.xebia.com/2014/10/17/how-to-deploy-a-docker-application-into-production-on-amazon-aws/)