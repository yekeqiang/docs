# Automate Docker and enhance your DevOps flows with HP Operations Orchestration

---

Author: Meir Wahnon, HP Software (RnD)

---

If you’ve built a DevOps process before, you’ve probably experienced how complex it can be. You can easily find yourself maintaining a pile of different scripts, configurations, tools and pieces of code, each one with its own specific flavors, benefits and disadvantages to consider.
 
One new and emerging tool that’s getting a lot of attention in the DevOps space is Docker, which is designed to be an easy way to ship your services in isolated, lightweight and portable containers.
 
But just how easy is it? In this post, I will show how to use Docker to create a simple web app with a MySQL database, and then how you can [use the free HP Operations Orchestration Community Edition](http://hpsw.co/Rm3b9L5) to automate the part of your DevOps flow that links the application container to the database container.
 
## 1. Spring Boot App

First, I created a simple Spring Boot application that uses a Tomcat web server and MySQL as its database. I like Spring Boot because it brings the “convention over configuration” approach to the Java world.
 
I also decided to take the Docker approach of one container per service, so my app will reside in one container, and my DB container (MySQL) in another, with OO helping me to link them. You can [check out the app codebase in GitHub](https://github.com/meirwa/spring-boot-tomcat-mysql-app).
 
Let’s take a look at my app Dockerfile:

```  
# Start from a clean Ubuntu image
FROM ubuntu:latest
MAINTAINER Meir Wahnon
 
# Install Java + maven on it
RUN apt-get update
RUN apt-get install default-jre -y
RUN apt-get install default-jdk -y
RUN apt-get install maven –y
 
# add app sources to image
ADD pom.xml /app/
ADD src/ /app/src/
 
# declaring app working directory
WORKDIR /app/
 
# Build app
RUN mvn package
 
# Expose webapp port
EXPOSE 8080
 
# Run the app jar
CMD ["java","-jar","target/spring-boot-sample-tomcat-1.1.5.RELEASE.jar"]
```
  
You can see that the Spring Boot build (the ‘mvn package’ command) bundles the whole application, including the Tomcat web server, into a single jar, so starting our app is easy.
 
Since the app uses a repository (I used spring-data-rest for it), I used MySQL as my DB, so I added an application.properties file where I declared the DB+ hibernate properties: 

``` 
spring.datasource.url=jdbc:mysql://${DB_URL:localhost}:${DB_PORT:3306}/${DB_NAME:boot}
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.username=${DB_USER:user}
spring.datasource.password=${DB_USER_PASS:pass}
 
# Hibernate
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
```
  
Notice that here I use environment variables (DB_URL, DB_PORT, and DB_NAME). These will be important for the linking stage that I address later in this post.
 
## 2.   Integrate GitHub project with Docker Hub
 
Since we want that after each commit to GitHub, a new Docker image (of the app) will be built out of it, I used the feature of automated builds on Docker Hub. (It’s very easy, [click here to see how to do it](http://docs.docker.com/docker-hub/builds/#setting-up-automated-builds-with-github).)
 
You can view my image here on Docker’s registry hub.
 
To get the latest app image, all you need to do is run this command from your Docker engine:  

``` 
docker pull meirwa/spring-boot-tomcat-mysql-app
```

   
## 3.   Link the containers
 
Okay, so now you have an updated docker image of the app, but this app image does not include the DB. You need another container for my DB. For that, I used the official MySQL image (“docker pull mysql”).
After the image is downloaded to the host, you need to start it by running:  

``` 
docker run --name mysqldb -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_DATABASE=boot -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -d mysql
```
  
Let’s review the parameters in this command:
  
- **`--name mysqldb`** — used to give a name to the container, so that the app container can link to the DB using this name
 
- **`MYSQL_ROOT_PASSWORD`** — admin password, a requirement for this image
 
- **`MYSQL_DATABASE`** — name of the database to create for the app
 
- **`MYSQL_USER + MYSQL_PASSWORD`** — user name and password for the app connection
 
Now that your DB is up and running, you are left to start an app container that is linked to the DB container:

``` 
docker run --name spring-boot-app --link mysqldb:mysql -p 8080:8080 -d meirwa/spring-boot-tomcat-mysql-app
```
 
Here is what is happening in this command:
 
- **`--link mysqldb:mysql`** — will create the link between the DB container to the app container. Note that the **`mysqldb`** is the container name, while mysql is a prefix to the environment exposed to the app container
 
- **`-p 8080:8080`** — just exposing the port for the app
 
But something is missing! Note that the environment variables created by the linkage do not match the environment the app is expecting. Let’s look at the DB_URL, for instance: the environment variable the app is expecting is “DB_URL”, but this information will reside in an environment variable with the key “MYSQL_PORT_3306_TCP_ADDR”. 
 
So how do you solve this? There are several potential solutions:
 
- 1. Get the DB container IP after its creation, and then serve it as environment variable to the app
- 2. Run a script in app to rename the environment variable “MYSQL_PORT_3306_TCP_ADDR” to “DB_URL”.
- 3. Use some other tool (like fig)
  
Let’s go for the easy one and just run: 

``` 
docker inspect --format='{{.NetworkSettings.IPAddress}}' mysqldb
```
 
The return value will be the IP of the DB container, so you can serve it to the app. The command will be:

``` 
docker run --name spring-boot-app --link mysqldb:mysql -e DB_URL=${DbContainerIP} -p 8080:8080 -d meirwa/spring-boot-tomcat-mysql-app
```
  
Where DbContainerIP is the result of previous command.
And that’s it, now you have an application on port 8080 up and running that uses MySQL DB.
 
### How to automate the process using OO

You can easily create and link the same containers using OO with the “build app flow” function in the OO studio:
 
 
![alt](http://resource.docker.cn/meir-1.png)
 
That’s it, the same flow as described in previous section was created by just wiring up steps and configuring the inputs and outputs relevant for each step. You can find all of this in the [Docker content pack from the OO community](https://hpln.hp.com/contentoffering/community-devops).
 
Once you build the flow, it’s easy to automatically trigger it by REST API, or via manual trigger. It’s also easy to extend the flow (i.e. add more steps) and maintain it, because the visual structure makes it straightforward to understand and modify.
 
Once you trigger it in central (management component) you get a report of your run:
 
![alt](http://resource.docker.cn/meir-2.png)
 
And there you have it, we have created a simple web app that uses MySQL DB, and you can see how easy it to build our app and link our app container to the DB container using OO.
 
One note: Building the jar in the same image as the image you start the app is inefficient (the image size gets bigger since it holds both the app sources and the Java JDK, while all you need is the final jar and JRE. You can improve this by either building the jar in a separate image or using a tool like Jenkins.
 
### Next post: Extend the flow

In my next blog post, I will extend this flow to include:

- 1. Validate our app is up and running
- 2. Test the web app using REST API
- 3. Clear the environment (destroy and remove containers)
 
### Learn more

First, download and play with [the latest Operations Orchestration Community Edition](http://hpsw.co/Rm3b9L5). It’s free!

Next, read about [all that Operations Orchestration has to offer](http://hpsw.co/Sq3p4PZ).

Come, join us at [HP Discover in Barcelona](http://h30614.www3.hp.com/Discover), and see first hand Operations Orchestration in action.

---

Original source: [Automate Docker and enhance your DevOps flows with HP Operations Orchestration](http://h30499.www3.hp.com/t5/Grounded-in-the-Cloud/Automate-Docker-and-enhance-your-DevOps-flows-with-HP-Operations/ba-p/6672688#.VG75rFeUdlb)