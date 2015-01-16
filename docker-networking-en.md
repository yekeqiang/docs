# Docker networking

---

##### Author: Pedro Marques

---

When docker launches a linux container it will, by default, assign it a private IP address out of [RFC 1918](https://tools.ietf.org/html/rfc1918) space. It connects this container to the host OS using a bridged interface (docker0). Connectivity between the outside world and the container depends on NAT.

Outbound traffic is NATed using the host’s IP address. Inbound traffic requires explicit port mapping rules that map a port on the host to a port in the container. Given that typically one runs multiple containers in the same host there needs to be a map between a host port (in the dynamic port range) and a service port on the container.

For example, the HTTP service port (80) in container-1 will be mapped to port 49153 while container-2 would see its HTTP port mapped to host port 49154. Ports that are not explicitly mapped cannot receive incoming traffic. Also containers within the same host will see different IP address ports than containers across different hosts (not very ‘cloudy’).

This is the reason why using a network virtualization solution such as [OpenContrail](http://opencontrail.org/docker-with-opencontrail/) is so appealing. OpenContrail, replaces docker’s networking implementation which can be disabled by using –net=none. It provides each container its own IP address in the overlay without the need to do port mapping.

OpenContrail can then be used to provide network based isolation, network based policies, floating-ip address, load-balancing and traffic monitoring capabilities.

The basic construct used by OpenContrail is a virtual-network. Virtual-networks are used to map a set of instances that have a common administrative characteristic. Typically an application tier maps to a virtual network.

Administrators can then define which virtual-networks are allowed to communicate with each other and control which services (ports) are allowed between a set of networks. For instance, one may define that a set of front-end instances can send its logs to a management network running syslog servers by establishing a TCP connection to the appropriate port; but not ssh into the same servers.

With floating-ips an external address can be assigned to a specific container; with the LBaaS API the floating-ip (or external address) can be associated with a set of back-ends.

The OpenContrail vrouter records all traffic flows to a centralized time-series database. The analytics component can then access this database and serve queries that provide visibility into the traffic patterns of the network.

OpenContrail can be used with and without OpenStack. The same OpenContrail install can span both domains providing a consistent API interface as well as direct distributed routing functionality.
When docker launches a linux container it will, by default, assign it a private IP address out of RFC 1918 space. It connects this container to the host OS using a bridged interface (docker0). Connectivity between the outside world and the container depends on NAT.

Outbound traffic is NATed using the host’s IP address. Inbound traffic requires explicit port mapping rules that map a port on the host to a port in the container. Given that typically one runs multiple containers in the same host there needs to be a map between a host port (in the dynamic port range) and a service port on the container.

For example, the HTTP service port (80) in container-1 will be mapped to port 49153 while container-2 would see its HTTP port mapped to host port 49154. Ports that are not explicitly mapped cannot receive incoming traffic. Also containers within the same host will see different IP address ports than containers across different hosts (not very ‘cloudy’).

This is the reason why using a network virtualization solution such as OpenContrail is so appealing. OpenContrail, replaces docker’s networking implementation which can be disabled by using –net=none. It provides each container its own IP address in the overlay without the need to do port mapping.

OpenContrail can then be used to provide network based isolation, network based policies, floating-ip address, load-balancing and traffic monitoring capabilities.

The basic construct used by OpenContrail is a virtual-network. Virtual-networks are used to map a set of instances that have a common administrative characteristic. Typically an application tier maps to a virtual network.

Administrators can then define which virtual-networks are allowed to communicate with each other and control which services (ports) are allowed between a set of networks. For instance, one may define that a set of front-end instances can send its logs to a management network running syslog servers by establishing a TCP connection to the appropriate port; but not ssh into the same servers.

With floating-ips an external address can be assigned to a specific container; with the LBaaS API the floating-ip (or external address) can be associated with a set of back-ends.

The OpenContrail vrouter records all traffic flows to a centralized time-series database. The analytics component can then access this database and serve queries that provide visibility into the traffic patterns of the network.

OpenContrail can be used with and without OpenStack. The same OpenContrail install can span both domains providing a consistent API interface as well as direct distributed routing functionality.

---

Original source: [Docker networking](http://pedrormarques.wordpress.com/2014/10/24/docker-networking/)