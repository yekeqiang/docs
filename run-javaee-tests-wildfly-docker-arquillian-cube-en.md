# Run Java EE Tests on Docker using Arquillian Cube (Tech Tip #62)

---

Author: Arun Gupta

---

[Tech Tip #61](http://blog.arungupta.me/2014/12/javaee7-lab-wildfly-docker/) showed how to run Java EE 7 Hands-on Lab using Docker. The Dockerfile [used there](https://github.com/arun-gupta/docker-images/blob/master/javaee7-hol/Dockerfile) can be used to create a new image that can deploy any Java EE 7 WAR file to the WildFly instance running in the container.

For example [github.com/arun-gupta/docker-images/blob/master/javaee7-test/Dockerfile](http://github.com/arun-gupta/docker-images/blob/master/javaee7-test/Dockerfile) can be copied to the root directory of [javaee7-samples](https://github.com/javaee-samples/javaee7-samples) and be used to deploy jaxrs-client.war file to the container. Of course, you first need to build the sample as:


```
mvn -f jaxrs/jaxrs-client/pom.xml clean package -DskipTests
```
 
The exact Dockerfile is shown here:


```
FROM arungupta/wildfly-centos
ADD jaxrs/jaxrs-client/target/jaxrs-client.war /opt/wildfly/standalone/deployments/
```
 
If you want to deploy another Java EE 7 application, then you need to do the following steps:

- Create the WAR file of the sample
- Change the Dockerfile
- Build the image
- Stop the previous container
- Start the new container

Now, if you want to run tests against this instance then mvn test alone will not do it because either you need to bind the IP address of the Docker container statically, or dynamically find out the address and then patch it at runtime. Anyway, the repeated cycle is little too cumbersome. How do you solve it?

Meet [Arquillian Cube](http://arquillian.org/blog/2014/11/17/arquillian-cube-1-0-0-Alpha1/)!

Arquillian Cube allows you to control the lifecycle of Docker images as part of the test lifecyle, either automatically or manually.

The [blog entry](http://arquillian.org/blog/2014/11/17/arquillian-cube-1-0-0-Alpha1/) provide more details about getting started with Arquillian Cube, and this functionality has now been enabled in [“docker” branch of javaee7-samples](https://github.com/javaee-samples/javaee7-samples/tree/docker). Arquillian Cube Extension Alpha2 was [recently released](http://arquillian.org/blog/2014/12/10/arquillian-cube-1-0-0-Alpha2/) and is used to provide integration. Here are the key concepts:

- A new “wildfly-docker-arquillian” profile is being introduced
- The profile adds a dependency on:

```
<dependency>
    <groupId>org.arquillian.cube</groupId>
    <artifactId>arquillian-cube-docker</artifactId>
    <version>1.0.0.Alpha2</version>
    <scope>test</scope>
</dependency>
```
 
- Uses Docker REST API to talk to the container. [Complete API docs](https://docs.docker.com/reference/api/docker_remote_api_v1.15/) shows the sample payloads and explains the query parameters and status codes.
- Uses WildFly remote adapter to talk to the application server running within the container
- Configuration for Docker image is specified as part of maven-surefire-plugin.:

```
<configuration>
    <systemPropertyVariables>
    <arquillian.launch>wildfly-docker</arquillian.launch>
    <arq.container.wildfly-docker.configuration.username>admin</arq.container.wildfly-docker.configuration.username>
    <arq.container.wildfly-docker.configuration.password>Admin#70365</arq.container.wildfly-docker.configuration.password>
    <arq.extension.docker.serverVersion>1.15</arq.extension.docker.serverVersion>
    <arq.extension.docker.serverUri>http://127.0.0.1:2375</arq.extension.docker.serverUri>
    <arq.extension.docker.dockerContainers>
        wildfly-docker:
            image: arungupta/javaee7-samples-wildfly
            exposedPorts: [8080/tcp, 9990/tcp]
            await:
                strategy: polling
                sleepPollingTime: 50000
                iterations: 5
            portBindings: [8080/tcp, 9990/tcp]
    </arq.extension.docker.dockerContainers>
    </systemPropertyVariables>
</configuration>
``` 

Username and password are specified are for the WildFly in [arungupta/javaee7-samples-wildfly](https://github.com/javaee-samples/javaee7-samples/blob/docker/Dockerfile) image. All the configuration values can be overridden by arquillian.xml for each test case, as [explained here](http://arquillian.org/blog/2014/11/17/arquillian-cube-1-0-0-Alpha1/).
How do you try out this functionality?


```
git clone https://github.com/javaee-samples/javaee7-samples.git
git checkout docker
mvn test -f servlet/simple-servlet/pom.xml -Pwildfly-docker-arquillian
```
 
Here is a complete log of running simple-servlet test:


```
Running org.javaee7.servlet.metadata.complete.SimpleServletTest
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Dec 04, 2014 11:19:51 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 1 * Sending client request on thread main
1 > GET http://127.0.0.1:2375/v1.15/_ping
 
Dec 04, 2014 11:19:51 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 2 * Client response received on thread main
2 < 200
2 < Content-Length: 2
2 < Content-Type: application/json; charset=utf-8
2 < Date: Thu, 04 Dec 2014 19:19:51 GMT
OK
 
Dec 04, 2014 11:19:51 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 3 * Sending client request on thread main
3 > POST http://127.0.0.1:2375/v1.15/containers/create?name=wildfly-docker
3 > Accept: application/json
3 > Content-Type: application/json
{"name":"wildfly-docker","Hostname":"","User":"","Memory":0,"MemorySwap":0,"CpuShares":0,"AttachStdin":false,"AttachStdout":false,"AttachStderr":false,"PortSpecs":null,"Tty":false,"OpenStdin":false,"StdinOnce":false,"Env":null,"Cmd":null,"Dns":null,"Image":"arungupta/javaee7-samples-wildfly","Volumes":{},"VolumesFrom":[],"WorkingDir":"","DisableNetwork":false,"ExposedPorts":{"8080/tcp":{},"9990/tcp":{}}}
 
Dec 04, 2014 11:19:51 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 4 * Client response received on thread main
4 < 201
4 < Content-Length: 90
4 < Content-Type: application/json
4 < Date: Thu, 04 Dec 2014 19:19:51 GMT
{"Id":"d2fc85815256be7540ae85fef1ecb26a666a41a591e2adfae8aa6a32fde3393b","Warnings":null}
 
 
Dec 04, 2014 11:19:51 AM org.arquillian.cube.impl.docker.DockerClientExecutor assignPorts
INFO: Only exposed port is set and it will be used as port binding as well. 8080/tcp
Dec 04, 2014 11:19:51 AM org.arquillian.cube.impl.docker.DockerClientExecutor assignPorts
INFO: Only exposed port is set and it will be used as port binding as well. 9990/tcp
Dec 04, 2014 11:19:52 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 5 * Sending client request on thread main
5 > POST http://127.0.0.1:2375/v1.15/containers/wildfly-docker/start
5 > Accept: application/json
5 > Content-Type: application/json
{"containerId":"wildfly-docker","Binds":[],"Links":[],"LxcConf":null,"PortBindings":{"8080/tcp":[{"HostIp":"","HostPort":"8080"}],"9990/tcp":[{"HostIp":"","HostPort":"9990"}]},"PublishAllPorts":false,"Privileged":false,"Dns":null,"DnsSearch":null,"VolumesFrom":null,"NetworkMode":"bridge","Devices":null,"RestartPolicy":null,"CapAdd":null,"CapDrop":null}
 
Dec 04, 2014 11:19:52 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 6 * Client response received on thread main
6 < 204
6 < Date: Thu, 04 Dec 2014 19:19:52 GMT
 
Dec 04, 2014 11:19:52 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 7 * Sending client request on thread main
7 > GET http://127.0.0.1:2375/v1.15/containers/wildfly-docker/json
7 > Accept: application/json
 
Dec 04, 2014 11:19:52 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 8 * Client response received on thread main
8 < 200
8 < Content-Type: application/json
8 < Date: Thu, 04 Dec 2014 19:19:52 GMT
8 < Transfer-Encoding: chunked
{"Args":["-b","0.0.0.0","-bmanagement","0.0.0.0"],"Config":{"AttachStderr":false,"AttachStdin":false,"AttachStdout":false,"Cmd":["/opt/wildfly/bin/standalone.sh","-b","0.0.0.0","-bmanagement","0.0.0.0"],"CpuShares":0,"Cpuset":"","Domainname":"","Entrypoint":null,"Env":["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin","WILDFLY_VERSION=8.1.0.Final","JBOSS_HOME=/opt/wildfly"],"ExposedPorts":{"8080/tcp":{},"9990/tcp":{}},"Hostname":"d2fc85815256","Image":"arungupta/javaee7-samples-wildfly","Memory":0,"MemorySwap":0,"NetworkDisabled":false,"OnBuild":null,"OpenStdin":false,"PortSpecs":null,"SecurityOpt":null,"StdinOnce":false,"Tty":false,"User":"wildfly","Volumes":null,"WorkingDir":""},"Created":"2014-12-04T19:19:51.7226858Z","Driver":"devicemapper","ExecDriver":"native-0.2","HostConfig":{"Binds":[],"CapAdd":null,"CapDrop":null,"ContainerIDFile":"","Devices":null,"Dns":null,"DnsSearch":null,"ExtraHosts":null,"Links":null,"LxcConf":null,"NetworkMode":"bridge","PortBindings":{"8080/tcp":[{"HostIp":"","HostPort":"8080"}],"9990/tcp":[{"HostIp":"","HostPort":"9990"}]},"Privileged":false,"PublishAllPorts":false,"RestartPolicy":{"MaximumRetryCount":0,"Name":""},"VolumesFrom":null},"HostnamePath":"/var/lib/docker/containers/d2fc85815256be7540ae85fef1ecb26a666a41a591e2adfae8aa6a32fde3393b/hostname","HostsPath":"/var/lib/docker/containers/d2fc85815256be7540ae85fef1ecb26a666a41a591e2adfae8aa6a32fde3393b/hosts","Id":"d2fc85815256be7540ae85fef1ecb26a666a41a591e2adfae8aa6a32fde3393b","Image":"3d08e8466496412daadeba7bb35b5b64d29b32adedd64472ad775d6da5011913","MountLabel":"system_u:object_r:svirt_sandbox_file_t:s0:c34,c113","Name":"/wildfly-docker","NetworkSettings":{"Bridge":"docker0","Gateway":"172.17.42.1","IPAddress":"172.17.0.7","IPPrefixLen":16,"MacAddress":"02:42:ac:11:00:07","PortMapping":null,"Ports":{"8080/tcp":[{"HostIp":"0.0.0.0","HostPort":"8080"}],"9990/tcp":[{"HostIp":"0.0.0.0","HostPort":"9990"}]}},"Path":"/opt/wildfly/bin/standalone.sh","ProcessLabel":"system_u:system_r:svirt_lxc_net_t:s0:c34,c113","ResolvConfPath":"/var/lib/docker/containers/d2fc85815256be7540ae85fef1ecb26a666a41a591e2adfae8aa6a32fde3393b/resolv.conf","State":{"ExitCode":0,"FinishedAt":"0001-01-01T00:00:00Z","Paused":false,"Pid":11406,"Restarting":false,"Running":true,"StartedAt":"2014-12-04T19:19:52.378418242Z"},"Volumes":{},"VolumesRW":{}}
 
 
Dec 04, 2014 11:20:44 AM org.xnio.Xnio <clinit>
INFO: XNIO version 3.2.0.Beta2
Dec 04, 2014 11:20:44 AM org.xnio.nio.NioXnio <clinit>
INFO: XNIO NIO Implementation Version 3.2.0.Beta2
Dec 04, 2014 11:20:44 AM org.jboss.remoting3.EndpointImpl <clinit>
INFO: JBoss Remoting version (unknown)
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 65.635 sec - in org.javaee7.servlet.metadata.complete.SimpleServletTest
Dec 04, 2014 11:20:54 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 9 * Sending client request on thread main
9 > POST http://127.0.0.1:2375/v1.15/containers/wildfly-docker/stop?t=10
9 > Accept: application/json
9 > Content-Type: application/json
 
Dec 04, 2014 11:21:04 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 10 * Client response received on thread main
10 < 204
10 < Date: Thu, 04 Dec 2014 19:21:04 GMT
 
Dec 04, 2014 11:21:04 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 11 * Sending client request on thread main
11 > DELETE http://127.0.0.1:2375/v1.15/containers/wildfly-docker?v=0&force=0
11 > Accept: application/json
 
Dec 04, 2014 11:21:05 AM org.glassfish.jersey.filter.LoggingFilter log
INFO: 12 * Client response received on thread main
12 < 204
12 < Date: Thu, 04 Dec 2014 19:21:05 GMT
 
 
Results :
 
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
 
[INFO] 
[INFO] --- maven-surefire-plugin:2.17:test (spock-test) @ simple-servlet ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1:27.831s
[INFO] Finished at: Thu Dec 04 11:21:05 PST 2014
[INFO] Final Memory: 21M/59M
[INFO] ------------------------------------------------------------------------
```
 
REST payload from the client to Docker server are shown here. This was verified on a Fedora 20 Virtual Box image. Here are some quick notes on setting it up there:

1. Install the required packages

```
yum install docker-io git maven
yum upgrade selinux-policy
```
 
2. Configure Docker

```
sudo vi /etc/sysconfig/docker
Change to "OPTIONS=--selinux-enabled -H tcp://127.0.0.1:2375 -H unix:///var/run/docker.sock"
sudo service docker start
```
 
3. Verify Docker TCP configuration

```
docker -H tcp://127.0.0.1:2375 version
 
Client version: 1.3.1
Client API version: 1.15
Go version (client): go1.3.3
Git commit (client): 4e9bbfa/1.3.1
OS/Arch (client): linux/amd64
Server version: 1.3.1
Server API version: 1.15
Go version (server): go1.3.3
Git commit (server): 4e9bbfa/1.3.1
```
 
[Boot2docker](http://boot2docker.io/) on Mac still has [issue #49](https://github.com/arquillian/arquillian-cube/issues/49), this is Alpha2 after all :-)

Try some other Java EE 7 tests and [file bugs here](https://github.com/arquillian/arquillian-cube/issues).

Enjoy!

---

Original source: [Run Java EE Tests on Docker using Arquillian Cube (Tech Tip #62)](http://blog.arungupta.me/2014/12/run-javaee-tests-wildfly-docker-arquillian-cube/)

