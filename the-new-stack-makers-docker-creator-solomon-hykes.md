# The New Stack Makers: Docker Creator Solomon Hykes

---

##### Author: Luke Lefler

![alt](http://resource.docker.cn/solomon-newstack-maker.jpg)

Much has been discussed [here](http://thenewstack.io/?s=docker) and [elsewhere](http://www.wired.com/2013/09/docker/all/) about Docker, the tremendously popular (and justifiably so) open-source project that automates the deployment of applications inside software containers.

At New Relic’s [FutureStack14 Conference](http://futurestack.io/), Solomon Hykes, Docker project progenitor and CTO of [Docker, Inc.](http://www.docker.com/), delivered this keynote address, in which he describes the context within which Docker arose and grows, the reasons for its creation and advancement, and the greater need to build on its momentum in order to help developers as a whole:

Summarizing the key conditions that brought about Docker, Solomon observes that the distributed applications of today “are not distinguishable from the Internet, really.” They’re expected to always be available and always work, and to be accessible from anywhere at any time, no matter how many different users and types of devices are using them. Applications must be elastic, massively scale-able and completely interoperable. End users have come to expect no less. An ever-expanding population of developers not only must meet the current demand but also build the next generations of these complex and often widely-adopted applications.

This is hard work, especially in an environment where old platforms no longer meet these challenges. It’s become less of an option, when a developer hits a snag, to simply phone an iOS friend or an Android friend.

The tooling solution must address the biggest problem facing developers: the application is not running on a single computer, but it has to seem that way in order for the developer to deconstruct and reason it. This solution “requires independence between the logical software component and the underlying infrastructure, so that when machines go down, or are upgraded or deployed somewhere else, somehow the service is always there, independent of one machine.”

Large tech shops such as Amazon, Apple, Google and Microsoft have invested considerable time and resources in their own, in-house stacks, and this has left everyone else to cobble together open source tools and persevere.

“The general feeling,” Solomon laments on behalf of all developers, “is that the whole thing lacks a cohesive answer. Developers can’t point to something and say, ‘That’s what I’m building on; that’s my platform.’ That’s really the urge that, collectively, we’ve felt for some time: to come together and just build it, just define what the hell this platform is and just make it meet the requirements, and then we can go on to the interesting stuff…”

The answer actually turns out to be a series of answers to separate problems, each of which has to be dealt with separately. Before launching Docker, Solomon and his team compiled a list of these problems and began to work through them.

The first problem they attacked was the challenge of packaging and distribution; without consistent packaging, instability results when a singular software component meets a divergent array of operating systems, devices and data centers. They sought to define a standardized format using existing technologies: a “consistent box” – the mobile and payload-ready Docker container. The second problem was sandbox runtime; “how do you execute those payloads on the different machines so that the resulting behavior is consistent and predictable?” These two issues defined the better part of Docker’s year zero and led to stabilizing the project.

The remaining items on the list of problems promise an exciting (and busy) future. Docker is focused on the difficulty of networking, and is working on simple tools for developers to sort out the wiring between an application’s multiple components and the services with which it needs to communicate.

“You’ve got multiple machines but you want to interact with them as one,” is how Solomon defines the issue of clustering. “You’ve got all sorts of technologies out there, and we’re trying to combine the capabilities and expose them through a set of interfaces that actually make them simple to use.” A nearly identical issue is composition, the problem of having multiple logical components that need to be reasoned, versioned and tested as a whole.

“When you’re deploying complex distributed applications across complex infrastructures, very quickly the problem arises, especially in the production environment, where security and reliability and control over the trail of changes is important, it matters to know exactly what’s running, who built it, from what pieces, who provided those pieces, and so on, all the way back to source code and organizations and people that you know and trust,” Solomon explains the critical challenge to have cryptographically-enforced proof of identity for each part of one’s deployment. This goes hand-in-hand with the problem of authorization, “a layer that connects to your identity infrastructure” to facilitate permission management of each component of the application. Authorization tends to be a hot topic in IT departments, who take most of the heat when there’s a failure of authorization control.

The IRC channel for Docker is humming with activity, joined by engineers from dozens of different companies, some of whom are contributing to an open source project for the first time ever, and who are “basically duking it out all day” to improve a project that looks promising for the betterment of all.

To create this snowball effect of attracting more and more talent and resources to the project, it’s necessary to give the impression that “it’s going to happen, with or without you.”

However, if every specification and variation that arises is adopted, the project won’t scale. “You want loosely-coupled tools,” Solomon insists. “Individual tools that solve individual problems in the simplest possible way” are combined to produce new, aggregate solutions in a process called “scale by composition.” This composition must be enabled by standard interfaces, so that future contributors can expose extensions to the toolkit “using an interface that’s agreed upon in advance, and it’s agreed upon by everybody.”

“All we’re doing is taking the principles of Unix design, that made the Unix system possible in the first place, and applying those principles for problems that we have now in the 21st Century, thirty years later,” and all ideas are welcome, says Solomon.

“Building a system that is jointly owned by everyone producing software, and somehow getting enough people to adopt it, and make it work well enough, and scale well enough, that everyone can rely on it (and) start taking it for granted, and each go on building our own amazing stuff — that’s the challenge.”

After his presentation Solomon recollects with his host, New Relic Founder Lew Cirne, that while it took a few behind-the-scenes machinations to birth the Docker project, “eventually it just kind of took a life of its own, and it forced our hand.”

“There was no big decision to make,” Solomon remembers, “until Docker blew up and the community appeared.” And its message was clear: “You will now do our bidding, and be the stewards of this community.”

---

Original source: [The New Stack Makers: Docker Creator Solomon Hykes](http://thenewstack.io/the-new-stack-makers-docker-creator-solomon-hykes/)
