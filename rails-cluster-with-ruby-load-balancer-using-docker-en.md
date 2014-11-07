# Rails cluster with Ruby load balancer using Docker

---

##### Author: Muriel Salvan

---

Lately I discovered [Docker](http://www.docker.io/).

I made a quick presentation on it for the [rivierarb meetup of february 2014](http://rivierarb.fr/2014/02/04/Drinkup/) ([slides available](https://speakerdeck.com/murielsalvan/ruby-and-docker-on-rails)).

As an example of Docker’s usage, I decided to make something cool: a Rails cluster, with a Ruby load balancer on top of it. I wanted to simulate a whole cluster of different machines (with their own IP and port) running the same Rails server, and an extra Ruby process that acts as a proxy to load balance the requests among the Rails servers.

And Docker made this simulation available in a single computer, very easily setup.


![alt](http://resource.docker.cn/setup.png)


Here is how I did it.
Source files used can be seen in my [rails-cluster-docker project on Github](https://github.com/Muriel-Salvan/rails-cluster-docker).

## Create Docker images

First step was to create the Docker images: 1 for the Rails server, and 1 for the ruby proxy.

1. I began by creating a Trusted Docker build based on my [Github project docker-ruby](https://github.com/Muriel-Salvan/docker-ruby). This made the image `[murielsalvan/ruby](https://index.docker.io/u/murielsalvan/ruby/)` available, containing an Ubuntu Precise image with Ruby 2.1.0p0 installed.

```
docker pull murielsalvan/ruby
```

2. I then used this `murielsalvan/ruby` image to create a Docker container running a simple bash shell. I used it to install Rails and create a Rails app with a single page outputting its hostname and IP (this way each Rails server from my cluster will output different values). The source code of the Rails app can be found [here](https://github.com/Muriel-Salvan/rails-cluster-docker/tree/master/server). I then committed this container into a new image called `murielsalvan/server`, running the Rails server as a startup command, and opening port 3000.

```
docker run -t -i murielsalvan/ruby bash
docker commit -m="Test server" -author="Muriel Salvan <muriel@x-aeon.com>" -run='{"WorkingDir": "/root/server/", "Cmd": ["rails", "s"], "PortSpecs": ["3000"]}' acf566f7d155 murielsalvan/server
```

3. I created a second container from `murielsalvan/ruby` with a bash shell to install a Ruby proxy (I used [em-proxy](https://github.com/igrigorik/em-proxy): check it out, it is awesome!). The Ruby proxy takes a list of IP addresses as input, and will setup a load balancer (random strategy) among all those IPs on port 3000 (source code [here](https://github.com/Muriel-Salvan/rails-cluster-docker/tree/master/proxy)). I then committed this container into a new image: `murielsalvan/proxy`, also opening port 3000.

```
docker run -t -i murielsalvan/ruby bash
docker commit -m="Proxy server" -author="Muriel Salvan <muriel@x-aeon.com>" -run='{"PortSpecs": ["3000"]}' 7d2431c16b14 murielsalvan/proxy
```

## Run the Rails cluster

Once images are available, all we have to do is to run them in containers.

I wrote a small Ruby program to launch N Rails server’s containers (N being given as an argument) and outputting their IP once they are listening to their port 3000.

This script also binds the N container ports 3000 to host ports 5000+i. This way it is easy to check that each Rails container is working correctly by issuing `wget -S -O - http://localhost:5000` commands to target each one of them and make sure they behave correctly without any proxy in front.

run_cluster.rb

```
nbr_servers = ARGV[0].to_i
 
pipes_in = {}
nbr_servers.times do |idx|
  port = 5000 + idx
  pipe_cmd_in, pipe_cmd_out = IO.pipe
  cmd_pid = Process.spawn("docker run -p #{port}:3000 murielsalvan/server", :out => pipe_cmd_out, :err => pipe_cmd_out)
  puts "Launch server on port #{port}: PID=#{cmd_pid}"
  Process.detach(cmd_pid)
  pipe_cmd_out.close
  pipes_in[cmd_pid] = pipe_cmd_in
end
# Wait for all servers to be up
pipes_in.each do |pid, pipe_in|
  puts "Waiting for PID #{pid} to be listening..."
  found_info = false
  while !found_info
    out = pipe_in.readline.chomp
    puts out
    found_info = out.match(/WEBrick::HTTPServer/) != nil
    sleep 0.01 if !found_info
  end
end
 
puts 'All servers up and running.'
 
# Get their IP addresses
ips = []
`docker ps | sed -e 's/^\\(............\\).*$/\\1/' | tail -#{nbr_servers}`.split("\n").each do |container_id|
  ips << `docker inspect #{container_id} | grep IPAddress | sed -e 's/.*: \\"\\(.*\\)\\".*/\\1/g'`.chomp
end
 
puts ips.join(' ')
```

Here is the output obtained. Please note the IPs output at the end: those will be given to the proxy in the next step.

```
> ruby -w run_cluster.rb 5
Launch server on port 5000: PID=6559
Launch server on port 5001: PID=6561
Launch server on port 5002: PID=6565
Launch server on port 5003: PID=6571
Launch server on port 5004: PID=6573
Waiting for PID 6559 to be listening...
[2014-02-05 18:19:44] INFO  WEBrick 1.3.1
[2014-02-05 18:19:44] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:44] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6561 to be listening...
[2014-02-05 18:19:42] INFO  WEBrick 1.3.1
[2014-02-05 18:19:42] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:42] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6565 to be listening...
[2014-02-05 18:19:44] INFO  WEBrick 1.3.1
[2014-02-05 18:19:44] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:44] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6571 to be listening...
[2014-02-05 18:19:43] INFO  WEBrick 1.3.1
[2014-02-05 18:19:43] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:43] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
Waiting for PID 6573 to be listening...
[2014-02-05 18:19:41] INFO  WEBrick 1.3.1
[2014-02-05 18:19:41] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-05 18:19:41] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
All servers up and running.
172.17.0.16 172.17.0.14 172.17.0.15 172.17.0.13 172.17.0.12
```

## Run the Ruby proxy

Here again, a small Ruby program can help us:

run_proxy.rb

```
lst_ips = ARGV.clone
Process.wait(Process.spawn("docker run -p 3000:3000 -t murielsalvan/proxy ruby -w /root/run_proxy.rb #{lst_ips.join(' ')}"))
```

Here is the output:

```
> ruby -w run_proxy.rb 172.17.0.16 172.17.0.14 172.17.0.15 172.17.0.13 172.17.0.12
/root/run_proxy.rb:149: warning: `&' interpreted as argument prefix
/root/run_proxy.rb:150: warning: `&' interpreted as argument prefix
/root/run_proxy.rb:151: warning: `&' interpreted as argument prefix
/root/run_proxy.rb:152: warning: `&' interpreted as argument prefix
/usr/local/lib/ruby/gems/2.1.0/gems/em-proxy-0.1.8/lib/em-proxy/backend.rb:37: warning: method redefined; discarding old debug
/usr/local/lib/ruby/gems/2.1.0/gems/em-proxy-0.1.8/lib/em-proxy/connection.rb:126: warning: method redefined; discarding old debug
/root/run_proxy.rb:168: warning: method redefined; discarding old stop
/usr/local/lib/ruby/gems/2.1.0/gems/em-proxy-0.1.8/lib/em-proxy/proxy.rb:17: warning: previous definition of stop was here
Launching proxy at 0.0.0.0:3000...
```

And now our proxy is listening to port 3000 (with tons of warnings… maybe em-proxy would need some clean-up ;-) ).

## Unleash the requests!

Time to jump on our browser and target http://localhost:3000 to see if our requests target different hostnames and IPs. Don’t forget to clear the cache between your requests to make sure they are sent to your proxy.

Here is the output of 2 requests: we can see clearly that 2 different Rails instances were targeted.

![alt](http://resource.docker.cn/test1.png)

Aaaaaand… REFRESH!

![alt](http://resource.docker.cn/test2.png)

Hooray! A simple setup giving a complete Rails clustering solution, tested locally on our host.

Personnally, I managed to make the whole run on an Ubuntu 14.04 Alpha inside a VirtualBox from my Windows 7 64b host, and performance was quite acceptable (less than 1 sec per request)!

Enjoy

Muriel

Note: You can [read this article in Chinese](http://liubin.org/2014/02/18/rails-cluster-with-ruby-load-balancer-using-docker/) thanks to [Liu Bin](http://liubin.org/about/)!

---

Original source: [Rails cluster with Ruby load balancer using Docker](http://blog.x-aeon.com/2014/02/06/rails-cluster-with-ruby-load-balancer-using-docker/)


