# Hanlon in Docker! (Updated)

---

Author: Joseph Callen

---

[Hanlon](https://github.com/csc/Hanlon) can be initially a difficult install hopefully a Docker container should make it easier. So what we have today is Hanlon running in Ubuntu Precise container using Puma. The Mongo database container is provided from the community. We are still working on the rest of the prerequisites which includes TFTP, DHCP and iPXE (custom options for ESXi).

Update: The image has been pushed [here](https://registry.hub.docker.com/u/jcpowermac/hanlon/).

~~Since I don't have this uploaded to any repository yet copy this file locally and call it `Dockerfile`~~. Leaving this as an example.

```
FROM ubuntu:precise
MAINTAINER Joseph Callen <jcpowermac@gmail.com>

EXPOSE 8026

RUN apt-get -y update \
    && apt-get -y install ruby1.9.3 git build-essential libssl0.9.8 libssl-dev \
    && apt-get autoremove \
    && apt-get clean \
    && rm -rf /tmp/* /var/tmp/* /var/lib/apt/lists/*

RUN gem install bundle
RUN mkdir /home/hanlon 
RUN git clone https://github.com/csc/Hanlon.git /home/hanlon
WORKDIR /home/hanlon
RUN bundle install --system
RUN ./hanlon_init

WORKDIR /home/hanlon/web
CMD sed -i "s/127.0.0.1/$MONGO_PORT_27017_TCP_ADDR/g" /home/hanlon/web/config/hanlon_server.conf && ./run-puma.sh
```

Hanlon depends on mongodb running so lets start that first.

```
docker run -d --name mongodb dockerfile/mongodb
```

Next lets start the hanlon container.

```
docker run -d -p 8026:8026 --name hanlon --link mongodb:mongo jcpowermac/hanlon
```

Display the current running containers.

```
[root@fedora ~]# docker ps
CONTAINER ID        IMAGE                       COMMAND                 CREATED             STATUS              PORTS                    NAMES
79eef606a06c        jcpowermac/hanlon:latest    "\"/bin/sh -c 'sed -   11 minutes ago      Up 11 minutes       0.0.0.0:8026->8026/tcp   hanlon              
44f88c5c3efc        dockerfile/mongodb:latest   "mongod"               16 minutes ago      Up 16 minutes       27017/tcp, 28017/tcp     mongodb 
```
   
And just a quick `GET http://127.0.0.1:8026/hanlon/api/v1/config`

```
{
resource: "ProjectHanlon::Slice::Config"
command: "get_config"
result: "Ok"
http_err_code: 200
errcode: 0
response: {
@admin_port: 8025
@api_port: 8026
@api_version: "v1"
@base_path: "/hanlon/api"
@daemon_min_cycle_time: 30
@force_mk_uuid: ""
@hanlon_log_level: "Logger::ERROR"
@hanlon_server: "172.17.0.45"
@hnl_mk_boot_debug_level: "Logger::ERROR"
@hnl_mk_boot_kernel_args: ""
@image_path: "/home/hanlon/image"
@ipmi_password: ""
@ipmi_username: ""
@ipmi_utility: ""
@mk_checkin_interval: 60
@mk_checkin_skew: 5
@mk_gem_mirror: "http://localhost:2158/gem-mirror"
@mk_gemlist_uri: "/gems/gem.list"
@mk_kmod_install_list_uri: "/kmod-install-list"
@mk_log_level: "Logger::ERROR"
@mk_tce_install_list_uri: "/tce-install-list"
@mk_tce_mirror: "http://localhost:2157/tinycorelinux"
@node_expire_timeout: 300
@persist_dbname: "project_hanlon"
@persist_host: "172.17.0.46"
@persist_mode: "mongo"
@persist_password: ""
@persist_port: 27017
@persist_timeout: 10
@persist_username: ""
@register_timeout: 120
@sui_allow_access: "true"
@sui_mount_path: "/docs"
}-
}
```

Update: below is an ansible playbook to deploy the required packages and containers.

```

---
- hosts: all
  sudo: yes
  tasks:
  - name: ensure docker-io is installed
    yum: name=docker-io state=latest

  - name: ensure python-docker-py is installed
    yum: name=python-docker-py state=latest

  - name: ensure mongo is running
    docker: image=dockerfile/mongodb name=mongodb

  - name: ensure hanlon container is running
    docker: image=jcpowermac/hanlon ports=8026:8026 links=mongodb:mongo name=hanlon
```

## DOCKER IS COOL!

Sources: I certainly didn't do this by myself

- [http://docs.docker.com/userguide/dockerlinks/](http://docs.docker.com/userguide/dockerlinks/) 
- [http://docs.docker.com/articles/networking/](http://docs.docker.com/articles/networking/)
- [https://blog.codecentric.de/en/2014/01/docker-networking-made-simple-3-ways-connect-lxc-containers/](https://blog.codecentric.de/en/2014/01/docker-networking-made-simple-3-ways-connect-lxc-containers/) 
- [http://stackoverflow.com/questions/23692470/why-cant-i-use-docker-cmd-multiple-times-to-run-multiple-services](http://stackoverflow.com/questions/23692470/why-cant-i-use-docker-cmd-multiple-times-to-run-multiple-services) 
- [http://docs.docker.com/reference/builder/](http://docs.docker.com/reference/builder/)
- [https://registry.hub.docker.com/u/dockerfile/mongodb/](https://registry.hub.docker.com/u/dockerfile/mongodb/)
- [http://docs.ansible.com/docker_module.html](http://docs.ansible.com/docker_module.html)
