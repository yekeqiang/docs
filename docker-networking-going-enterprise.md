# Docker Networking Going Enterprise ?

---

##### Author: Balaji Sivasubramanian 

---

The second revolution in server virtualization is here.  Virtual Machines were the first revolution that allowed users the ability to run multiple workloads on a single server through a hypervisor. Now the next wave is here.  Linux Containers have recently started to gain momentum with many enterprise customers asking me if they should consider it and if Cisco offered Docker support in the enterprise-grade virtual networking products.

I approached my engineers to see whether our recently introduced [Nexus 1000v for the Linux KVM hypervisor](http://www.cisco.com/c/en/us/products/collateral/switches/nexus-1000v-kvm/solution-overview-c22-730808.html), which already has 10000+ customers across various hypervisors, is able to support linux containers or more specifically the popular linux container technology, Dockers.

One of the key advantages of Nexus 1000V today is that it allows easy management of policies for all of the virtual machines.  For example, with a single command or REST API call, a security policy can be deployed or altered across all virtual interfaces connected to a Virtual Extensible LAN (VXLAN). My reasoning was that we should able to extend that to support to linux containers/dockers.

So I approached Tim Kuik (@tjkuik) and Dave Thompson (@davetho610) and much to my delight, they not only said Nexus 1000V can do it but also showed how to do it so that customers can take advantage of this today in their deployments.

I have included Tim and Dave’s how to attach Docker containers to the Nexus 1000v and to assign policies write-up below so that you can try this in your setup.  Happy reading.

### How to use Dockers with Nexus 1000V for KVM Hypervisor: 

![alt](http://resource.docker.cn/docker-networking-going-enterprise-1.png)

Begin by [installing the Nexus 1000v](http://www.cisco.com/c/en/us/products/switches/nexus-1000v-kvm/index.html) to manage one or more Ubuntu servers:   The Nexus 1000v is managed by a Virtual Supervisor Module (VSM).  Once the package is installed on your servers, the servers will appear on the VSM as Virtual Ethernet Modules (VEM).  Below we can see our VSM is managing a server named Bilbo:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-2.png)

We’ve also pre-configured our server to have a port-channel that is capable of carrying vlan traffic 100-109.  We’ve used an Ethernet Port-profile to conveniently manage the uplinks for all of our servers:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-3.png)

A key concept of the Nexus 1000v is that of a Port-profile.  The Port-profile allows for a shared set of port attributes to be cohesively managed in a single policy definition.  This policy can include an ACL definition, Netflow specification, VLAN or VXLAN designation, and/or other common port configuration attributes.  We can, of course, create multiple Port-profiles.  Perhaps we would have one per level of service or tenant.  The Port-profile provides the mechanism to collect and manage the set of containers that share the same policy definition.

Below we create a Port-profile that could be used by any number of containers on any number of servers.

![alt](http://resource.docker.cn/docker-networking-going-enterprise-4.png)

Install docker on your server. [https://docs.docker.com/installation/ubuntulinux/]

![alt](http://resource.docker.cn/docker-networking-going-enterprise-5.png)

The purpose of the container is to run our application. Let’s create one for this example which can handle ssh sessions.  Here is an example Dockerfile which does that: 

![alt](http://resource.docker.cn/docker-networking-going-enterprise-6.png)

At this point, via Docker, you can build an image specified by this Dockerfile.

![alt](http://resource.docker.cn/docker-networking-going-enterprise-7.png)

All the pieces are now in place.  The Nexus 1000v is running.  We have a policy definition that will assign the interfaces to vlan 100 (port-profile vlan100).  Docker is installed on our server.  We have created a useful container image.  Let’s create an actual container from this image:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-8.png)

The container instance started at this point is running with just a loopback interface since we used the argument “–networking=false”.  We can now add an eth0 interface to this container and set it up to be handled by the Nexus 1000v on the host.

Setup a few env variables we will use as part of the procedure.  Find the PID of the running container and generate UUIDs to be used as identifiers for the port and container on the Nexus 1000v:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-9.png)

In this example the following PID and UUIDs were set:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-10.png)

Create a linux veth pair and assign one end to the Nexus 1000v.  We will use the port-profile defined on the VSM named “vlan100’ which will configure the port to be an access port on VLAN 100:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-11.png)

When an interface is added to the Nexus 1000v, parameters are specified for that interface by adding keys to the external_ids column of the Interface table.  In the example above the following keys are defined:

- iface-id: Specifies the UUID for the interface being added. The Nexus 1000v requires a UUID for each interface added to the switch so we generate one for this.
- attached-mac: Provides the MAC of the connected interface. We get this from the ‘ip link show’ command for the interface to be added to the container.
- profile:  Provides the name of the port-profile which the Nexus 1000v should use to configure policies on the interface.
- vm-uuid: Specifies the UUID for the entity which owns the interface being added.  So in this case that’s the container instance.  Since Docker doesn’t create a linux type UUID for the container instance, we generate one for this as well.
- vm-name: Specifies the name of the entity which owns the interface.  In this case it’s the container name.

Move the other end of the linux veth pair to the container’s namespace, rename it as eth0, and give it a static IP address of 172.22.64.201 (of course DHCP could be used instead to assign the address):

![alt](http://resource.docker.cn/docker-networking-going-enterprise-12.png)

On the Nexus 1000v’s VSM you will see logs like this indicating the interface has been added as switch port vethernet1 on our virtual switch:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-13.png)

The following VSM commands show that switch port veth1 is up on VLAN 100 and is connected to host interface veth18924_eth0 on host bilbo:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-14.png)

On the host bilbo we can use vemcmd to get information on the port status:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-15.png)

That’s it.  We now have a useful Docker container with an interface on the Nexus 1000v using our selected policy.   Using another server (and/or container) that is on the same vlan, we can ssh into this container using the IP address we assigned:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-16.png)

When shutting down the Docker container, remove the port before removing the container:

![alt](http://resource.docker.cn/docker-networking-going-enterprise-17.png)

---

Original source: [Docker Networking Going Enterprise?](http://blogs.cisco.com/datacenter/docker-networking-going-enterprise/)
