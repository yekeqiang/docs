# Docker Reports on Engagement and Performance Metrics

---

##### Author: Mark Boyd

---

Docker is undoubtedly the growth story of 2014. [Andrew Clay Shafer has already pointed out](http://thenewstack.io/containers-whats-new-what-isnt-what-matters/) how Docker’s containerization introduced convenient defaults and image management, which have been crucial in making the “technology accessible and compelling for mass adoption.” Mårten Mickos from HP-acquired Eucalypt Systems sees Docker as akin to [mopeds in a big city](http://thenewstack.io/marten-mickos-docker-containers-are-analogous-to-mopeds-in-a-big-city/) (they may be the new hotness, but you still need to wear protective clothing), while [cloud computing influencer Bernard Golden](http://thenewstack.io/what-docker-does-and-does-not-deliver/) believes Docker will “revolutionize the application world” if it is able to foster a “surrounding ecosystem of functionality so that Docker users can obtain all the benefits they desire.”

The first meeting of Docker’s first Governance Advisory Board (DGAB) last week reflected the key trends that The New Stack has been covering in the above linked articles, mostly, how does an open source project continue to grow developer engagement, while also supporting the ecosystem to stabilize? Maintaining scalability while managing expectations may be a common challenge for new open source projects, but Docker needs to do so while moving at hyperdrive speed.

The DGAB consists of Solomon Hykes (Docker CTO) and other lead maintainers at Docker, as well as representatives across three categories: Contributors (the top four contributors of merged, non-trivial pull requests for the past six months), Corporate Seats (Google, Rackspace, IBM and Red Hat), and Users (Atlassian, eBay, Spotify and Tutum).

Uptake of Docker has been almost universal: Large companies and new startups alike across verticals including e-commerce, media, life sciences, and IT SaaS/IaaS/PaaS are all talking publicly about using Docker. Other industries are also investing in Docker projects, with the Governance Board being told that “Virtually every major large investment bank has a Docker related initiative underway”.

## Docker Engagement Metrics

While Docker was launched in March 2013, it has really only been the last 6 months that growth hit that hyperdrive rate. Adoption numbers from immediately prior to DockerCon — held in June this year — and reviewed at mid-October show that the majority of uptake is new.

Contributor numbers overall are growing at a half-yearly rate of 44%, from 452 contributors at the time of DockerCon in early June, to 650 contributors in mid-October. Most impressive is the growth rate of container pulls (counted in the millions), which is clocking in at 1387% growth in the half year, an indicator of actual usage of Docker amongst the developer community. Other engagement metrics (all measured in the thousands) are equally impressive:

![alt](http://resource.docker.cn/docker-engagement-growth.png)

For the DGAB, Docker also compared their growth rate against other open source projects including Ansible, Chef and Cassandra. Only Meteor showed a similar ascendancy, although their growth has been spread over an eighteen month time frame rather than Docker´s six-month growth spurt.

Docker’s inflection point after June 2014’s DockerCon can also be seen in other independently collated metrics. Website stats services [SimilarWeb](http://www.similarweb.com/website/docker.com) and [Compete](https://siteanalytics.compete.com/docker.com/) both show sudden, sustained jumps in website traffic to Docker.com after June 2014, with SimilarWeb estimating 3.8 million global visitors on average in every month since DockerCon.

## Docker Performance

As developers are on-boarding — and particularly amongst diverse industry verticals — Golden’s point about supporting a ‘surrounding ecosystem of functionality’ becomes important for this growth to transform into a loyal developer community.

Docker uses pull requests data as its core metric for demonstrating its performance at responding to user needs, and therefore being able to be responsive to the various functional needs of the Docker ecosystem. Most encouragingly, overall pull requests are being closed just as much by community contributors as by Docker-staffed maintainers (almost perfectly split down the middle with 50.35% of pull requests closed by community contributors). This suggests an active and engaged open source developer community is truly being formed around Docker.

On average, pull requests remain open for 7.2 days. DGAB User Representative [Nicola Paolucci notes](https://blog.docker.com/2014/11/guest-post-notes-on-the-first-docker-advisory-board-meeting/) that while pull requests are being handled efficiently, more complex pull requests may be taking longer because of a focus on high quality standards: “the maintainers really care about clean design and are wary to accept a big drop of tens of thousands of lines without some previous design discussion happening. They want to get better at fostering a conversation before the big drops happen,” writes Paolucci.

## Mapping the Road Ahead

Perhaps the first step for the Docker team going forward will be to better delineate Docker the open source project from the commercial Docker Incorporated. According to Paolucci, several of the conversations continued to circle back to seeking a clarification on exactly where the line is drawn. As a result, Docker will be establishing a separate organization — ‘Team Meta’ — focused “solely on making the project more effective”. Team Meta will be responsible for reviewing pull requests, dedicating resources for tooling, and updating [the roadmap](https://github.com/docker/docker/blob/master/hack/ROADMAP.md) with interactive ways to gauge progress on key functional areas like networking and storage.

This roadmap will be crucial to share as part of the open culture Docker wants to demonstrate. APIs are at the heart of much of the work that needs to be done. Whether it be in orchestration capabilities, integration and customization via plugins, or dynamic networking and storage, APIs are seen as the key mechanism to advance these goals. A clearly documented roadmap will help users and ecosystem partners plan (and contribute) accordingly.

---

Original source: [Docker Reports on Engagement and Performance Metrics](http://thenewstack.io/managing-growth-and-fostering-an-ecosystem-in-the-open-docker-reports-on-engagement-and-performance-metrics/)