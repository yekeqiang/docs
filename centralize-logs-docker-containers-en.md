# How to Centralize Logs from Docker Containers

---

##### Author: Jason Skowronski

---

[Docker](http://www.docker.com/) is a fast-growing platform for distributing your applications. Thousands of companies are now using Docker to shorten their release cycles and implement continuous delivery because its containers isolate dependencies and make applications easier to maintain. Loggly’s new Docker support is very timely because there’s a lot of excitement out there right now:


I will go over several ways to log from Docker and give an example of how to send syslog from an application like Nginx.

## Loggly Adds Power to Docker Logging

Docker offers a built-in logging function, but it only gives you ready access to your most recent logs in text format. Managing your Docker logs in Loggly offers you some big benefits:

- You get logs from all of your containers in one place without having to run the docker logs command.
- You can analyze your Docker logs much faster and more efficiently with our search. tools, [Loggly Dynamic Field Explorer](https://www.loggly.com/blog/new-loggly-dynamic-field-explorer-streamlines-log-analysis-for-devops/), and visual trends.
- You can set up alerts so that you know immediately when errors occur or when event counts reach a particular threshold that may signal trouble.

## You Have Several Options for Sending Docker Logs to Loggly

Loggly offers you a lot of flexibility to send us your logs in the way that best fits your unique environment – all without requiring proprietary agents, of course. Here is the best approach for each type of logging:

1. **Syslog**:

	Use the [Loggly Docker container](https://www.loggly.com/docs/docker-syslog/) that’s now available. It’s also hosted on [Docker Hub](https://registry.hub.docker.com/u/sendgridlabs/loggly-docker/). This method was recommended to us by SendGrid Labs.

2. **Custom application logs**:

	Send data through our [HTTP endpoint](https://www.loggly.com/docs/http-endpoint/): This may work well if you’re logging one of our [logging libraries](https://www.loggly.com/docs/loggly-libraries-catalog/) for common programming languages.

3. **File and socket logging**:

	Make an [explicit bind-mount](http://jpetazzo.github.io/2014/08/24/syslog-docker/) to the host: you would then have rsyslog read from that mounted volume and subsequently forward your log events to Loggly. I will save the details on this for another time.

I’ll focus this post on the first option, which will work well if your application creates syslog-compatible log events. Syslog-compatible applications include for Apache, Nginx, and Java that use logback or Log4j for syslog.

Here are the advantages of using the container:

- You maintain separation of processes (one container for one process), and independence from the host.
- You can use your resources more efficiently. A single Loggly docker container can support dozens of other containers on a host.
- You can maintain performance and reliability of your core application. Since the Loggly container runs as a separate process, you can ensure that your logs get to Loggly in a reliable way, without blocking the processing of your application.
- You have a single point to control which logs you send to Loggly. For example, you may want to set up TLS encryption as the logs leave your network or filter your logs if you only want certain log levels sent to Loggly.

## How the Loggly Docker Container Works

The Loggly Docker container is available in an [image](https://github.com/sendgridlabs/loggly-docker) provided by SendGrid. The image builds a Docker container for you. Inside the container, rsyslog is running as a process that opens up port 514, which is the rsyslog port. Docker maps this port to a high numbered one on the host.  When you set up a link to another container, Docker will insert environment variables telling it how to send syslog to the Loggly container.  Any number of Docker containers can link to it.  This many-to-one relationship is illustrated in the diagram above.

## How to Use It

Our Support Center now has [step-by-step instructions](https://www.loggly.com/docs/docker-syslog/) on how to use the Docker container.  The basic command is to run the logger docker container, insert your customer token, and map port 514 to accept syslog.

```
sudo docker run -d -p 514/udp --name loggly-docker -e TOKEN=TOKEN -e TAG=Docker sendgridlabs/loggly-docker
```

The documentation also includes an example showing how to send test data using netcat, and configure an Nginx container for syslog and to send to Loggly.  Once the data is inside Loggly, you can use our search interface, trends, and more.

![alt](http://resource.docker.cn/nignx-docker.png)

## Win at Docker Logging Now

Sending your Docker logs to Loggly will make your operations team much happier and your developers much more productive in nailing down root causes. So get the Loggly Docker Container from [GitHub](https://github.com/psquickitjayant/loggly-docker/tree/psquickit-dev); or if you don’t have Loggly yet, [sign up for a free trial](https://www.loggly.com/simplify-log-management-with-loggly/)! It’s very quick to set up.

If you’re at AWS re:Invent this week, be sure to stop by the Loggly booth for a quick demo.

---

Original source: [How to Centralize Logs from Docker Containers](https://www.loggly.com/blog/centralize-logs-docker-containers/)