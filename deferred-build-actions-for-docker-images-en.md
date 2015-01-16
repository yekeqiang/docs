# Deferred build actions for Docker images

---

Author: Graham Dumpleton

---

In my [last blog post](http://blog.dscpl.com.au/2014/12/hosting-python-wsgi-applications-using.html) I introduced what I have been doing on creating a production quality Docker image for hosting Python WSGI applications using Apache/mod_wsgi.

In that post I gave an example of the Dockerfile you would use for a simple WSGI hello world application:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild

CMD [ "wsgi.py" ]
```

I also presented a more complicated example for a Django site. The Dockerfile for that was still only:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild

CMD [ "--working-directory", "example", \
  "--url-alias", "/static", "example/htdocs", \
  "--application-type", "module", "example.wsgi" ]
```

In the case of the Django site there are actually going to be quite a number of files that are required, including the Python code and static file assets. There was also a list of Python packages needing to be installed defined in a pip 'requirements.txt' file.

The big question therefore is how with only that small Dockerfile were all those required files getting copied across and how were all the Python packages getting installed?

## Using an ONBUILD Docker image

The name of the Docker image which the local Dockerfile derived from was in this case called:

```
grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild\
```

The clue as to what is going on here is the 'onbuild' qualifier in the name of the image.

Looking at the Dockerfile used to build that 'onbuild' image we find:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7
WORKDIR /app
ONBUILD COPY . /app
ONBUILD RUN mod_wsgi-docker-build
EXPOSE 80
ENTRYPOINT [ "mod_wsgi-docker-start" ]
```

If you were expecting to see here a whole lot of instructions for creating a Docker image consisting of Python, Apache and mod_wsgi you will be sadly disappointed. This is because all that more complicated stuff is actually contained in a further base image called:

```
grahamdumpleton/mod-wsgi-docker:python-2.7
```

So what has been done is to create a pair of Docker images. An underlying base image is what provides all the different software packages we need. The derived 'onbuild' image is what defines how a specific users application is combined with the base image to form the final deployable application image.

This type of split is a common pattern one sees for Docker images which are trying to provide a base level image for deploying applications written in a specific programming language.

Using a derived image to define how the application is combined with the base image means that if someone doesn't agree with the specific way that is being done, they can ignore the 'onbuild' image and simply derive direct from the base image and define things their own way.

## Deferring execution using ONBUILD

When creating a Dockerfile, you will often use an instruction such as:

```
COPY . /app
```

What this does is that at the point that the Docker image is being built, it will copy the contents of the directory the Dockerfile is contained in into the Docker image. In this case everything will be copied into the '/app' directory of the final Docker image.

If you are doing everything in one Dockerfile this is fine. In this case though I want to include that sort of boiler plate that is always required to be run in a base image, but for the instruction to only be executed when a user is creating their own derived image with their specific application code.

To achieve this I used the 'ONBUILD' instruction, prefixing the original 'COPY' instruction.

The affect of using 'ONBUILD' is that the 'COPY' instruction will not actually be run. All that will happen is that the 'COPY' instruction will be recorded. Any instruction prefixed with 'ONBUILD' will then only be replayed when a further derived image is being created which derives from this image.

Specifically, any such instructions will be run as the first steps after the 'FROM' instruction in a derived image, with:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7
WORKDIR /app
ONBUILD COPY . /app
ONBUILD RUN mod_wsgi-docker-build
EXPOSE 80
ENTRYPOINT [ "mod_wsgi-docker-start" ]
```

and:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild
CMD [ "wsgi.py" ]
```

being the same as if we had written:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7
WORKDIR /app
EXPOSE 80
ENTRYPOINT [ "mod_wsgi-docker-start" ]
```

and:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild
COPY . /app
RUN mod_wsgi-docker-build
CMD [ "wsgi.py" ]
```

You can determine whether a specific image you want to use is using 'ONBUILD' instructions by inspecting the image:

```
$ docker inspect grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild
```

Look in the output for the 'OnBuild' section and you should find all the 'ONBUILD' instructions listed.

```
"OnBuild": [
 "COPY . /app",
 "RUN mod_wsgi-docker-build"
 ],
```

Anyway, as noted before, the point here is to make what the final user does as simple as possible and avoid the need to always include specific boilerplate, thus the reason for using 'ONBUILD'.

## Installing extra system packages

For this specific Docker image, the deferred steps triggered by the 'ONBUILD' instructions will perform a number of tasks.

The first and most obvious action is that performed by the instruction:

```
COPY . /app
```

As explained before, this will copy the contents of the application directory into the Docker image.

The next step which is performed is:

```
RUN mod_wsgi-docker-build
```

This will execute the program 'mod_wsgi-docker-build'. This magic program is what is going to do all the hard work. The actual program was originally included into the image by way of the base image that the 'onbuild' image derived from. That is, in:

```
grahamdumpleton/mod-wsgi-docker:python-2.7
```

The key thing that the 'mod_wsgi-docker-build' script will do is run 'pip' to install any Python packages which are listed in the 'requirements.txt' file. Before 'pip' is run though there are a few other things we also want to do.

The first thing that the 'mod_wsgi-docker-build' script does is:

```
if [ -x .docker/action_hooks/pre-build ]; then
  echo " -----> Running .docker/action_hooks/pre-build"
  .docker/action_hooks/pre-build
fi
```

This executes an optional 'pre-build' hook script which can be supplied by the user and allows a user to perform additional steps prior to any Python packages being installed using 'pip'. Such a hook script would be placed into the '.docker/action_hooks' directory.

The main purpose of the 'pre-build' hook script would be to install additional system packages that may be required to be present when installing the Python packages. This would be necessary due to the fact that the base image is a minimal image that only installs the minimum system packages required for Python and Apache to run. It does not try and anticipate every single common package that may be required to cover the majority of use cases.

If one did try and guess what additional packages people might need, then the size of the base image would end up being significantly larger in size. As such, installation of such additional system packages is up to the user based on their specific requirements. Being Docker though, there are no restrictions on installing additional system packages. This is in contrast to your typical PaaS providers, where you are limited to the packages they decided you might need.

As an example, if the Python web application to be run needed to talk to a Postgres database using the Python 'psycopg2' module, then the Postgres client libraries will need to be first installed. To ensure that they are installed a 'pre-build' script containing the following would be used.

```
#!/usr/bin/env bash

set -eo pipefail

apt-get update
apt-get install -y libpq-dev

rm -r /var/lib/apt/lists/* 
```

The line:

```
set -eo pipefail
```

in this script is important as it ensures that the shell script will exit immediately on any error. The 'mod_wsgi-docker-build' script does the same, with the reason being that you want the build of the image to fail on any error, rather than errors being silently ignored and an incomplete image created.

The line:

```
rm -r /var/lib/apt/lists/*
```

is an attempt to clean up any temporary files so that the image isn't bloated with files that aren't required at runtime.

Do note that the script itself must also be executable else it will be ignored. It would be nice to not have this requirement and for the script to be made executable automatically if it wasn't but doing so seems to trip [a bug in Docker](https://github.com/docker/docker/issues/9547).

## Installing required Python packages

The next thing that the 'mod_wsgi-docker-build' script does is:

```
if [ ! -f requirements.txt ]; then
  if [ -f setup.py ]; then
    echo "-e ." > requirements.txt
  fi
fi
```

In this case we are actually checking to see whether there even is a 'requirements.txt' file. Not having one is okay and we simply wouldn't install any Python packages.

There is though a special case we check for if there is no 'requirements.txt' file. That is if the directory for the application contains a 'setup.py' file. If it does, then we assume that the application directory is itself a Python package that needs to first be installed, or at least, that the 'setup.py' has defined a set of dependencies that need to be installed.

Now normally when you have a 'setup.py' file you would run:

```
python setup.py install
```

We can though achieve the same result by listing the currently directory as '.' in the 'requirements.txt' file, preceded by the special '-e' option.

This approach of using a 'setup.py' as a means of installing a set of Python packages was an approach often used before 'pip' became available and the preferred option. It is still actually the only option that some PaaS providers allow for installing packages, with them not supporting the use of a 'requirements.txt' file and automatic installation of packages using 'pip'.

Interestingly, this ability to use a 'setup.py' file and installing the application as a package is something that the official Docker images for Python can't do at this time. This is because they use:

```
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
ONBUILD COPY requirements.txt /usr/src/app/
ONBUILD RUN pip install -r requirements.txt
ONBUILD COPY . /usr/src/app
```

The problem they have is that they only copy the 'requirements.txt' file into the image before running 'pip' and because the current working directory when running 'pip' only contains that 'requirements.txt' file, then the 'requirements.txt' file cannot refer to anything contained in the application directory.

So the order in which they do things prohibits it from working. I am not sure I fathom why they have chosen to do it the way they have and whether they have a specific reason. It is perhaps something they should change, but I don't rule out they may have a valid reason for doing that so haven't reported it as an issue at this point.

After seeing if what packages to install may be governed by a 'setup.py' file the next steps run are:

```
if [ -f requirements.txt ]; then
  if (grep -Fiq "git+" requirements.txt); then
    echo " -----> Installing git"
    apt-get update && \
      apt-get install -y git --no-install-recommends && \
      rm -r /var/lib/apt/lists/*
  fi
fi

if [ -f requirements.txt ]; then
  if (grep -Fiq "hg+" requirements.txt); then
    echo " -----> Installing mercurial"
    pip install -U mercurial
  fi
fi
```

These steps are required to determine whether the 'requirements.txt' file lists packages to be installed which are actually managed under a version control system such as git or mercurial. If there is, then since the base image provides only a minimal set of packages, then it will be necessary to install git or mercurial as necessary.

Finally we can now run 'pip' to install any packages listed in the 'requirements.txt' file:

```
if [ -f requirements.txt ]; then
  echo " -----> Installing dependencies with pip"
  pip install -r requirements.txt -U --allow-all-external \
    --exists-action=w --src=.docker/tmp
fi
```

## Performing final application setup

All the system packages and Python packages are now installed. This is along with all the Python application code also having been copied into the image. There is one more thing to do though. That is to provide the ability for a user to provide a 'build' hook of their own to perform any final application setup.

```
if [ -x .docker/action_hooks/build ]; then
  echo " -----> Running .docker/action_hooks/build"
  .docker/action_hooks/build
fi
```

As with the 'pre-build' script, the 'build' script needs to be placed into the '.docker/action_hooks' directory and needs to be executable.

The purpose of this script is to allow any steps to be carried out that require all the system packages and Python packages to have first been installed.

An example of such a script would be if running Django and you wanted to have the building of the Docker image collect together all the static file assets, rather than having to do it prior to attempting to build the image.

```
#!/usr/bin/env bash

set -eo pipefail

python example/manage.py collectstatic --noinput
```

Some PaaS providers in their build systems will try and automatically detect if Django is being used and run this step for you without you knowing. Although I have no problem with using special magic, I do have an issue where the system would actually be having to guess if it was your intent that such a step be run. I therefore believe in this case it is better that it be explicit and that the user define the step themselves. This avoids any unexpected problems and eliminates the need to provide special options to disable such magic steps when they aren't desired and go wrong.

## Starting the Python web application

The above steps complete the build process for the Docker image. The next phase would be to deploy the Docker image and get it running. This is where the final part of the 'onbuild' Dockerfile comes into play:

```
EXPOSE 80
ENTRYPOINT [ "mod_wsgi-docker-start" ]
```

I will explain what happens with that in my next post on this topic.

---

Original source: [Deferred build actions for Docker images](http://blog.dscpl.com.au/2014/12/deferred-build-actions-for-docker-images.html)