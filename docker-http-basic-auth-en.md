# Securing Docker with HTTP Basic Authentication

---

##### Author: Ahmet Alp Balkan

---

I recently needed to secure my Docker host instance simply with a basic username and password authentication as I mostly find the certificate creation steps tedious. Docker has no built-in username/password authentication support so I thought I could have a HTTP proxy server which asks for a password on top of Docker Remote API server. Below you will find how you can secure your Docker host using username and password, namely HTTP [Basic Authentication](http://en.wikipedia.org/wiki/Basic_access_authentication).

By default, your Docker installation uses a Unix socket binded to localhost if you installed Docker by yourself and that is not open to access from outside world (read more about [Docker Security](https://docs.docker.com/articles/security/)). By exposing an endpoint to the outside world, you will increase the attack surface (something not good), if you are not extensively worried about it.

> **Caution: HTTP Basic Authentication uses username/password credentials transferred without any encryption. If you are serious about security, please [secure your Docker host using TLS/SSL](https://docs.docker.com/articles/https/).**

Hope you find it useful. Here we go:

## Step 1: Install nginx

First, `ssh` into the Docker host machine you have. Installing nginx on Debian/Ubuntu is as simple as:

```
sudo apt-get install nginx
```

## Step 2: Create passsword file

This file will contain your access to Docker instance.

```
sudo htpasswd -c /etc/nginx/.htpasswd YOUR_USERNAME
```

Running this will prompt you a password, pick a password and keep that in mind.

## Step 3: Configure nginx

In order to access Docker API running on the Unix socket locally, nginx process must have root privileges. Therefore edit `/etc/nginx/nginx.conf` with `user` configuration changed to `root` user.

The following command do it for you:

```
sudo sed -i 's/user .*;/user root;/' /etc/nginx/nginx.conf
```

## Step 4: Add Docker site

Now we need to expose the port 4243 as a proxy for the Unix socket running locally with HTTP Basic Authentication. The following command will create the nginx site for you:

```
sudo tee /etc/nginx/sites-enabled/docker <<EOF 
upstream docker {
  server unix:/var/run/docker.sock;
}

server {
  listen 4242 default_server;
  location / {
    proxy_pass http://docker;
    auth_basic_user_file /etc/nginx/.htpasswd;
    auth_basic "Access restricted";
  }
}
EOF
```

## Step 5: Restart nginx

```
sudo service nginx restart
```
You must not see any errors at this step.

## Step 6: Profit!

If all has gone well, you can now use your Docker instance with HTTP Basic Authentication with the username/password you defined earlier.

You should be able to access the Docker REST API running on your host using the following while you’re still in `ssh`:

```
curl -i http://username:password@localhost:4242/info
```

Alternatively, you can use:

```
curl -i http://username:password@YourDockerHost.com:4242/info
```

to verify if your basic auth works file. You can now even use your browser to verify if this works.

## Known issues

Docker client currently does not accept URLs with basic authentication credentials (e.g. tcp://user:pass@host.com:4243). I have an [outstanding proposal](https://github.com/docker/docker/issues/8283) for this.

> Note: My [Docker.DotNet](https://ahmetalpbalkan.com/blog/docker-dotnet/) library supports HTTP basic authentication thanks to [Andreas Bieber](https://github.com/ahmetalpbalkan/Docker.DotNet/pull/3)!

---

Original source: [Securing Docker with HTTP Basic Authentication](https://ahmetalpbalkan.com/blog/docker-http-basic-auth/)