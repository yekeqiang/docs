# INITIAL THOUGHTS ON THE ROCKET ANNOUNCEMENT

---

##### Author: [Ben Golub](https://twitter.com/golubbe)

---

When Docker launched 18 months ago, we set out on a mission to build “the button” that enables any application to instantly and consistently run on any server anywhere.

Our first task was to define a standard container format that would let any application get packaged into a lightweight container that could run on any infrastructure.

With a lot of hard work and participation across the community, Docker capabilities grew, we were able to make the same Docker container run successfully across all major infrastructures, and we built a robust ecosystem that now includes:

- over 700 contributors (95% of whom do not work for Docker, Inc.)
- over 65,000 free Dockerized languages, frameworks, and applications (services)
- support by every major DevOps tool, every major public cloud, and every major operating system
- a robust ecosystem of third-party tools built on top of Docker. There are now over 18,000 projects in GitHub with Docker in the title.
- over 175 Docker meetup groups in over 40 countries
- millions of Docker users

Along the way, we built a robust, open design and governance structure, to enable users, vendors, and contributors to help guide the direction of the project.

For the past nine months, we have articulated a vision of Docker that extends beyond a single container. While Docker continues to define a single container format, it is clear that our users and the vast majority of contributors and vendors want Docker to enable distributed applications consisting of multiple, discrete containers running across multiple hosts.

We think it would be a shame if the clean, open interfaces, anywhere portability, and robust set of ecosystem tools that exist for single Docker container applications were lost when we went to a world of multiple container, distributed applications.  As a result, we have been promoting the concept of a more comprehensive set of orchestration services that cover functionality like networking, scheduling, composition, clustering, etc. While more detail will be provided at the DockerCon conference this week in Amsterdam, a few design points are worth noting:

1. Multi-container orchestration capabilities–as with the container standard itself–should be be created through an open design process with collaboration and feedback from a community and ecosystem.
2. These orchestration functions should be delivered as open APIs, developed in the open using the open design process
3. These capabilities should not be monolithic. Individuals should be free to use, modify, or not use these services and their higher level APIs
4. These capabilities and APIs should support plug-ins, so that people can choose the scheduling, clustering, logging, or other services that work best for them, without sacrificing portability, the ability to work across infrastructures, or the ability to leverage the 65K+ Dockerized apps or 18K+ tools that work with Docker. This plug-in model has worked exceptionally well for execution engines (e.g., libcontainer, LXC) and file systems (BTRFS, device mapper, AUFS, XFS). Expect to see more in our announcements this week.

Of course, different people have different views of how open source projects should develop. As noted above, the overwhelming majority of users, the vast majority of contributors, and the vast majority of ecosystem vendors want the project to support standard, multi-Docker container distributed applications. Many vendors, large and small, both welcome and are contributing to this effort. (For more on open governance in Docker, please see this [post](http://blog.docker.com/2014/11/docker-governance-advisory-board-output-of-first-meeting).)

We are committed to the ecosystem of users, vendors, and contributors. Whether people add value in the form of contributions to Docker, as independent projects that build upon the Docker container format, as plug-ins to the Docker orchestration APIs, or otherwise, we hope that the open, layered approach provides options for all. Nonetheless a small number of vendors disagree with this direction. Some have expressed their concern that, as Docker expands its scope, there may be less room for them to create differentiated, value-added offerings. In some cases, these vendors want to create orchestration solutions that are tailored for their particular infrastructure or offerings, and do not welcome the notion of portability. In some cases, of course, there are technical or philosophical differences, which appears to be the case with the recent announcement regarding Rocket. We hope to address some of the technical arguments posed by the Rocket project in a subsequent post.

For now, we want to emphasize that this is all part of a healthy, open source process. As Docker is open source, and Apache-based, people are free to use, modify, or adapt Docker for their own purposes. They are free to use Docker as a single container format. They are free to build higher level services that plug in to Docker. And, of course, they are free to promote the notion of an alternative standard, as the folks behind Rocket have chosen to do.


While we disagree with some of the arguments and questionable rhetoric and timing of the Rocket announcement, we hope that we can all continue to be guided by what is best for users and developers.

---

Original source: [INITIAL THOUGHTS ON THE ROCKET ANNOUNCEMENT](http://blog.docker.com/2014/12/initial-thoughts-on-the-rocket-announcement/)
