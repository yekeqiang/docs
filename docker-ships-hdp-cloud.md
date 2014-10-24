# Docker Ships HDP Into the Cloud

---

##### Author: Lisa Sensmeier

---


*[SequenceIQ](http://www.hortonworks.com/partner/sequenceIQ)* is a new Hortonworks Technology Partner and recently achieved HDP and YARN Ready certification for Cloudbreak, the SequenceIQs Hadoop as a Service API. In this guest blog, SequenceIQ Co-founder and CTO Janos Matyas (@sequenceiq), describes provisioning and autoscaling [HDP](http://hortonworks.com/hdp) cluster with Cloudbreak.

During our daily work at [SequenceIQ](http://www.hortonworks.com/partner/sequenceIQ), we are provisioning HDP clusters on different environments. Be it for a random cloud provider or on bare metal, we were looking for a common solution to automate and speed up the process. Welcome [Docker](https://www.docker.com/)—this case study shows how easy it is to provision and autoscaling HDP cluster using **Cloudbreak**.

> What is the origin of the name Cloudbreak?

> Cloudbreak is a powerful left surf that breaks over a coral reef, a mile off southwest the island of Tavarua, Fiji. Cloudbreak the product is a cloud agnostic Hadoop as a Service API that abstracts the provisioning and ease of management and monitoring of on-demand clusters.

## The provisioning challenge

As we have discussed in a previous [blog post](http://hortonworks.com/blog/searching-apache-hadoop-provisioning-swiss-army-knife/), we use [Apache Ambari](http://hortonworks.com/hadoop/ambari) quite a lot and have built toolsets around (Ambari Shell, Ambari REST client) and contributed these back to the Ambari community. While with our contribution we were able to automate most of the HDP provisioning, the infrastructure part was still a missing piece. We needed to find a way to be able to use the same process, toolset and API’s to provision HDP – literarily anywhere. We were among the first Docker adopters and started to use the “containerized” version of the [HDP sandbox](http://hortonworks.com/products/hortonworks-sandbox/) to ease our development process – and from there it was only a step away to have a fully functional Docker based HDP cluster on bare metal, initially merely for development purposes.

We commonly use different cloud providers. After we had “containerized” HDP for bare metal, we came up with the idea of [Cloudbreak](http://blog.sequenceiq.com/blog/2014/07/18/announcing-cloudbreak/) – the open source, cloud agnostic and autoscaling Hadoop as a service API. While Cloudbreak’s primary role is to launch on-demand Hadoop clusters in the cloud, the underlying technology actually does more. It can launch on-demand Hadoop clusters in any environment that supports Docker – in a dynamic way. There is no predefined configuration needed as all the setup, orchestration, networking and cluster membership are done dynamically.

Here are the components to the solution:

- Docker containers – all the Hadoop services are installed and running inside Docker containers, and these containers are shipped between different cloud vendors, keeping Cloudbreak cloud agnostic.

- Apache Ambari – to declaratively define a Hadoop cluster.

- Serf – for cluster membership, failure detection, and orchestration that is decentralized, fault-tolerant and highly available for dynamic clusters.

- dnsmasq – to provide resolvable fully qualified domain names between dynamically created Docker containers.

## Autoscaling and SLA policies

Now that we have an open source Hadoop as a Service API that runs HDP in the cloud, we moved forward, and wanted to have an open source SLA policy based autoscaling API which works with Cloudbreak or a Hadoop YARN cluster. Welcome Periscope.

> Where does Periscope name come from?

> Periscope is a powerful, fast, thick and top-to-bottom right-hander, eastward from Sumbawa’s famous west-coast. Timing is critical, as it needs a number of elements to align before it shows its true colors.

> Obviously we have a surfing related naming theme here!

> Periscope the product brings QoS and autoscaling to Hadoop YARN. Built on cloud resource management and YARN schedulers, it allows to associate SLA policies to applications.

We followed up the same route as we did with Ambari: identified the key components and features we considered that would better fit into the [Apache Hadoop YARN](http://hortonworks.com/hadoop/yarn) codebase and contributed there. The API allows configuring metric based alarms and creates [SLA scaling policies](http://blog.sequenceiq.com/blog/2014/09/01/sla-samples-periscope/) to dynamically adjust the size of your HDP cluster.

Beside the policies, we provide a visual monitoring dashboard – collecting over 400 metrics from the cluster from different sources (RM, timeline/history server, Metrics2 sinks). End users can drill down at node or component level and identify problems and view logs, by using the default queries or configuring custom ones.

## DevOps toolsets, resources

When we start a project, we always approach it from a very strong DevOps perspective. It was the same for Cloudbreak and Periscope, and we have created toolsets to ease and automate your HDP cluster provision on any environment.

- [Cloudbreak UI](https://cloudbreak.sequenceiq.com/) – a responsive and intuitive UI to create HDP clusters
- [Cloudbreak REST client](https://github.com/sequenceiq/cloudbreak-rest-client) – a Groovy based REST client to work with Cloudbreak’s REST API
- [Cloudbreak CLI/shell](https://github.com/sequenceiq/cloudbreak-shell) – a CLI to provision HDP clusters using a shell
- [Cloudbreak API](http://docs.cloudbreak.apiary.io/) – a secure REST API to create HDP clusters on your favorite cloud
- [Periscope API](http://docs.periscope.apiary.io/) – a secure REST API to configure SLA policies for your cluster

## Try it out

We have a hosted version of [Cloudbreak](https://cloudbreak.sequenceiq.com/) where you can create your arbitrary size HDP cluster with support for the full stack on your favorite cloud provider. Give it a try and let us know how it works for you. Provisioning a HDP cluster has never been easier and faster – and the options to do so are listed above (UI, REST client, CLI shell, REST calls). Stay tuned, as we will be announcing cool things with the next release as well come up with a follow up post with deeper technical details.

---

Original source: [Docker Ships HDP Into the Cloud](http://hortonworks.com/blog/docker-ships-hdp-cloud/)