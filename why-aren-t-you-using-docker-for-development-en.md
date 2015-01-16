# Why aren't you using Docker (for development)?

---

##### Author: Joe Hirn

---

Virtualizing, it's fantastic!

A little while back I wrote about [the merits of using a virtual machine](https://www.devmynd.com/blog/2014-2-why-aren-t-you-using-vagrant) for local development dependencies. As consultants, we jump around projects pretty frequently. We sometimes swap people on projects. We're sometimes forced to dig up ancient code bases to investigate bugs or new features. And by old, I mean like 6-9 months old. You know, prehistoric!!!

Locally we have Bundler/Leiningen/SBT/Maven/etc... for configuring and running our application under the context of its own dependency versions, but what about when you go back to that Elasticsearch 0.9 project (which has breaking changes to 1.0)? You can’t easily run two different versions of Elasticsearch on your machine, nor do you want to. It's good to keep the versions of services you're developing against locked down in the same way we do with our libraries. We need something to manage this, but I want it to be easy because I am lazy.

I set out a little while ago to build something in house combining Vagrant and Chef to provision a local virtual machine. It was coined Soup Kitchen by the best namer in the business, [Mr. Dayton Nolan](http://daytonnolan.com/). Where vagrants and chefs meet. Despite my best efforts to roll everything into customized roles per project, it still required a basic understanding of Chef. Chef's awesome, but ultimately overkill for configuring dev environments. Additionally, Soup Kitchen was best managed as something outside of your project due to the amount of code the wrapper cookbooks and roles took up, even if you weren't enabling them in your Vagrantfile. Now you have two repos. Yo, yo dawg ensues.

In short, it didn't take off.

## First attempt at Docker

I really wanted to do something with Docker, but I went in and spent time learning it and came out a little dizzy. The tooling around this was difficult due to the number of arguments you had to supply to the CLI for multiple containers. I still have to sell this to members on my team as something easy. That's where Soup Kitchen failed.

I'm also not the base use case for Docker. I'm not containerizing the app we're building and most of the Docker reads out there seem to be based around that. Perhaps I crack that nut at some point and write a post about how to run your app entirely from a container, but I feel configuring your local environment for running the app itself will ultimately give you direct access to all of your development tools, especially those around debugging.

So you still have to configure your own machine for development, but that's fine. Leiningen or Bundler do well enough to isolate execution on the local machine. CI should rule out any "works on my machine" discrepancies that arise by not having a 100% homogeneous development environment. Plus you can still keep your machine personalized, which I think is important to developer happiness (and productivity).

In short, I couldn't figure out how to make Docker as simple as `vagrant up`, and kinda gave up for a little while.

## Outside influence

A few weeks back at our Thursday code review the fantastic [Laura Frank](https://twitter.com/rhein_wein) came in to do her [RubyConf talk about Containerized Ruby Applications with Docker](http://rubyconf.org/program#prop_674). If you're attending the conference, you should definitely check out what she has to say about it. In her talk, she introduced us to [Panamax](http://panamax.io/) which she is a core committer on. She also mentioned [Fig](http://www.fig.sh/) because it is also good and because she's also a fan of the fig fruit.

This talk inspired me to go have a look at these two tools as they seemed like the missing link for my previous goals of having one command configure all of our dev dependencies. That used to be `vagrant up` and `foreman start` before that, but my world was soon to change.

## A demo rails app using fig

I [created this rails app](https://github.com/devmynd/fig_rails_demo) as a POC to see how much effort would be involved getting three services (Postgres, Elasticsearch, Redis) running and having our rails app talk to them. After about two hours (mainly due to side track reading), I had Postgres and Elasticsearch containers running. Once I knew what I was doing I showed [Chris Sprehe](https://www.devmynd.com/culture/team/chris-sprehe) and he asked how much it would take to add Redis. Two minutes later, it was done.

Holy crap this is easy. And the entirety of the fig.yml:

```
db:
  image: postgres:9.4
  ports:
    - 5432:5432

elasticsearch:
  build: docker/elasticsearch/
  ports:
    - 9200:9200
    - 9300:9300

redis:
  image: redis:2.8.17
  ports:
    - 6379:6379
```

This is far smaller than the vagrant file used for Soup Kitchen, not to mention easier to parse with your brain. To be honest, I can't see how much more difficult it will become even if you have more dependencies.

The main thing I like about this is that it can be checked right into your project. A docker directory and single fig.yml isn't too obtrusive for the amount of setup you gain.

The README for the project has more info on how to run the project and its goals which I've covered at length here. Thanks to Chris for giving it a run through and touching up the README so it sucks just a little bit less. If you want more info please enter an issue or PR.

## No hate....

It should be mentioned I don't think poorly of Chef or Vagrant in any way. Both solve entire classes of problems and I think the correct way to manage large environments is somewhere between leveraging all three. But 90% of our projects deploy to Heroku so we just don't need the DevOps overhead on many of our projects. But for my needs of configuring a dev environment and being able to work on multiple projects easily, this fig setup is a no brainer.

So since it's that easy, I revise my original question: why aren't you using docker?

---

Original source: [Why aren't you using Docker (for development)?](https://devmynd.com/blog/2014-10-why-aren-t-you-using-docker-for-development)