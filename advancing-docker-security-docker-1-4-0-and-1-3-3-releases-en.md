# ADVANCING DOCKER SECURITY: DOCKER 1.4.0 AND 1.3.3 RELEASES

---


We’re pleased to announce that we have today released Docker Engine 1.4. What’s in it? As Solomon Hykes described last week at DockerCon Europe, its major “feature” is an emphasis on bug fixes and platform stability. Over 180 commits for fixes were merged! Docker 1.4 also adds the [Overlay Filesystem](https://github.com/docker/docker/pull/7619) as a new, experimental, storage driver. (see [release notes](https://docs.docker.com/release-notes/) and [bump branch](https://github.com/docker/docker/pull/9345)).

Today, we also release Docker Engine 1.3.3 fixing three vulnerabilities ([see advisory](https://groups.google.com/d/msg/docker-user/nFAz-B-n4Bw/0wr3wvLsnUwJ)). These fixes are also in version 1.4.0. Here is a bit more info on the fixes:

- On November 24, 2014, we released Docker 1.3.2 to remedy two critical issues which could be exploited by a malicious image to break out of Docker container. Please see our security advisory for more details.

- After releasing 1.3.2, we discovered additional vulnerabilities that could be exploited by a malicious Dockerfile, image, or registry to compromise a Docker host, or spoof official images.

- The remediations have been released today in both Docker Engine 1.3.3 and Docker Engine 1.4.  All users should plan to upgrade to Docker Engine 1.3.3 or higher.  Please see the docs for upgrade instructions.

- **Please note that these vulnerabilities only affect users downloading or running malicious images, or building from malicious Dockerfiles. Users may protect themselves from malicious content by only downloading, building, or running images from trusted sources**.  In addition, we recommend that you:

	- Run Docker Engine with AppArmor or SELinux to provide additional containment.

	- Map groups of mutually-trusted containers to separate machines and VMs.

## Advancing Docker Security

Following on our [security advisory](https://groups.google.com/d/msg/docker-user/nFAz-B-n4Bw/0wr3wvLsnUwJ) and subsequent Docker Engine 1.3.3 release, I would like to share some of our thoughts and plans regarding security. Security is of paramount importance to Docker which is reflected in our

1. Efforts to rapidly address vulnerabilities when they are identified
Enhancements to the security of the platform, our users, and their applications through our roadmap
2. Collaboration with our contributors and ecosystem partners to define a set of best practices for Docker security.

Specifically, our efforts have focused on the following:

1. ***Product & Ecosystem*** – Docker Engine takes advantage of the security mechanisms and isolation provided by the OS. This is pluggable, with support on Linux for namespaces, capabilities, and cgroups implemented through either libcontainer or lxc. In the future, we expect new execution engine plugins to offer more choice and greater granularity for our security-focused users. These mechanisms are part of what define a container and running in a container is safer than running without. On systems where supported, Docker has incorporated SELinux and AppArmor integration. We have added signed Docker images in our Docker Hub Official Repos starting in [release 1.3](http://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/). This is the first step towards a more robust chain of trust that allows users to have confidence in the origin of their images. However, do note that untrustworthy sources may still create signed images and it will be up to users to trust, or not trust, the developers of those images. **You can read more about our [Trust System proposal](https://github.com/docker/docker/pull/9036) here and you are encouraged to add feedback**.  We continue to receive great input from all around the community on ideas for security features, and as these come together we’ll be sure to share the proposals and roadmaps here – stay tuned for more!

2. ***Security auditing, reporting, and response*** – we perform our own security testing as well as engaging a private security firm to audit and perform penetration testing. Issues are also received by our active user and developer community. All issues found or reported are promptly triaged, with critical issues initiating an immediate response. Our goal is to have security fixes for the current stable release in the hands of our users absolutely as quickly as possible. Fixes, once prepared, are initially sent to an early disclosure notification list for review and for vendor preparedness in advance of public disclosure. This list includes Linux distributions and cloud providers. We continue to develop and update our [practices](http://www.docker.com/resources/security) as we learn.

3. ***Disclosure & Transparency*** – we practice [responsible disclosure](http://en.wikipedia.org/wiki/Responsible_disclosure). Without compromising users, we disclose and provide updates on security issues in a timely manner by issuing security releases and associated security advisories. We further plan to enhance our security page where we will be providing a historical accounting of published advisories and will provide a hall of fame for researchers.

As we grow, we will continue our investment in our security team, contributions, tooling and processes. This investment will make Docker safer, helping it become a secure and trusted partner for our users.

You can help! Please report issues to [security@docker.com](mailto:security@docker.com).

For more, see [www.docker.com/resources/security](http://www.docker.com/resources/security).

 

Marianna Tessel

Docker SVP of Engineering

---

Original source: [ADVANCING DOCKER SECURITY: DOCKER 1.4.0 AND 1.3.3 RELEASES](http://blog.docker.com/2014/12/advancing-docker-security-docker-1-4-0-and-1-3-3-releases/)