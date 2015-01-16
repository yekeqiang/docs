# Building Docker Java images with Maven

---

Author: Ladislav Gazo

---

It has been a while since Docker was introduced as a stable platform for container management. Over the time various ways and procedures of how to deploy Java applications evolved as well. Several plugins in Maven or Gradle world emerged.

I am always surprised how many projects can be created for one and still the same thing in the open source community. For building Docker images from Maven there are at least these three to mention:


- [alexec/docker-maven-plugin](https://github.com/alexec/docker-maven-plugin)

	Contribute to docker-maven-plugin development by creating an account on GitHub.

- [rhuss/docker-maven-plugin](https://github.com/rhuss/docker-maven-plugin)

	docker-maven-plugin - Maven plugin for managing Docker images and containers

- [spotify/docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)

	docker-maven-plugin - A maven plugin for docker


But you can find whole abstracts and articles containing others…

In our SiteHero product I tried to use couple of these. It is Maven-based project. I wanted to filter some resources and inject some properties into the scripts that were going to be part of the image. Unfortunately almost every single plugin introduced its own way how to write Dockerfiles. But I wanted to write Dockerfile with Dockerfile syntax because it is so damn simple and clearly documented. And in the end the whole process is going to be built in CI anyway.

Therefore I realized I don’t need these plugins. Maven already comes with plugins like maven-resources-plugin or maven-antrun-plugin that can help me to achieve the goal -> produce clean basis for Docker context. That means to have final Dockerfile with all necessary scripts and configurations in the resulting build directory of my Java project.

So I have created a profile … named ‘docker’ ;)

```
<profile>
	<id>docker</id>
	<activation>
		<file>
			<exists>${basedir}/.docker</exists>
		</file>
	</activation>
	<build>
		<plugins>
			<plugin>
				<artifactId>maven-resources-plugin</artifactId>
				<executions>
					<execution>
						<id>copy-resources</id>
						<phase>validate</phase>
						<goals>
							<goal>copy-resources</goal>
						</goals>
						<configuration>
							<outputDirectory>${basedir}/target</outputDirectory>
							<resources>
								<resource>
									<directory>src/main/docker</directory>
									<filtering>true</filtering>
								</resource>
							</resources>
						</configuration>
					</execution>
				</executions>
			</plugin>
			
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-antrun-plugin</artifactId>
				<version>1.6</version>
				<executions>
					<execution>
						<id>process-classes</id>
						<phase>process-classes</phase>
						<configuration>
							<target>
								<chmod file="target/bin/start.sh" perm="755"/>
							</target>
						</configuration>
						<goals>
							<goal>run</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</profile>
```

I have put all the files in the `/src/main/docker` directory with Dockerfile in it’s root. This structure conforms to the layout prescribed by Maven. These files are processed and copied over to `/target`.

Our Jenkins CI docker image job takes built files within /target directory of the build job and executes standard `docker build` command. The produced image is pushed to our private registry where QA process follows.

In case you do not want to introduce additional complexity with new plugins you can keep talking to Docker in its own language and use existing Maven plugins to bridge both worlds. In the end it is the job of your CI to overlook the whole deployment workflow so let the right tool do its job.

---

Original source: [Building Docker Java images with Maven](https://medium.com/@ladislavGazo/building-docker-java-app-images-with-maven-bdd88305abb)