# FAST AND EASY INTEGRATION TESTING WITH DOCKER AND OVERCAST

---

##### Author: Paul van der Ende

---

## Challenges With Integration Testing

Suppose that you are writing a MongoDB driver for java. To verify if all the implemented functionality works correctly, you ideally want to test it against a REAL MongoDB server. This brings a couple of challenges:

- Mongo is not written in java, so we can not embed it easily in our java application
- We need to install and configure MongoDB somewhere, and maintain the installation, or write scripts to set it up as part of our test run.
- Every test we run against the mongo server, will change the state, and tests might influence each other. We want to isolate our tests as much as possible.
- We want to test our driver against multiple versions of MongoDB.
- We want to run the tests as fast as possible. If we want to run tests in parallel, we need multiple servers. How do we manage them?

Let's try to address these challenges.

First of all, we do not really want to implement our own MonogDB driver. Many implementations exist and we will be reusing the [mongo java driver](http://docs.mongodb.org/ecosystem/drivers/java/) to focus on how one would write the integration test code.

## Overcast And Docker

logoWe are going to use [Docker](https://www.docker.com/) and [Overcast](https://github.com/xebialabs/overcast). Probably you already know Docker. It's a technology to run applications inside software containers. Overcast is the library we will use to manage docker for us. Overcast is a open source java library developed by [XebiaLabs](http://xebialabs.com/) to help you to write test that connect to cloud hosts. Overcast has support for various cloud platforms, including [EC2](http://aws.amazon.com/ec2), [VirtualBox](https://www.virtualbox.org/), [Vagrant](http://www.vagrantup.com/), [Libvirt](http://libvirt.org/) (KVM). Recently support for Docker has been added by me in Overcast version [2.4.0](http://mvnrepository.com/artifact/com.xebialabs.cloud/overcast/2.4.0).

Overcast helps you to decouple your test code from the cloud host setup. You can define a cloud host with all its configuration separately from your tests. In your test code you will only refer to a specific overcast configuration. Overcast will take care of creating, starting, provisioning that host. When the tests are finished it will also tear down the host. In your tests you will use Overcast to get the hostname and ports for this cloud host to be able to connect to them, because usually these are dynamically determined.

We will use Overcast to create Docker containers running a MongoDB server. Overcast will help us to retrieve the dynamically exposed port by the Docker host. The host in our case will always be the docker host. Docker in our case runs on an external Linux host. Overcast will use a TCP connection to communicate with Docker. We map the internal ports to a port on the docker host to make it externally available. MongoDB will internally run on port 27017, but docker will map this port to a local port in the range 49153 to 65535 ([defined by docker](https://docs.docker.com/userguide/usingdocker/)).

## Setting Up Our Tests
Lets get started. First, we need a Docker image with MongoDB installed. Thanks to the Docker community, this is as easy as reusing one of the already existing images from the Docker Hub. All the hard work of creating such an image is already done for us, and thanks to containers we can run it on any host capable of running docker containers. How do we configure Overcast to run the MongoDB container? This is our minimal configuration we put in a file called overcast.conf:

```
mongodb {
    dockerHost="http://localhost:2375"
    dockerImage="mongo:2.7"
    exposeAllPorts=true
    remove=true
    command=["mongod", "--smallfiles"]
}
```

That's all! The dockerHost is configured to be localhost with the default port. This is the default value and you can omit this. The docker image called mongo version 2.7 will be automatically pulled from the central docker registry. We set exposeAllPorts to true to inform docker it needs to dynamically map all exposed ports by the docker image. We set remove to true to make sure the container is automatically removed when stopped. Notice we override the default container startup command by passing in an extra parameter "--smallfiles" to improve testing performance. For our setup this is all we need, but overcast also has support for defining static port mappings, setting environment variables, etc. Have a look at the [Overcast documentation](https://github.com/xebialabs/overcast/blob/master/README.markdown) for more details.

How do we use this overcast host in our test code? Let's have a look at the test code that sets up the Overcast host and instantiates the mongodb client that is used by every test. The code uses the TestNG @BeforeMethod and @AfterMethod annotations.

```
private CloudHost itestHost;
private Mongo mongoClient;
 
@BeforeMethod
public void before() throws UnknownHostException {
    itestHost = CloudHostFactory.getCloudHost("mongodb");
    itestHost.setup();
 
    String host = itestHost.getHostName();
    int port = itestHost.getPort(27017);
 
    MongoClientOptions options = MongoClientOptions.builder()
        .connectTimeout(300 * 1000)
        .build();
 
    mongoClient = new MongoClient(new ServerAddress(host, port), options);
    logger.info("Mongo connection: " + mongoClient.toString());
}
 
@AfterMethod
public void after(){
    mongoClient.close();
    itestHost.teardown();
}
```

It is important to understand that the mongoClient is the object under test. Like mentioned before, we borrowed this library to demonstrate how one would integration test such a library. The itestHost is the Overcast CloudHost. In before(), we instantiate the cloud host by using the CloudHostFactory. The setup() will pull the required images from the docker registry, create a docker container, and start this container. We get the host and port from the itestHost and use them to build our mongo client. Notice that we put a high connection timeout on the connection options, to make sure the mongodb server is started in time. Especially the first run it can take some time to pull images. You can of course always pull the images beforehand. In the @AfterMethod, we simply close the connection with mongoDB and tear down the docker container.

## Writing A Test

The before and after are executed for every test, so we will get a completely clean mongodb server for every test, running on a different port. This completely isolates our test cases so that no tests can influence each other. You are free to choose your own testing strategy, sharing a cloud host by multiple tests is also possible. Lets have a look at one of the tests we wrote for mongo client:

```
@Test
public void shouldCountDocuments() throws DockerException, InterruptedException, UnknownHostException {
 
    DB db = mongoClient.getDB("mydb");
    DBCollection coll = db.getCollection("testCollection");
    BasicDBObject doc = new BasicDBObject("name", "MongoDB");
 
    for (int i=0; i &lt; 100; i++) {
        WriteResult writeResult = coll.insert(new BasicDBObject("i", i));
        logger.info("writing document " + writeResult);
    }
 
    int count = (int) coll.getCount();
    assertThat(count, equalTo(100));
}
```

Even without knowledge of MongoDB this test should not be that hard to understand. It creates a database, a new collection and inserts 100 documents in the database. Finally the test asserts if the getCount method returns the correct amount of documents in the collection. Many more aspects of the mongodb client can be tested in additional tests in this way. In our example setup, we have implemented two more tests to demonstrate this. [Our example project](https://github.com/paulvanderende/mongo-itests/blob/master/src/test/java/mongo/MongoTest.java) contains 3 tests. When you run the 3 example tests sequentially (assuming the mongo docker image has been pulled), you will see that it takes only a few seconds to run them all. This is extremely fast.

## Testing Against Multiple MongoDB Versions

We also want to run all our integration tests against different versions of the mongoDB server to ensure there are no regressions. Overcast allows you to define multiple configurations. Lets add configuration for two more versions of MongoDB:

```
defaultConfig {
    dockerHost="http://localhost:2375"
    exposeAllPorts=true
    remove=true
    command=["mongod", "--smallfiles"]
}
 
mongodb27=${defaultConfig}
mongodb27.dockerImage="mongo:2.7"
 
mongodb26=${defaultConfig}
mongodb26.dockerImage="mongo:2.6"
 
mongodb24=${defaultConfig}
mongodb24.dockerImage="mongo:2.4"
```

The default configuration contains the configuration we have already seen. The other three configurations extend from the defaultConfig, and define a specific mongoDB image version. Lets also change our test code a little bit to make the overcast configuration we use in the test setup depend on a parameter:

```
@Parameters("overcastConfig")
@BeforeMethod
public void before(String overcastConfig) throws UnknownHostException {
    itestHost = CloudHostFactory.getCloudHost(overcastConfig);
```

Here we used the paramaterized tests feature from TestNG. We can now define a TestNG suite to define our test cases and how to pass in the different overcast configurations. Lets have a look at our TestNG suite definition:

```
<suite name="MongoSuite" verbose="1">
    <test name="MongoDB27tests">
        <parameter name="overcastConfig" value="mongodb27"/>
        <classes>
            <class name="mongo.MongoTest" />
        </classes>
    </test>
    <test name="MongoDB26tests">
        <parameter name="overcastConfig" value="mongodb26"/>
        <classes>
            <class name="mongo.MongoTest" />
        </classes>
    </test>
    <test name="MongoDB24tests">
        <parameter name="overcastConfig" value="mongodb24"/>
        <classes>
            <class name="mongo.MongoTest" />
        </classes>
    </test>
</suite>
```

With this test suite definition we define 3 test cases that will pass a different overcast configuration to the tests. The overcast configuration plus the TestNG configuration enables us to externally configure against which mongodb versions we want to run our test cases.

## Parallel Test Execution

Until this point, all tests will be executed sequentially. Due to the dynamic nature of cloud hosts and docker, nothing limits us to run multiple containers at once. Lets change the TestNG configuration a little bit to enable parallel testing:

```
<suite name="MongoSuite" verbose="1" parallel="tests" thread-count="3">
```

This configuration will cause all 3 test cases from our test suite definition to run in parallel (in other words our 3 overcast configurations with different MongoDB versions). Lets run the tests now from IntelliJ and see if all tests will pass:

![alt](http://resource.docker.cn/screenshot-2014-10-08.png)

We see 9 executed test, because we have 3 tests and 3 configurations. All 9 tests have passed. The total execution time turned out to be under 9 seconds. That's pretty impressive!

During test execution we can see docker starting up multiple containers (see next screenshot). As expected it shows 3 containers with a different image version running simultaneously. It also shows the dynamic port mappings in the "PORTS" column:

![alt](http://resource.docker.cn/screen-shot-2014-10-08.png)

That's it!

## Summary

To summarise, the advantages of using Docker with Overcast for integration testing are:

1. Minimal setup. Only a docker capable host is required to run the tests.
2. Save time. Minimal amount of configuration and infrastructure setup required to run the integration tests thanks to the docker community.
3. Isolation. All test can run in their isolated environment so the tests will not affect each other.
4. Flexibility. Use multiple overcast configuration and parameterized tests for testing against multiple versions.
5. Speed. The docker container starts up very quickly, and overcast and testng allow you to even parallelize the testing by running multiple containers at once.

The example code for our integration test project is available here. You can use Boot2Docker to setup a docker host on Mac or Windows.

Happy testing!

>  Note: Due to a bug in the gradle parallel test runner you might run into this [random failure](http://forums.gradle.org/gradle/topics/when-running-a-suite-in-parallel-i-sometimes-get-this-error-received-a-completed-event-for-test-with-unknown-id) when you run the example test code yourself. The work around is to disable parallelism or use a different test runner like IntelliJ or maven.

---

Original source: [FAST AND EASY INTEGRATION TESTING WITH DOCKER AND OVERCAST](http://blog.xebia.com/2014/10/13/fast-and-easy-integration-testing-with-docker-and-overcast/)