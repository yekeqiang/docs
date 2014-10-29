# Easily transfer docker images between two machines over the network

---

##### Author: Dustin Spicuzza

---

I’ve been using docker a lot, and on occasion I need to transfer images between two machines that are on a local network. If a particular image is large, I might not want to download it twice from two machines, so I download it on one machine and transfer it to the other over the local network.

Now, I could stand up a local docker registry and use that, but it’s a bit of work. Instead, I’ve found that the quickest and easiest solution is to combine the docker ‘save’ and ‘load’ commands with a bit of netcat magic, and it’s pretty fast and easy. (**Update**: you can do it easily using SSH too, see the end of the post). Check it out.

First, on the destination machine (make sure your firewall allows traffic to the specified port, in this case 1234):

```
nc -v -l 1234 | docker load
```

Next, on the source machine, transfer the image (virtuald/etcd:0.4.6) to the destination IP (192.168.0.42):

```
docker save virtuald/etcd:0.4.6 | nc -v 192.168.0.42 1234
```

And that’s it!

The sad thing is that docker save/load doesn’t show a status message when saving/loading, so it might look like it’s not doing anything. However, using the -v flag for netcat shows when the connection is successfully opened/closed, so that’s something.

**Security warning**: Obviously, running netcat like this is a **huge**  security hole while its up and listening, as anyone who can connect to the port can upload arbitrary images into your docker registry. This is mitigated a bit since netcat will immediately disconnect after the first client disconnects, but still risky on an untrusted network. ***Only use this on trusted networks***!

**Note**: due to [this bug](https://github.com/docker/docker/issues/3877), you’ll want to be using docker 1.2+, otherwise you may get unexpected results.

**Update**! As [Joshua Barratt](http://www.virtualroadside.com/blog/index.php/2014/09/29/easily-transfer-docker-images-between-two-machines-over-the-network/comment-page-1/#comment-170185) points out, since this method generalizes to any transport that allows piping via stdin/stdout, you can also do the transfer via SSH too, which is certainly more secure. Use the -C option to enable compression for faster transfers (thanks [Andreas Steffan](http://www.virtualroadside.com/blog/index.php/2014/09/29/easily-transfer-docker-images-between-two-machines-over-the-network/comment-page-1/#comment-171587)).

```
docker save virtuald/etcd:0.4.6 | ssh -C 192.168.0.42 ‘docker load’
```

**Update II**: As a number of people have pointed out, you can use PV to show a status message:

```
docker save virtuald/etcd:0.4.6 | pv | ssh -C 192.168.0.42 'docker load'
```

---

Original source: [Easily transfer docker images between two machines over the network](http://www.virtualroadside.com/blog/index.php/2014/09/29/easily-transfer-docker-images-between-two-machines-over-the-network/)