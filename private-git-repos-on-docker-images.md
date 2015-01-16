# Leverage Bundler to Get Private Git Repos on Docker Images

---

##### Author: Simon Robson

----

I’m not sure why it took me so long to come up with a workable approach for this. But I guess I can simply blame thinking in outmoded ways. For some time, I’ve been struggling on-and-off with dockerizing one of our (closed-source) applications. The application in question is a rails application - and like most such beasts it has a slew of dependencies managed via Bundler. The sticking point was that a couple of those dependencies were in private repos (on GitHub, though, as it turns out, that isn’t too relevant).

My old way of thinking - influenced by a few years of Chef and, more recently, Ansible experience, was to use the Dockerfile to set up my image with the application code, then run `bundle install`, on the image, to bring in the gem dependencies The approach was something [like this](http://danielmartins.ninja/posts/a-week-of-docker.html) - taking care of [make use of Docker’s cacheing](http://ilikestuffblog.com/2014/01/06/how-to-skip-bundle-install-when-deploying-a-rails-app-to-docker/) to avoid pulling the dependencies over the wire on each build.

But this didn’t work for private repos - as, obviously enough, there was no access to those repos from the image context (ssh agent forwarding is not a possibility here).

The various solutions to the general problem [as offered by GitHub](https://developer.github.com/guides/managing-deploy-keys/) didn’t really appeal. Oauth tokens, deploy keys and machine users all involved storing secrets in the image - and also some amount of adminsitrative overhead. I was keen to avoid all of that, but remained stumped until yestedray.

In the end, my feeling is that building Docker images is easiest when as many of your code assets as possible are sitting right there on the build machine. And all the Dockerfile needs to do is to `ADD` them to the image.

It turns out that with some juggling, Bundler can help achieve this aim. The key is to run `bundle package --all` locally. This causes bundler to download all gems to vendor/cache within your project (and the –all flag extends this from publically available gems to private ones too. It will be the default in Bundler 2.0). Thereafter, `bundle install --deployment` from within the image will install gems directly from the cache - without hitting the network.

The final step was to combine this with the workaround to capitalize on Docker’s cacheing. Again there was a bit of juggling involved. But the end result works well.

*(Note: this approach depends on [.dockerignore](http://docs.docker.com/reference/builder/#the-dockerignore-file), so you’ll need to be running Docker 1.1.0 or higher)*

On your build/development machine:

```
bundle  package --all
```

Then in your Dockerfile, adjusted to taste:

```
FROM <whatever>

RUN mkdir /app
ADD Gemfile /app/
ADD Gemfile.lock /app/
RUN mkdir -p /var/bundle
ADD vendor/cache /app/vendor/cache
RUN cd /app && bundle install --deployment --path /var/bundle --without development test
ADD . /app
```

---

Original source: [Leverage Bundler to Get Private Git Repos on Docker Images](http://simonrobson.net/2014/10/14/private-git-repos-on-docker-images.html)