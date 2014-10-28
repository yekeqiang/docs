# Storage Concepts in Docker: Network and Cloud Storage

---

##### Author: Mark Lamourine  

---

This is the third and final post on storage in Docker.  It's also going to be the most abstract, as most of it now is still wishful.

- [Shared Storage](http://cloud-mechanic.blogspot.com/2014/10/storage-concepts-in-docker.html)
- [Persistent (local) storage](http://cloud-mechanic.blogspot.com/2014/10/storage-concepts-in-docker.html)
- Network and Cloud Storage (this post)

The previous two posts dealt with shared internal storage and persistent host storage in Docker. These two mechanisms allow you to share storage on a single host. While this has its uses, very quickly people find that they need more than local storage

## Types of Storage in Containers

Lots of people are talking about storage in Docker containers.  Not many are careful to qualify what they mean by that. Some of the conversation is getting confused because different people have different goals for storage in containers.

## Docker Internal Storage

This is the simplest form of storage in Docker.  Each container has its own space on the host.  This is inside the container and it is temporary, being created when the container is instantiated and removed some time after the container is terminated.  When two containers reside on the same host they can share this docker-internal storage.

## Host Storage

Containers can be configured to use host storage.  The space must be allocated and configured on the host so that the processes within the containers will have the necessary permissions to read and write to the host storage.  Again, containers on the same host can share storage.

## Network Storage

Or "Network Attached Storage" (NAS) in which I slovenly include Storage Area Networks (SAN).
I'm also including modern storage services like Gluster and Ceph. For container purposes these are the same thing: Storage which is not directly attached via the SCSI or SATA bus, but rather over an IP network but which, once mounted appears to the host as a block device.

If you are running your minions in an environment where you can configure NAS universally then you may be able to use network storage within your Kubernetes cluster.

Remember that Docker runs as root on each minion.  You may find that there are issues related to differences in the user database between the containers, minions and storage.  Until the cgroup user namespace work is finished and integrated with Docker, unifying UID/GID maps will be a problem that requires attention when building containers and deploying them.

## Cloud Storage

Cloud storage is.. well not the other kinds.  It's generally offered in a "storage as a service" model. Most people think of Amazon AWS storage (EBS and S3) but Google is growing its cloud storage and OpenStack offers the possibility of creating on-premise cloud storage services as well.

Cloud storage generally takes two forms.  The first is good old-fashioned block storage. The other is newer and is known as object storage.  They have different behaviors and use characteristics.

## Block Storage

Once it is attached to a host, cloud block storage is indistinguishable from direct attached storage. You can use disk utilities to partition it and create filesystems.  You can mount it so that the filesystem appears within the host file tree.

Block storage requires very low latency.  This means that it is generally limited to relatively local networks.  It works fine within the infrastructure of a cloud service such as AWS or OpenStack, but running block storage over wide area networks is often difficult and prone to failure.

Block storage is attached to the host and then the docker VOLUME mechanism is used to import the storage tree into one or more containers.  If the storage is mounted automatically and uniformly on every minion (and that information is public) then it is possible to use block storage in clusters of container hosts.

## Object Storage

Object storage is a relatively new idea.  For files with a long life that do not change often and can be retrieved as a unit object storage is often a good   They're also good to use as a repository configuration information which is too large or sensitive to be placed in an environment variable or CLI argument.

OpenStack Cinder, AWS S3 and Google Cloud Storage are examples of open source and commercial object stores.

The usage characteristics of object storage make it so that latency is not the kind of issue that it is with block storage.

One other characteristic of object storage makes it really suited to use in containers.  Object storage is usually accessed by retrieval over HTTP using a RESTful protocol.  This means that the container host does not need to be involved in accessing the contents.  So long as the container has the software and the access information for the storage processes within the container can retrieve it.  All that is required is that the container is able to reach the storage service through the host network interface(s).  This makes object storage a strong choice for container storage where ever the other characteristics are acceptable.

## Storage and Kubernetes

Pretty much every application will need storage in some form.  To build large scale containerized applications it will be essential for Kubernetes to make it possible for the containers to access and share persistent storage.  The form that the storage takes will depend on the character of the application and the environment of the cluster.

With all of the forms of NAS (remember, I'm being slovenly) the host is involved in accessing and mounting the storage so that it appears to Docker as if it is normal host storage.  This means that one of three conditions must be met on the host:

1. All of the available storage is mounted on all minions before any containers start
2. The host is configured to automount the storage on the first attempt to access a path
3. This host is able to accept and act on mount requests from Kubernetes

This third also requires modifications to Kubernetes so that the user can specify the path to the required storage and provide any access/authentication information that will be required by the host.

For Cloud block storage the only option is #3 from above.  Google has added a mechanism to mount Google Cloud Engine Persistent Disk volumes into Kubernetes clusters.  The current mechanism (as of 20-Oct-2014) is hard coded.  The developers understand that they will need a plugin mechanism to allow adding AWS EBS, OpenStack Cinder and others. I don't think work on any other cloud storage services has begun yet.

Object storage is the shining light.  While it has limited use cases, those cases are really common and really important. Object  storage access can be built into the image and the only thing the Kubernetes cluster must provide is network access to the object store service.  

## Summary

Generalized shared and cloud storage within Kubernetes clusters (or any cluster of container hosts) is, at this time, an unsolved problem.  Everyone knows it is a top priority and everyone working on the idea of clustered container hosts is thinking about it and experimenting with solutions.  I don't think it will be long before some solutions become available and I'm confident that there will be working solutions within the timeframe of *mumble*. 

For Kubernetes, there is [an open issue](https://github.com/GoogleCloudPlatform/kubernetes/pull/1515) discussing persistent storage options and how to design them into the service, both on the back end and the front end (how does one tell Kubernetes how to access storage for containers?)

I'm going to be playing with a few of the possibilities because I'm going to need them. Until they are available, I can create a Pulp service in Kubernetes, but I can't make it persistent.  Since the startup cost of creating an RPM mirror is huge, it's not much use except as a demonstrator until persistent storage is available.

## References

- [Network Attached Storage](https://en.wikipedia.org/wiki/Network-attached_storage)
	- [Storage Area Network](https://en.wikipedia.org/wiki/Storage_area_network)
	- [Network File System](https://en.wikipedia.org/wiki/Network_File_System) (NFS)
	- [Gluster](http://www.gluster.org/)
	- [Ceph](http://ceph.com/)
	- [iSCSI](https://en.wikipedia.org/wiki/ISCSI)
- [OpenStack Cloud Storage](http://www.openstack.org/software/openstack-storage/) 
	- [Cinder](https://wiki.openstack.org/wiki/Cinder#OpenStack_Block_Storage_.28.22Cinder.22.29) - block storage
	- [Swift](https://wiki.openstack.org/wiki/Swift#OpenStack_Object_Storage_.28.22Swift.22.29) - object storage
- AWS Cloud Storage
	- [EBS](https://aws.amazon.com/ebs/) - block storage
	- [S3](https://aws.amazon.com/s3/) - object storage
- [Google Cloud Storage](https://cloud.google.com/storage/docs)
	- Google Cloud Engine [Persistent Disks](https://cloud.google.com/compute/docs/disks#persistentdisks) - block storage
	- [Google Storage](https://cloud.google.com/storage/) - object storage
	
---

Original source: [Storage Concepts in Docker: Network and Cloud Storage](http://cloud-mechanic.blogspot.jp/2014/10/storage-concepts-in-docker-network-and.html)