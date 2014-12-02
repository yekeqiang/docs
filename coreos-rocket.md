# CoreOS is building a container runtime, Rocket

---

##### Author: [Alex Polvi](https://twitter.com/polvi)

---

![alt](http://resource.docker.cn/rocket-launch.png)

Rocket is a new container runtime, designed for composability, security, and speed. Today we are releasing a [prototype version on GitHub](https://github.com/coreos/rocket) to begin gathering feedback from our community and explain why we are building Rocket.

## Why we are building Rocket

When we started building CoreOS, we looked at all the various components available to us, re-using the best tools, and building the ones that did not exist. We believe strongly in the Unix philosophy: tools should be independently useful, but have clean integration points. We hope this is reflected in tools that we build, such as etcd, which have seen widespread adoption and use outside CoreOS itself.

When Docker was first introduced to us in early 2013, the idea of a “standard container” was striking and immediately attractive: a simple component, a composable unit, that could be used in a variety of systems. The Docker repository [included a manifesto](https://github.com/docker/docker/commit/0db56e6c519b19ec16c6fbd12e3cee7dfa6018c5) of what a standard container should be. This was a rally cry to the industry, and we quickly followed. Brandon Philips, co-founder/CTO of CoreOS, became a top Docker contributor, and now serves on the Docker governance board. CoreOS is one of the most widely used platforms for Docker containers, and ships releases to the community hours after they happen upstream. We thought Docker would become a simple unit that we can all agree on.

Unfortunately, a simple re-usable component is not how things are playing out. Docker now is building tools for launching cloud servers, systems for clustering, and a wide range of functions: building images, running images, uploading, downloading, and eventually even overlay networking, all compiled into one monolithic binary running primarily as root on your server. The standard container manifesto [was removed](https://github.com/docker/docker/commit/eed00a4afd1e8e8e35f8ca640c94d9c9e9babaf7). We should stop talking about Docker containers, and start talking about the Docker Platform. It is not becoming the simple composable building block we had envisioned.

## Rocket + App Container

We still believe in the original premise of containers that Docker introduced, so we are doing something about it. While we are at it, we are cleaning up and fixing a few things that we’d like to see in a production ready container. What is important to us in the design of a container?

- **Composable**. All tools for downloading, installing, and running containers should be well integrated, but independent and composable.
- **Security**. Isolation should be pluggable, and the crypto primitives for strong trust, image auditing and application identity should exist from day one.
- **Image distribution**. Discovery of container images should be simple and facilitate a federated namespace, and distributed retrieval. This opens the possibility of alternative protocols, such as BitTorrent, and deployments to private environments without the requirement of a registry.
- **Open** . The format and runtime should be well-specified and developed by a community. We want independent implementations of tools to be able to run the same container consistently.

### Rocket 0.1.0

Rocket is a command line tool, `rkt`, for running App Containers. An “App Container” is the specification of an image format, container runtime, and a discovery mechanism. Rocket is the first implementation of an App Container, but we do not expect it to be the only one.

- App Container Image: definition of a signed/encrypted tgz that includes all the bits to run an app container.
- App Container Runtime: definition of the environment the running app container should be given.
- App Container Discovery: a federated protocol for finding and downloading an app container image.

The best way to drive a standard is to let a successful and proven implementation become the de facto one. However, when the point of the software is interoperability (as it is with containers), we think things are different.

In developing the App Container specification, we started with some thoughtfulness around the requirements up front, then spent time refining it as we worked on an implementation (Rocket). We feel these specs and implementations are pretty well thought out, but are still early enough that we need your help refining it. Please [contribute to the spec](https://github.com/coreos/rocket/blob/master/app-container/SPEC.md) by sending a pull request to start the discussion.

### App Container Image

An App Container Image (ACI) is a specification for the image format of a container. It is a simple flat tarball that is always signed and optionally encrypted. By convention, an ACI is minimal, meaning it only includes the bits absolutely required to execute the application. Since an ACI may be encrypted, distribution via systems like BitTorrent, public object storage, or mirror networks is a possibility. Think of ACI as a variant of Amazon’s AMI, but created for a container world.

[Read about and contribute to the ACI draft](https://github.com/coreos/rocket/blob/master/app-container/SPEC.md#app-container-image).

### App Container Runtime

The App Container Runtime defines what environment and facilities a container runtime should provide. This includes devices, environment variables, and privileges that a container should expect. It also includes a definition of a meta-data service interface for exposing data to the environment from outside the container.

Security primitives are very important to us, so we added an identity feature to the meta-data service. This means every instance of a running container is given a unique identity, coupled with a lightweight HSM-like service for signing. No existing VM or container environment has a concept like this, so we would welcome and appreciate community feedback on this design.

[Read about and contribute to the runtime draft](https://github.com/coreos/rocket/blob/master/app-container/SPEC.md#app-container-executor).

### App Container Discovery

App Container Discovery is a method for finding your app container image. It is inspired by golang’s [vanity URL convention for import paths](http://golang.org/cmd/go/#hdr-Remote_import_paths). This means you’ll be able to refer to containers with simple names like `coreos.com/etcd`, allowing organizations to federate their downloads without running their own registry.

## Standard Containers

We want the world to run containers. A world where your application can be packaged once, and ran in the environment you choose. We believe Rocket and a definition around the App Container is a requirement for this to work. Please contribute your thoughts to our [mailing list](https://groups.google.com/forum/#!forum/coreos-dev), our [source code](https://github.com/coreos/rocket) or join us in person at the [CoreOS meet-up](http://www.meetup.com/coreos/events/218886679/) tonight in San Francisco.

## FAQ

### What is Rocket?

Rocket is an alternative to the Docker runtime, designed for server environments with the most rigorous security and production requirements. Rocket is oriented around the App Container specification, a new set of simple and open specifications for a portable container format.

### When is Rocket available?

Rocket is [available today on GitHub](https://github.com/coreos/rocket). We are releasing a 0.1.0 prototype to gather community feedback. Please note that this is a prototype quality release, very much in the spirit of “release early, release often”. Please provide feedback via GitHub.

### Why not just fork Docker?

From a security and composability perspective, the Docker process model - where everything runs through a central daemon - is fundamentally flawed. To “fix” Docker would essentially mean a rewrite of the project, while inheriting all the baggage of the existing implementation.

### Why are you doing this now?

At CoreOS we have large, serious users running in enterprise environments. We cannot in good faith continue to support Docker’s broken security model without addressing these issues. Additionally, in the past few weeks Docker has demonstrated that it is on a path to include many facilities beyond basic container management, turning it into a complex platform. Our primary users have existing platforms that they want to integrate containers with. We need to fill the gap for companies that just want a way to securely and portably run a container.

### Will Rocket run on [Ubuntu, RHEL, CentOS, etc]?

Yes, `rkt` is a stand alone tool that will live outside of CoreOS itself and can be used on a variety of platforms. In a sense it is similar to the etcd project in that it is a tool we built because nothing like it existed.

### What is the difference between Rocket and App Container?

“App Container” [defines a specification](https://github.com/coreos/rocket/blob/master/app-container/SPEC.md) of the facilities surrounding the container. Rocket implements these facilities as a command line tool. An open specification allows other systems do their own implementation of App Container without using Rocket at all. CoreOS fully supports and embraces alternative implementations.

### Could App Container support be contributed to the Docker Platform?

Definitely. If the App Container specifications were implemented inside of Docker, the projects will be interoperable, meeting the original goal of the manifesto. CoreOS will evaluate contributing this work once App Container matures.

### Will CoreOS continue to ship Docker?

Yes. We will continue to make sure CoreOS is the best place to run Docker. We will save the details for a future post, once Rocket has developed further, but expect Docker to continue to be fully integrated with CoreOS as it is today.

---

**[Discuss on Hacker News](javascript:window.location=%22http://news.ycombinator.com/submitlink?u=%22+encodeURIComponent(document.location)+%22&t=%22+encodeURIComponent(document.title))**

---

Original source: [CoreOS is building a container runtime, Rocket](https://coreos.com/blog/rocket/)