# Simple Monitoring for Docker (Part I)

---

Author: Christophe Labouisse

---

Migrating from VMs to Docker containers is quite easy except for the monitoring part. A straightforward approach, running a data collecting agent (such as [Zabbix](http://www.zabbix.com/) agent), is definitely a not a good solution as it goes against Docker’s philosophy of having one clearly identified task in each container and also because it will require to use custom images. Starting with [Gathering LXC and Docker Containers Metrics](http://blog.docker.com/2013/10/gathering-lxc-docker-containers-metrics/) I came to a simple script based system to gather metrics from Docker containers.

I’m using Zabbix to aggregates the performance metrics so the scripts will be designed to be used in a Zabbix agent [user parameter](https://www.zabbix.com/documentation/2.4/manual/config/items/userparameters). A user parameter is basically a script run by Zabbix and returning some information. User parameters have to be defined in the agent configuration file but may receive arguments in order to multiple informations from single script.

In spite of being created with Zabbix in mind the rest of this post should be applicable to pretyy much any decent monitoring system.

## Host Metrics

These metrics are gathered on the Docker host level rather than the container level. This is more a warm up, a proof of concept or a smoke test to assert that everything is installed correctly in my monitoring system. The goal is to collect several metrics related to the containers:

1. the number of running containers
2. the total number of defined containers
3. the number of crashed containers i.e. how many stopped containers have a non zero exit code.

Below is a straightforward shell implementation:

```
#!/bin/bash
 
function countContainers() {
	docker ps -q $1 | wc -l
}
 
function countCrashedContainers() {
	docker ps -a | grep -v -F 'Exited (0)' | grep -c -F 'Exited ('
}
 
TYPE=${1-all}
 
case $TYPE in
	running) COUNT_FUNCTION="countContainers"; shift;;
	crashed) COUNT_FUNCTION="countCrashedContainers"; shift;;
	all) COUNT_FUNCTION="countContainers -a"; shift;;
esac
 
$COUNT_FUNCTION
```

After some Zabbix configuration this leads to a graph looking like this on my box:

![alt](http://resource.docker.cn/number-of-containers.png)

Since I have 9 containers running permanently, 3 data containers and one container started by cron every hour this is consistent with my expectations.

A similar script could be written to gather metrics on the images such as the total number of images and how many of them are dangling.

## Container Metrics

For a start I wanted to collect for each container the following metrics:

1. the container IP address
2. the container status (running, paused, stopped, crashed)
3. the user and system CPU time
4. the memory used by the container processes
5. the network activity (in and out)

### IP Address and Container Status

Those are found in `docker inspect <container-id>`. The IP address is found in `NetworkSettings.IPAddress` and I compute the status from `State` to get the following values:

- 0 -> Running
- 1 -> Paused
- 2 -> Stopped
- 3 -> Crashed (i.e. stopped with non zero exit code)

### CPU and Memory

CPU and memory will be retrieved by peeking under the `/sys/fs/cgroup/docker` hierarchy in the `cpuacct.stat` and `memory.stat` files.

### Network activity

According to the blog article, retrieving the network activity is far more complicated than retrieving the CPU or Memory and I was not a big fan of the method mentioned in the article. However those data are quite easy to retrieve from inside the container for instance by running a simple `ifconfig eth0` command or by peeking in the /sys hierarchy. Thanks to exec command introduced in Docker 1.3, running a command into a running container can now be done easily without requiring any custom image or any special command when starting the container.

## The Script

In order to collect those metrics I created the following python script:

```
#!/usr/bin/env python
 
__author__ = 'Christophe Labouisse'
 
import argparse
import re
import os
 
from docker import Client
from docker.utils import kwargs_from_env
 
 
def display_cpu(args):
    detail = c.inspect_container(args.container)
    if bool(detail["State"]["Running"]):
        container_id = detail['Id']
        cpu_usage = {}
        with open('/sys/fs/cgroup/cpuacct/docker/' + container_id + '/cpuacct.stat', 'r') as f:
            for line in f:
                m = re.search(r"(system|user)\s+(\d+)", line)
                if m:
                    cpu_usage[m.group(1)] = int(m.group(2))
        if args.type == "all":
            cpu = cpu_usage["system"] + cpu_usage["user"]
        else:
            cpu = cpu_usage[args.type]
        user_ticks = os.sysconf(os.sysconf_names['SC_CLK_TCK'])
        print(float(cpu) / user_ticks)
    else:
        print(0)
 
 
def display_ip(args):
    detail = c.inspect_container(args.container)
    print(detail['NetworkSettings']['IPAddress'])
 
 
def display_memory(args):
    detail = c.inspect_container(args.container)
    if bool(detail["State"]["Running"]):
        container_id = detail['Id']
        with open('/sys/fs/cgroup/memory/docker/' + container_id + '/memory.stat', 'r') as f:
            for line in f:
                m = re.search(r"total_rss\s+(\d+)", line)
                if m:
                    print(m.group(1))
                    return
 
    print(0)
 
 
def display_network(args):
    detail = c.inspect_container(args.container)
    if bool(detail["State"]["Running"]):
        ifconfig = c.execute(args.container, "ifconfig eth0")
        m = re.search(("RX" if args.direction == "in" else "TX") + r" bytes:(\d+)", str(ifconfig))
        if m:
            print(m.group(1))
        else:
            b = c.execute(args.container, "cat /sys/devices/virtual/net/eth0/statistics/"+("rx" if args.direction == "in" else "tx")+"_bytes")
            if re.match(r"\s*\d+\s*", b):
                print(b)
            else:
                print(0)
    else:
        print(0)
 
 
def display_status(args):
    detail = c.inspect_container(args.container)
    state = detail["State"]
    if bool(state["Paused"]):
        print(1) # Paused
    elif bool(state["Running"]):
        print(0) # Running
    elif int(state["ExitCode"]) == 0:
        print(2) # Stopped
    else:
        print(3) # Crashed
 
 
parser = argparse.ArgumentParser()
 
parser.add_argument("container", help="Container name")
 
subparsers = parser.add_subparsers(title="Counters", description="Available counters", dest="dataType")
 
cpu_parser = subparsers.add_parser("cpu", help="Display CPU usage")
cpu_parser.add_argument("type", choices=["system", "user", "all"])
cpu_parser.set_defaults(func=display_cpu)
 
ip_parser = subparsers.add_parser("ip", help="Display IP Address")
ip_parser.set_defaults(func=display_ip)
 
memory_parser = subparsers.add_parser("memory", help="Display memory usage")
memory_parser.set_defaults(func=display_memory)
 
network_parser = subparsers.add_parser("network", help="Display network usage")
network_parser.add_argument("direction", choices=["in", "out"])
network_parser.set_defaults(func=display_network)
 
status_parser = subparsers.add_parser("status", help="Display the container status")
status_parser.set_defaults(func=display_status)
 
c = Client(**(kwargs_from_env()))
 
args = parser.parse_args()
args.func(args)
```

## Conclusion of Part I

Using beginner level python programming we are now able to retrieve some interesting metrics:

- the IP Address
- the container status
- the total CPU time consumed by the container (in seconds)
- the total memory used by the container
- the number of bytes sent or received by the container.

While some datar such as the memory usage or the ip address are directly usable others like the CPU or the network activity will require post processing as we are interested in the changes rather than the total value. This is totally OK and the computation of deltas will be left to the monitoring system (Zabbix).

The worst part of it is the retrieval of the network activity which is a little bit hackish. While I loved the used of the `ifconfig` command I found out that some images (like the official Mongo image) does not provide the command hence the fallback to the `/sys` hierarchy. A cleaner solution would be to query the virtual interface from the host but at the moment there is no easy way to retrieve the virtual interface assigned to a container unless I missed something.

---

Original source: [Simple Monitoring for Docker (Part I)](http://www.labouisse.com/how-to/2014/11/17/simple-monitoring-for-docker-part-1/)