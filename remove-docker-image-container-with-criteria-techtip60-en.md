# Remove Docker image and container with a criteria (Tech Tip #60)

---

Author: Arun Gupta

---

You have installed multiple Docker images and would like to clean them up using rmi command. So, you list all the images as:


```
~> docker images --no-trunc
REPOSITORY                 TAG                 IMAGE ID                                                           CREATED             VIRTUAL SIZE
mysql                      latest              98840bbb442c7dc3640ffe3a8ec45d3fee934c2f6d85daaaa2edf65b380485a0   25 hours ago        236 MB
wildfly-centos             latest              fc378232f03d04bff96987f4c23969461582f73c3a7b473a7cb823ac67939f48   5 days ago          619.6 MB
arungupta/wildfly-centos   latest              e4f1dbdff18956621aa48a83e5b05df309ee002c3668fa452f1235465d881020   6 days ago          619.6 MB
wildfly-ubuntu             latest              a2e96e76eb10f4df87d01965ce4df5310de6f9f3927aceb7f5642393050e8752   7 days ago          749.5 MB
registry                   latest              7e2db37c6564bf030e6c5af9725bf9f9a8196846e3a77a51e201fc97871e2e60   2 weeks ago         411.6 MB
centos                     latest              ae0c2d0bdc100993f7093400f96e9abab6ddd9a7c56b0ceba47685df5a8fe906   4 weeks ago         224 MB
jboss/wildfly              latest              365390553f925f96f8c00f79525ad101847de7781bb4fec23b1188f25fe99a6a   5 weeks ago         948.7 MB
centos/wildfly             latest              1de9304f58bbc2d401b4dcbba6fc686bdd6f6bff473fe486e7cb905c02163b1a   6 weeks ago         606.6 MB
```
 
Then try to remove the “arungupta/wildfly-centos” image as shown below, but get an error:


```
~> docker rmi e4f1dbdff18956621aa48a83e5b05df309ee002c3668fa452f1235465d881020
Error response from daemon: Conflict, cannot delete e4f1dbdff189 because the container bafc2b3327a4 is using it, use -f to force
2014/12/02 12:56:53 Error: failed to remove one or more images
```
 
So you follow the recommendation of using -f switch but get another error:


```
~> docker rmi -f e4f1dbdff18956621aa48a83e5b05df309ee002c3668fa452f1235465d881020
Error response from daemon: No such id: c345720579e024df4f6d28d2062fda64b7743f7dbb214136d4d2285bc3afc95b
2014/12/02 12:56:55 Error: failed to remove one or more images
```
 
What do you do ?

This message indicates that the image is used by one of the containers and that’s why could not be removed. The error message is very ambiguous and a [#9458](https://github.com/docker/docker/issues/9458) has been filed for the same.

In the meanwhile, an easy way to solve this is to list all the containers as shown:


```
CONTAINER ID        IMAGE                             COMMAND                CREATED             STATUS                      PORTS                NAMES
bafc2b3327a4        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   4 days ago                                                           boring_ptolemy        
bfe71d92a612        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   4 days ago                                                           agitated_einstein     
e1c0965d202c        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   4 days ago                                                           thirsty_blackwell     
ddc400c26f1a        mysql:latest                      "/entrypoint.sh mysq   5 days ago          Exited (0) 27 minutes ago   3306/tcp             sample-mysql          
05c741b5e22f        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago          Exited (130) 5 days ago                          agitated_lalande      
ff10b83d6c17        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   5 days ago                                                           insane_wilson         
b2774b17460c        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   5 days ago                                                           goofy_pasteur         
2d64f4eb8fb9        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   5 days ago                                                           focused_lalande       
c3f61947671a        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   5 days ago                                                           silly_ardinghelli     
ac6f29b92c7a        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   5 days ago                                                           stoic_leakey          
fc16f3f8c139        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago                                                           desperate_babbage     
4555628a5d0a        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago          Exited (-1) 4 days ago                           sharp_bardeen         
3bdae1d2527a        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago          Exited (130) 5 days ago                          sick_lovelace         
2697c769c2ee        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago                                                           thirsty_fermat        
f8c686d1d6be        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago          Exited (130) 5 days ago                          cranky_fermat         
a1945f2ca473        wildfly-centos:latest             "/opt/jboss/wildfly/   5 days ago          Exited (-1) 4 days ago                           suspicious_turing     
31b9c4df0633        arungupta/wildfly-centos:latest   "/opt/jboss/wildfly/   5 days ago                                                           distracted_franklin   
cd8dad2b1e22        c345720579e0                      "/bin/sh -c '#(nop)    5 days ago                                                           cocky_blackwell       
```
 
There are lots of containers that are using “arungupta/wildfly-centos” image but none of them seem to be running. If there are any containers that are running then you need to stop them as:


```
docker rm $(docker stop $(docker ps -q))
```
 
Remove the containers that are using this image as:


```
docker ps -a | grep arungupta/wildfly-centos | awk '{print $1}' | xargs docker rm
bafc2b3327a4
bfe71d92a612
e1c0965d202c
ff10b83d6c17
b2774b17460c
2d64f4eb8fb9
ac6f29b92c7a
31b9c4df0633
```
 
The criteria here is specified as a grep pattern.

[docker ps](https://docs.docker.com/reference/commandline/cli/#ps) command has other options to specify criteria as well such as only the latest created containers or containers in a particular status. For example, containers that exited with status -1 can be seen as:


```
~> docker ps -a -f "exited=-1"
CONTAINER ID        IMAGE                   COMMAND                CREATED             STATUS                       PORTS               NAMES
68aca76aa690        wildfly-centos:latest   "/opt/jboss/wildfly/   39 minutes ago      Exited (-1) 37 minutes ago                       insane_yonath
```
 
All running containers, as opposed to meeting a specific criteria, can be removed as:

```
docker rm $(docker ps -aq)
```
 
And now the image can be easily removed as

```
~> docker rmi e4f1dbdff189
Untagged: arungupta/wildfly-centos:latest
Deleted: e4f1dbdff18956621aa48a83e5b05df309ee002c3668fa452f1235465d881020
Deleted: ad2899e176a2e73acbcf61909426786eaa195fcea7fb0aa27061431a3aae6633
```
 
Just like removing all containers, all images can be removed as:


```
docker rmi $(docker images -q)
```
 
Enjoy!

---

Original source: [Remove Docker image and container with a criteria (Tech Tip #60)](http://blog.arungupta.me/2014/12/remove-docker-image-container-with-criteria-techtip60/)