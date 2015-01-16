# Mono Docker language stack

---

##### Author: Michael Friis

---

Couple weeks ago, [Docker announced official pre-built Docker images for a bunch of popular programming languages](https://blog.docker.com/2014/09/docker-hub-official-repos-announcing-language-stacks/). Each stack generally consists of two Dockerfiles: a base Dockerfile that installs system dependencies required for that language to run, and an onbuild Dockerfile that uses [ONBUILD](https://docs.docker.com/reference/builder/#onbuild) instructions to transform app source code into a runnable Docker image. As an example of the latter, the [Ruby onbuild Dockerfile](https://github.com/docker-library/ruby/blob/master/2.1/onbuild/Dockerfile) runs bundle install to install libraries specified in an app’s Gemfile.

Managing system dependencies and composing apps from source code is very similar to what we do with [Stacks](https://devcenter.heroku.com/articles/stack) and [Buildpacks](https://devcenter.heroku.com/articles/buildpacks) at Heroku. To better understand the Docker approach, I created a [language stack for Mono](https://registry.hub.docker.com/u/friism/mono/), the open source implementation of Microsoft’s .NET Framework.

![alt](http://resource.docker.cn/mono-logo.png)

![alt](http://resource.docker.cn/docker-logo.png)


## How to use

**A working Docker installation** is required for this section.

To turn a .NET app into a runnable Docker image, first add a Dockerfile to your app source root. The sample below assumes a simple console app with an output executable name of TestingConsoleApp.exe:

```
FROM friism/mono:3.10.0-onbuild
CMD [ "mono", "./TestingConsoleApp.exe" ]
```

Now build the image:

```
docker build -t my-app .
```

The friism/mono images are [available in the public Docker Registry](https://registry.hub.docker.com/u/friism/mono/) and your Docker client will fetch them from there. Docker will then execute the [onbuild instructions](https://github.com/friism/docker-mono/blob/master/onbuild/Dockerfile) to restore NuGet packages required by the app and use xbuild (the Mono equivalent of msbuild) to compile source code into executables and libraries.

The Docker image with your app is now ready to run:

```
docker run my-app
```

If you don’t have an app to test with, you can experiment with this [console test app](https://github.com/friism/TestingConsoleApp).

## Notes

The way Docker languages stacks are split into a base image (that declares system dependencies) and an onbuild Dockerfile (that composes the actual app to be run) is perfect. It allows each language to get just the system libraries and dependencies needed. In contrast, Heroku has only one [stack image](https://devcenter.heroku.com/articles/stack) (in several versions, reflecting underlying Linux distribution versions) that all language buildpacks share. That stack is at once both too thick and too thin: It includes a broad assortment of libraries to make supported languages work, but most buildpack maintainers still have to hand-build dependencies and vendor in the binaries when apps are built.

Docker has no notion of a cache for `ONBUILD` commands whereas the [Heroku buildpack API has a cache interface](https://devcenter.heroku.com/articles/buildpack-api#caching). No caching makes the Docker stack maintainer’s life easier, but it also makes builds much slower than what’s possible on Heroku. For example, Heroku buildpacks can cache the result of running `bundle install` (in the case of Ruby) or `nuget restore` (for Mono), greatly speeding up builds after the first one.

Versioning is another interesting difference. Heroku buildpacks bake support for all supported language versions into a single monolithic release. What language version to use is generally specified by the app being built in a `requirements.txt` (or similar) file and the buildpack uses that to install the correct packages.

Docker language stacks, on the other hand, support versioning with version tags. The app chooses what stack version to use with the FROM instruction in the Dockerfile that’s added to the app. Stack versions map to versions of the underlying language or framework (eg. `FROM python:3-onbuild` gets you Python 3). This approach lets the Python stack, for example, compile Python 2 and 3 apps in different ways without having a bunch of branching logic in the `onbuild Dockerfile`. On the other hand, pushing an update to all Python stack versions becomes more work because the tags have to be updated individually. There are tradeoffs in both the Docker and Heroku buildpack approaches, I don’t know which is best.

Docker maintains a [free, automated build service](http://docs.docker.com/docker-hub/builds/) that churns out hosted Docker images for everyone to use. For my Mono stack, Docker Hub pulls updates from the [GitHub repo with the Dockerfiles](https://github.com/friism/docker-mono) and [builds the relevant tags into images](https://registry.hub.docker.com/u/friism/mono/). This is very convenient for stack maintainers. Heroku has no hosted service for building buildpack binaries, although I have [documented a (Docker-based) approach](http://friism.com/building-heroku-buildpack-binaries-with-docker) to scripting this work.

(Note that, while Heroku buildpacks are wildly successful, it’s an older standard that predates Docker by many years. If it seems like Docker has gotten more things right, it’s probably because that project was informed by Heroku’s experience and by the passage of time).

Finally, and unrelated to Docker and Heroku, the [Mono Project now has an APT package repository](http://download.mono-project.com/repo). This is pretty awesome, and I sincerely hope that the days of having to compile Mono from source are behind us. I don’t know if the repo is quite stable yet (I had to download a key without using SSL, the `mono-devel` package is versioned `3.10.0-0xamarin1` and the package fails to declare a dependency on udev), but it made the Mono Docker stack image a lot simpler. Check out the [diff going from 3.8.0 (compiled from source) to 3.10.0 (installed from APT repo)](https://github.com/friism/docker-mono/commit/4e476d6ed350bc22be869fbdcd7df3e109fa6c6e).

---

Original source: [Mono Docker language stack](http://friism.com/mono-docker-language-stack)