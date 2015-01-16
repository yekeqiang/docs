# Hosting Python WSGI applications using Docker

---

Author: Graham Dumpleton

---

As I mentioned in my previous blog post I see a lot of promise for Docker. The key thing that I personally see as being able to gain from Docker, as a provider of a hosting solution for Python WSGI applications, is that I can get back some control over the hosting experience that developers will have.

Right now things can quickly become a bit of a mess, because the experience that developers have of Apache/mod_wsgi is going to be dictated by how a Linux distribution or hosting provider has setup Apache, and how easy they have made customising it in order to add the ability to host Python WSGI applications and then tune the Apache server. The less than optimal experience that developers usually have means they do not get to appreciate how well Apache/mod_wsgi can work and simply desert it for other options.

In the case of Docker, I can provide a pre packaged image for hosting Python WSGI applications which uses my knowledge of how to set up Apache and mod_wsgi properly to give the best experience. I can therefore hope that Docker may help me to win back some of those who otherwise never really understood the strengths of Apache and mod_wsgi.

## Current offerings for Python and Docker

Although Docker is still young, to be frank, the bulk of the information around about running Python WSGI application with Docker is pretty woeful. The instructions provided focus more on how to use Docker itself rather than how to create a production capable hosting solution for Python WSGI applications within the container. Nearly all explanations I have found describe the use of builtin development servers for Python web frameworks such as Flask and Django. Using inbuilt development servers for production is generally a very bad idea.

In some cases they will suggest the use of gunicorn or CherryPy WSGI servers, but these themselves cannot handle hosting of static files. How exactly you are meant to host the static files they don't really provide details on, at most perhaps suggesting the use of S3 as a completely separate mechanism for hosting them.

There are available some Docker images for using uWSGI, but they are generally setup with the specific requirements of that user in mind, rather than trying to provide a good reusable image that can be applied across many uses cases, without you yourself having to do some measure of re-configuration. Again they aren't exactly transparent as far as handling static files and leave that mostly up to you to work out how to solve.

The final problem with the uWSGI Docker images is that they are effectively unsupported efforts and haven't been updated in some time. They therefore are not keeping up to date with any security fixes or general bug fixes in the packages they are using.

## Using Apache/mod_wsgi and Docker

To date I have not seen anyone attempt to describe how to use Apache and mod_wsgi with Docker. It isn't something that I am going to do exactly either, in as much as rather than describe how you yourself could create an image for using Apache and mod_wsgi with Docker, I am simply going to provide a pre packaged image instead. What I will describe therefore is how to use that image and how best to use it in its pre packaged form.

This blog post is therefore the first introduction to this endeavour. I will show you how to use the Docker image with a couple of different WSGI applications and then in subsequent blog posts I will start peeling apart the layers and explain the different parts that go into it and what capabilities it has. Provided I don't get too carried away with doing more coding, which is obviously the fun bit, I will back things up by finally starting to upgrade the mod_wsgi documentation to cover it and all the other new features that are available in mod_wsgi these days.

## Running a Hello World WSGI application

Lets start out therefore with the canonical WSGI hello world application.

```
def application(environ, start_response):
 status = '200 OK'
 output = 'Hello World!'
response_headers = [('Content-type', 'text/plain'),
 ('Content-Length', str(len(output)))]
 start_response(status, response_headers)
return [output]
```

Create a new empty directory and place this in a file called 'wsgi.py'.

This Hello World program has no associated static files, nor does it require any additional Python modules to be installed. Even though no separate modules are required at this point, we will still create a 'requirements.txt' file in the same directory. This 'requirements.txt' file will be left empty for this example.

