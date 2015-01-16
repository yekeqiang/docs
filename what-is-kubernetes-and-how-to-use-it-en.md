# WHAT IS GOOGLE’S KUBERNETES AND HOW TO USE IT

---

##### Author: Laura Frank

---

![alt](http://resource.docker.cn/lhc-cern.jpg)

What do you do when you want Docker containers managed across vast fleets of servers and infrastructure? You use Docker orchestration tools like Kubernetes.

Developed by Google, [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262) is essentially a cluster manager for Docker. With it, you can schedule and deploy any number of container replicas onto a node cluster and Kubernetes will take care of making decisions like which containers go on which servers for you.

The name Kubernetes originates from Greek, meaning “helmsman” or “pilot,” and that’s the role it will fill in your Docker workflow. Kubernetes is a solution for overseeing and managing multiple containers at scale, rather than just working with Docker on a manually-configured host.

## WHAT DOCKER IS… AND WHAT IT ISN’T

Docker and its ecosystem are great for managing images, and running containers in a specific host. But these interactions are local; Docker alone can’t manage your containers across multiple nodes, or schedule and manage tasks to be completed across your cluster. Docker alone works best when you’re able to manually manipulate and configure the host. But we all know that only takes us so far.

In order to manage a Docker workload on a distributed cluster, you need to bring some other players into the mix. Several other services are available to make working with Docker more efficient and streamlined. Fleet, Geard, and Marathon are designed to schedule and orchestrate jobs on the host; Fleet also functions as a cluster manager, along with Apache Mesos. Theoretically speaking, you could mix and match these different components and come up with a roll-your-own solution.

## WHY KUBERNETES?

While Kubernetes was designed to make working with containers on Google Compute Engine easier, the bits are available for anyone to use, and you need not be running GCE. Kubernetes offers a few distinct advantages, first and foremost being that it packages all necessary tools — orchestration, service discovery, load balancing — together in one nice package for you. Kubernetes also boasts heavy involvement from the developer community. Written in go, [the Kubernetes project](https://github.com/GoogleCloudPlatform/kubernetes?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262) has close to 2500 commits from over 100 different contributors. Despite heavy development, Kubernetes is still in beta, so you may discover some bugs.

## DECONSTRUCTING THE PARTS OF KUBERNETES

While Kubernetes does work as a cohesive package, there are several components at play, each with a specific role. Kubernetes also has a specific collection of terms, some of which are overloaded in the container/cloud space.

- Master: the managing machine, which oversees one or more minions.
- Minion: a slave that runs tasks as delegated by the user and Kubernetes master.
- Pod: an application (or part of an application) that runs on a minion. This is the basic unit of manipulation in Kubernetes.
- Replication Controller: ensures that the requested number of pods are running on minions at all times.
- Label: an arbitrary key/value pair that the Replication Controller uses for service discovery
- kubecfg: the command line config tool
- Service: an endpoint that provides load balancing across a replicated group of pods

To manage resources within Kubernetes, you will interact with the Kubernetes API. Pulling down the Kubernetes binaries will give you all the services necessary to get your Kubernetes configuration up and running. Like most other cluster management solutions, Kubernetes works by creating a master, which exposes the Kubernetes API, allowing you to request certain tasks to be completed. The master then spawns containers to handle the workload you’ve asked for.

![alt](http://resource.docker.cn/kubernetes-architecture.png)

Aside from running Docker, each node runs the Kubelet service — which is an agent that works with the container manifest — and a proxy service.

The Kubernetes control plane is comprised of many components, but they all run on the single Kubernetes master node.

## HOW TO USE KUBERNETES

Let’s assume you’ve pulled down and started the Kubernetes services, and you’re ready to build your first pod. Interacting with Kubernetes is quite simple, even outside of the context of Google Compute Engine. Kubernetes provides a RESTful API in order to manipulate the three main resources: pods, services, and replicationControllers.

### PODS

A `pod` file is a json representation of the task you want to run. A simple pod configuration for a Redis master container might look like this:

In this example — from the Kubernetes [example project](https://github.com/GoogleCloudPlatform/kubernetes/tree/master/examples/guestbook?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262) — you can see information pertaining to the service’s source image, but also some Kuberentes-specific information, like the labels used for service discovery.

Let’s take this pod and actually run it on a single-node cluster using Kubernetes. You can use the kubecfg tool to send a request to your running Kubernetes API server:

```
$ kubecfg -c my-pod-directory/redis-master-pod.json create /pods
```

Then, the kubelet starts the service. Verify the pod is running by listing the pods:

```
$ kubecfg list /pods
```

After you’re finished, you can delete your pod:

```
$ kubecfg delete /pods/redis-master-pod
```

This is a very rudimentary example of working with Kubernetes, but it does illustrate the way in which you interact with the Kubernetes API in order to manipulate resources. For this example, only `pods` were handled, though the API also handles `replicationControllers` and `services`.

### REPLICATION CONTROLLERS

Along with the `service`, the `replicationController` uses the pod’s labels in order to do its job. In short, the role of the `replicationController` is to ensure that a certain number of replicas of each pod are running. If there are too many, it will kill them, and if there are too few, it spins them up. The pods that the `replicationController` monitors is determined via labels.

A simple example of a redisSlaveController looks something like this, again coming from the Kubernetes example application:

```
23
{
  "id": "redisSlaveController",
  "kind": "ReplicationController",
  "apiVersion": "v1beta1",
  "desiredState": {
    "replicas": 2,
    "replicaSelector": {"name": "redisslave"},
    "podTemplate": {
      "desiredState": {
         "manifest": {
           "version": "v1beta1",
           "id": "redisSlaveController",
           "containers": [{
             "name": "slave",
             "image": "brendanburns/redis-slave",
             "ports": [{"containerPort": 6379, "hostPort": 6380}]
           }]
         }
       },
       "labels": {"name": "redisslave"}
      }},
  "labels": {"name": "redisslave"}
}
```

In short, this file tells Kubernetes that you want two replicas of the Redis template you’ve defined, and from there, Kubernetes works to make sure you have two running at all times.

### SERVICES

Kubernetes perhaps chose a poor word for this resource, since service is already so overloaded. But to Kubernetes, a `service` is a config unit for the proxies running on the minion nodes. It too can have a name and labels, and it points to one or more pods. It provides an endpoint for load balancing across a replicated group of pods.

Below, you can see a simple schematic of what an example application could look like with Kubernetes.

![alt](http://resource.docker.cn/an-example-application.png)


## USEFUL LINKS

Now that you have a basic understanding of Kubernetes, go forth and built something! Here are a few resources to help you get started.

- [A deeper dive in to the design and architecture of Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/DESIGN.md?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262#the-kubernetes-node)

- [Kubernetes Documentation](https://github.com/GoogleCloudPlatform/kubernetes/tree/master/docs?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262)

- [Running Kubernetes on CoreOS](https://coreos.com/blog/running-kubernetes-example-on-CoreOS-part-1/?__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262)

- [Google IO 2014: Containerizing the Cloud](http://www.youtube.com/watch?v=tsk0pWf4ipw&__hstc=257401556.d7fdcd68297a3daa8ae7f8c9e95fcb96.1414661217786.1414661217786.1414661217786.1&__hssc=257401556.1.1414661217787&__hsfp=2081402262)

---

Original source: [WHAT IS GOOGLE’S KUBERNETES AND HOW TO USE IT](http://www.centurylinklabs.com/what-is-kubernetes-and-how-to-use-it/)