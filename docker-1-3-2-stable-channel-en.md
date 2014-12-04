# Docker 1.3.2 in Stable Channel

---

Author: Alex Crawford, December 3, 2014

---


This morning we started rolling out 494.3.0 to the [stable channel](https://coreos.com/releases). Among other things, this version updates Docker to 1.3.2, following in the footsteps of our releases to the [alpha and beta channels](https://coreos.com/releases) last week.

The update will introduce the Docker `--insecure-registry` flag and, by default, require secure connections to container registries. This flag is not backwards compatible and will prevent many users from being able to deploy containers from self-run private repositories without SSL configured.

In order to temporarily ensure backward compatibility, the Docker daemon will be run with `--insecure-registry=0.0.0.0/0` by default in version 494.3.0 of CoreOS.

As a CoreOS user, this means that your insecure private registries will continue to temporarily work in the same way as before, but we plan to remove this shim and require each user to manage whether or not their machines will communicate with an insecure registry.

Before January 12th, 2015, CoreOS machines which connect to insecure registries will need to add the appropriate flags to the Docker daemon. The preferred method is via a systemd drop-in:

```
$ cat /etc/systemd/system/docker.service.d/50-insecure-registry.conf
[Service]
Environment=DOCKER_OPTS='--insecure-registry="10.0.1.0/24"'
```

This step can be automated by coreos-cloudinit by adding the following to your cloud-config:

```
#cloud-config

write_files:
  - path: /etc/systemd/system/docker.service.d/50-insecure-registry.conf
    content: |
        [Service]
        Environment=DOCKER_OPTS='--insecure-registry="10.0.1.0/24"'
```

Again, if you depend on insecure registries, you will need to update your configuration by January 12th, 2015.

As always, if you have any questions or concerns, please join us in [IRC #coreos](irc://irc.freenode.org:6667/#coreos).

---

Original source: [Docker 1.3.2 in Stable Channel](https://coreos.com/blog/docker-1-3-2-stable-channel/)