The next step is to create a 'Dockerfile' to build up our Docker image. As we are going to use the pre packaged Docker image I am providing and it embeds various magic, all that the 'Dockerfile' needs to contain is:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild
CMD [ "wsgi.py" ]
```

For the image being derived from, an 'ENTRYPOINT' is already defined which will run up Apache/mod_wsgi. The 'CMD' instruction therefore only needs to provide any options, which at this point consists only of the path to the WSGI script file, which we had called 'wsgi.py'.

We can now build the Docker image for the Hello World example:

```
docker build -t my-python-app .
```

and then run it:

```
docker run -it --rm -p 8000:80 --name my-running-app my-python-app
```

The Hello World WSGI application will now be accessible by pointing your browser at port 8000 on the Docker host.

## Running a Django based web site

We don't run Hello World applications as our web sites, so we also need to be able to run whole Python web sites constructed using web frameworks such as Django as well. It is with more complicated web applications that we start to also have static files that need to be hosted at the same time, so we need to deal with that somehow. The Python module search path can also require special setup so that the Python interpreter can actually find where the Python code for the web application is located.

So imagine that you have a Django web application constructed using the standard layout. From the top directory of this we would therefore have something like:

```
example/
example/example/
example/example/__init__.py
example/example/settings.py
example/example/urls.py
example/example/views.py
example/example/wsgi.py
example/htdocs/
example/htdocs/admin
example/htdocs/admin/...
example/manage.py
requirements.txt
```

The 'requirements.txt' which was used to create any local virtual environment used during development would already exist, and at the minimum would contain:

```
Django
```

Within the directory would then be the actual project directory which was created using the Django admin 'startproject' command.

As this example requires static files, we setup the Django settings file to define the location of a directory to keep the static files:

```
STATIC_ROOT = os.path.join(BASE_DIR, 'htdocs')
STATIC_URL = '/static/'
```

and then run the Django admin 'collectstatic' command. The 'collectstatic' command copies all the static file assets from any Django applications into the common 'htdocs' directory. This directory will then need to be mounted at the '/static' URL when we run Apache/mod_wsgi.

What we are going to do now is create a 'Dockerfile' in the same directory as the 'requirements.txt' file. This will be the root of our application when copied across to the Docker image.

Now normally when Apache/mod_wsgi gets run with with the pre packaged image, the root directory of the application would normally be the current working directory for the application and also be added to the Python module search path. For a Django site, what we really want is for the top level 'example' directory to be the current working directory and for it to be searched for Python modules. This is necessary so that the correct directory is searched from for the Django settings file, which for this example has the module path 'example.settings'.

With the way Django lays out the project and creates the 'wsgi.py' file such that it is importable as 'example.wsgi', it can be preferable to use it as a module rather than as a WSGI script file. I'll get into the distinction another time, but importing it as a module does allow me to show off that it is possible to use a WSGI script file, a module or even a Paste style ini configuration file as the application entry point.

With all that said, we now actually create the 'Dockerfile' and in it we place:

```
FROM grahamdumpleton/mod-wsgi-docker:python-2.7-onbuild
CMD [ "--working-directory", "example", \
 "--url-alias", "/static", "example/htdocs", \
 "--application-type", "module", "example.wsgi" ]
```

The options to the 'CMD' instruction in this case serve the following purposes.

The '--working-directory' option says that the 'example' directory should actually be set to be the current working directory for the WSGI application when run. That directory will also be added automatically to the Python module search path so that the 'example' package which contains all the code can be found.

The '--url-alias' option says that the static files in the 'examples/htdocs' directory should be mounted at the '/static' URL as was specified by the 'STATIC_URL' setting in the Django settings module.

The '--application-type' option says that rather than the WSGI application entry point being specified as a WSGI script file, it is defined by the listed module path. The default for this would have been 'script', with another possible value being 'paste' for a Paste style ini configuration file.

Finally, the 'example.wsgi' option is the Python module path for the 'wsgi.py' sub module in the project 'example' package.

As before we build the Docker image and then run it.

In a real Django site we would normally also have a database and possibly a key/value cache of some sort. Setting these up is beyond the scope of this post but would follow normal Docker practices. That or you might use a tool such as Fig to manage the linking and spin up of all the containers.

## Setting up the Apache configuration file

In short there isn't one and you do not have to concern yourself with it.

This is a key point with being able to supply a pre packaged image for hosting using Apache/mod_wsgi. Since users often don't want to learn properly how to set up Apache and as such it causes so much grief, I can completely remove the need for a developer to have to worry about it.

Instead I can provide a simplified set of command line options which implement the basic features that most sites would want to use when setting up Apache/mod_wsgi. The scripts under pinning the pre packaged Docker image can then dynamically generate the Apache configuration on the fly based on the specific options provided.

In doing this, this is where I can apply my knowledge of how to set up Apache/mod_wsgi and ensure things are down correctly, securely and in a way that would give a good level of performance out of the box.

This doesn't mean you can't avoid needing to tune the settings to get Apache/mod_wsgi to run for your specific site, but the number of knobs you would have to worry about is greatly reduced as everything else would be handled automatically.

## But how does this all actually work?

So this is an initial introduction to just one of a number of new things I have been working on related to mod_wsgi. As I start to peel back some of the layers to explain how all this works I will start to introduce some of the other things I have been cooking up over the last year, including alluding to other things that hopefully I will get to down the track.

If you can't wait and want to play around with these docker images they can be found, along with some basic setup information, on the [Docker hub](https://registry.hub.docker.com/u/grahamdumpleton/mod-wsgi-docker/).

If you need help in working out how to use them or have any other questions, then rather than try and post questions against this blog, go and ask your questions on the [mod_wsgi mailing list](http://code.google.com/p/modwsgi/wiki/WhereToGetHelp#Asking_Your_Questions). Please do not use StackOverflow or related sites as I don't answer questions there any more and no one there will know anything anyway since this is all so new.

---

Original source: [Hosting Python WSGI applications using Docker](http://blog.dscpl.com.au/2014/12/hosting-python-wsgi-applications-using.html)