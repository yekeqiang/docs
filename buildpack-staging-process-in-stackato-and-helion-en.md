# Buildpack staging process in Stackato and Helion

---

##### Author: Daniel Watrous

---

One of the major strengths of [CloudFoundry](http://software.danielwatrous.com/overview-of-cloudfoundry/) was the adoption of [buildpacks](https://devcenter.heroku.com/articles/buildpacks). A buildpack represents a blueprint which defines all runtime requirements for an application. These may include web server, application, language, library and any other requirements to run an application. There are many buildpacks available for common environments, such as Java, PHP, Python, Go, etc. It is also easy to fork an existing buildpack or create a new buildpack.

When an application is pushed to CloudFoundry, there is a staging step and a deploy step, as shown below. The buildpack comes in to play when an application is staged.

![alt](http://resource.docker.cn/cloudfoundry-stage-deploy.png)

## Staging, or droplet creation



A droplet is a compressed tar file which contains all application files and runtime dependencies to run the application. Depending on the technology, this may include source code for interpreted languages, like PHP, or bytecode or even compiled objects, as with Python or Java. It will also include binary executables needed to run the application files. For example, in the case of Java, this would be a Java Runtime Environment (JRE).

The staging process, which produces a droplet that is ready to deploy, is shown in the following activity diagram.

![alt](http://resource.docker.cn/helion-app-staging.png)

### Staging environment

The staging process happens on a docker instance created using the same docker image used to deploy the app. Once created, all application files are copied to the new docker staging instance. Among these application files may be a deployment definition in the form of a [manifest.yml](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html) or [stackato.yml](https://docs.stackato.com/user/deploy/stackatoyml.html). The YAML files can include information that will prepare or alter the staging environment, including installation of system libraries. The current docker image is based on Ubuntu.

Next we get to the buildpack. Most installations of CloudFoundry include a number of buildpacks for common technologies, such as PHP, Java, Python, Go, Node and Ruby. If no manifest is included, or if there is a manifest which doesn’t explicitly identify a buildpack, then CloudFoundry will loop through all available buildpacks and run the detect script to find a match.

If the manifest explicitly identifies a buildpack, which must be hosted in a GIT repository, then that buildpack will be cloned onto the staging instance and the compile process will proceed immediately.

#### environment variables

[CloudFoundry makes certain environment variables](https://docs.stackato.com/user/reference/environment.html) available to the docker instance used for staging and deploying apps. These should be treated as dynamic, since every instance (stage or deploy) will have different variables. For example, the PORT will be different for every host.

Here is a sample of the environment available during the staging step:

```
USER=stackato
HOME=/home/stackato
PWD=/tmp/staged
SHELL=/bin/bash
SSH_CLIENT=172.17.42.1 41465 22
STACKATO_LOG_FILES=stdout=/home/stackato/logs/stdout.log:stderr=/home/stackato/logs/stderr.log
STACKATO_APP_NAME=static-test
STACKATO_APP_NAME_UPCASE=STATIC_TEST
STACKATO_DOCKER=True
LC_ALL=en_US.UTF-8
http_proxy=http://10.0.0.13:8123
https_proxy=http://10.0.0.13:8123
no_proxy=
VCAP_APPLICATION={"limits":{"mem":40,"disk":200,"fds":16384,"allow_sudo":false},"application_version":"79eb8896-37e4-4a22-8d0f-f14bd3fbe7e5","application_name":"static-test","application_uris":["static-test.stackato.danielwatrous.com"],"version":"79eb8896-37e4-4a22-8d0f-f14bd3fbe7e5","name":"static-test","space_name":"EA","space_id":"fb8ea5f4-36f1-4484-86ea-b12655669d5f","uris":["static-test.stackato.danielwatrous.com"],"users":null,"sso_enabled":false}
VCAP_SERVICES={}
STAGING_TIMEOUT=900.0
TCL_TEMPLOAD_NO_UNLINK=1
MAIL=/var/mail/stackato
PATH=/opt/ActivePython-2.7/bin:/opt/ActivePython-3.3/bin:/opt/ActivePerl-5.16/bin:/opt/rubies/current/bin:/opt/node-v0.10/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
STACKATO_SERVICES={}
LANG=en_US.UTF-8
SHLVL=3
LANGUAGE=en_US.UTF-8
LOGNAME=stackato
SSH_CONNECTION=172.17.42.1 41465 172.17.0.120 22
GOVERSION=1.2.1
BUILDPACK_CACHE=/var/vcap/packages/buildpack_cache
MEMORY_LIMIT=40m
```

#### detect

The detect script analyzes the application files looking for specific artifacts. For example, the Ruby buildpack looks for a Gemfile. The PHP buildpack looks for index.php or composer.json. The Java buildpack looks for a pom.xml. And so on for each technology. If a buildpack detects the required artifacts, it prints a message and exits with a system value of 0, indicating success. At this point, the buildpack files are also copied to the staging instance.

#### compile

The compile script is responsible for doing the rest of the work to prepare the runtime environment. This may include downloading and compiling source files. It may include downloading or copying pre-compiled binaries. The compile script is responsible for arranging the application files and runtime artifacts in a way that accommodates configuration.

The compile script, as well as all runtime components, are run as local user without sudo privileges. This will often mean that configuration files, log files, and other typical file system paths will need to be adapted to run as an unprivileged user.

The follow image shows the directory structure on the docker staging instance which will be useful when creating a compile script.

![alt](http://resource.docker.cn/stackato-staging-directory-structure.png)

[Hooks are available for pre-staging and post-staging](https://docs.stackato.com/user/deploy/stackatoyml.html#hooks). This may be useful if access to external resources are needed to stage and require authentication or other preliminary setup.

### Create droplet
 
Once the compile script completes, everything in the user’s directory is packaged up as a compressed tar file. This is the droplet. The droplet is copied to the cloud controller and the staging instance is destroyed.

---

Original source: [Buildpack staging process in Stackato and Helion](http://software.danielwatrous.com/buildpack-staging-process-in-stackato-and-helion/)