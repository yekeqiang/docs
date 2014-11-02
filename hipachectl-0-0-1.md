
# hipachectl 0.0.1 - CLI tool to manage hipache

---

##### Author: James Mills

---

Hi,

I'm pleased to announce the release of hipachectl 0.0.1

## What is hipache?

hipache is dotCloud's high performance reverse proxy and load balancer.

See: https://github.com/hipache/hipache

## What is hipachectl?


hipachectl is a Command Line Tool to help manage hipache.

hipachectl allows you to:

- Add virtualhosts
- List virtualhosts
- Delete virtualhosts

You can use hipachectl as a Docker container too! If run as a Docker cintainer and the hipache redis container linked as "redis" hipachectl will use this as the host to connect to and manage. e.g: docker run -i -t --link hipache:redis prologic/hipachectl ...

Links:

- PyPi Page: [https://pypi.python.org/pypi/hipachectl](https://pypi.python.org/pypi/hipachectl)
- Docker Image: [https://registry.hub.docker.com/u/prologic/hipachectl/](https://registry.hub.docker.com/u/prologic/hipachectl/)
- Source Code: [https://bitbucket.org/prologic/hipachectl](https://bitbucket.org/prologic/hipachectl)

Feedback welcome as well as contributions!

cheers
James
