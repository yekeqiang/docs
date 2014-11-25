# Using Docker to Encapsulate Complicated Program is Successful

---

Author: Brewster Kahle

---

The Internet Archive has been using [docker](https://docker.com/) in a useful way that is a bit out of the mainstream: to package a command-line binary and its dependencies so we can deploy it on a cluster and use it in the same way we would a static binary.

Columbia University’s [Daniel Ellis](http://www.ee.columbia.edu/~dpwe/) created an [audio fingerprinting](http://labrosa.ee.columbia.edu/matlab/audfprint/) program that was used in [a competition](http://www.music-ir.org/mirex/wiki/2014:MIREX2014_Results). It was not packaged as a debian package or other distribution approach. It took a while for our staff to find how to install it and its many dependencies consistently on Ubuntu, but it seemed pretty heavy handed to install that on our worker cluster. So we explored using docker and it has been successful. While old hand for some, I thought it might be interesting to explain what we did.

1) Created a docker file to make a [docker container](https://registry.hub.docker.com/u/rajbot/audfprint/) that held all of the code needed to run the system.

2) Worked with our systems group to figure out how to install docker on our cluster with a security profile we felt comfortable with. This included running the binary in the container as user nobody.

3) Ramped up slowly to test the downloading and running of this container. In general it would take 10-25 minutes to download the container the first time. Once cached on a worker node, it was very fast to start up. This cache is persistent between many jobs, so this is efficient.

4) Use the container as we would a shell command, but passed files into the container by mounting a sub filesystem for it to read and write to.   Also helped with signaling errors.

5) Starting production use now.

We hope that docker can help us with other programs that require complicated or legacy environments to run.

Congratulations to Raj Kumar, Aaron Ximm, and Andy Bezella for the creative solution to problem that could have made it difficult for us to use some complicated academic code in our production environment.

Go docker!

---

Original source: [Using Docker to Encapsulate Complicated Program is Successful](http://blog.archive.org/2014/11/14/docker-to-encapsulate-complicated-program-successful/)