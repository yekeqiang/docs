# Speeding up CPAN module contributions using the Docker language stack images

---

##### Author: Sven Dowidei

---

Docker Inc. just released our first set of programming language images on the Docker Hub. They cover c/c++ (gcc), clojure, go (golang), hy (hylang), java, node, perl, php, python, rails, and ruby.

As I need to do some work on API testing when I come back from holidays, I thought I’d look at the Net:Docker CPAN module – and of course, there is no Perl on my Boot2Docker image, so its a perfect opportunity to see what I should do.

After forking and cloning the Git repository, I created the following initial Dockerfile:

```
FROM perl:5.20
MAINTAINER Sven Dowideit

COPY . /docker-perl
WORKDIR /docker-perl

RUN cpanm --installdeps .
RUN perl Build.PL
RUN ./Build build
RUN ./Build test
```

It fails to build during the ‘test’ step:

```
$ docker build -t docker-perl .

... snip ...

Step 6 : RUN ./Build test
---> Running in 367afe04c77e
Can't open socket var/run/docker.sock: No such file or directory at /usr/local/lib/perl5/site_perl/5.20.0/LWP/Protocol/http/SocketUnixAlt.pm line 27. at t/docker-api.t line 9.
# Tests were run but no plan was declared and done_testing() was not seen.
# Looks like your test exited with 255 just after 1.
t/docker-api.t ....
Dubious, test returned 255 (wstat 65280, 0xff00)
All 1 subtests passed
Can't locate IO/String.pm in @INC (you may need to install the IO::String module) (@INC contains: /docker-perl/blib/arch /docker-perl/blib/lib /usr/local/lib/perl5/site_perl/5.20.0/x86_64-linux /usr/local/lib/perl5/site_perl/5.20.0 /usr/local/lib/perl5/5.20.0/x86_64-linux /usr/local/lib/perl5/5.20.0 .) at t/docker-start.t line 3.
BEGIN failed--compilation aborted at t/docker-start.t line 3.
t/docker-start.t ..
Dubious, test returned 2 (wstat 512, 0x200)
No subtests run

Test Summary Report
-------------------
t/docker-api.t (Wstat: 65280 Tests: 1 Failed: 0)
Non-zero exit status: 255
Parse errors: No plan found in TAP output
t/docker-start.t (Wstat: 512 Tests: 0 Failed: 0)
Non-zero exit status: 2
Parse errors: No plan found in TAP output
Files=2, Tests=1, 0 wallclock secs ( 0.02 usr 0.00 sys + 0.21 cusr 0.03 csys = 0.26 CPU)
Result: FAIL
2014/09/26 16:08:19 The command [/bin/sh -c ./Build test] returned a non-zero code: 1
```

I’m going to have to give this Dockerfile a DOCKER_HOST (incorrectly using http://) setting (to one of my insecure plain text tcp based servers :), and add IO::String and JSON:XS to the cpanfile.

Unfortunately, because `cpanm --installdeps .` uses the files in the build context, this way does not use the build cache – so its slow. Its worth duplicating the contents of the cpanfile before the COPY instruction for speed.

So the working Dockerfile looks like:

```
FROM perl:5.20
MAINTAINER Sven Dowideit

RUN cpanm Module::Build::Tiny
RUN cpanm Moo
#', '1.002000';
RUN cpanm JSON
RUN cpanm JSON::XS
RUN cpanm LWP::UserAgent
RUN cpanm LWP::Protocol::http::SocketUnixAlt
RUN cpanm URI
RUN cpanm AnyEvent
RUN cpanm AnyEvent::HTTP
RUN cpanm IO::String

COPY . /docker-perl
WORKDIR /docker-perl

RUN cpanm --installdeps .
RUN perl Build.PL
RUN ./Build build

# This is a terrible cheat.
ENV DOCKER_HOST http://10.10.10.4:2375

RUN ./Build test
RUN ./Build install

CMD ["docker.pl", "ps"]
```

and then `docker build -t docker-perl .` results in:

```
bash-3.2$ docker build -t docker-perl .
Sending build context to Docker daemon 138.8 kB
Sending build context to Docker daemon
Step 0 : FROM perl:5.20
---> 4d4674548e76
Step 1 : MAINTAINER Sven Dowideit 
---> Using cache
---> 4ad0946e76aa
Step 2 : RUN cpanm Module::Build::Tiny
---> Using cache
---> f1b94d36a51c
Step 3 : RUN cpanm Moo
---> Using cache
---> 98de8c3a19a8
Step 4 : RUN cpanm JSON
---> Using cache
---> 73debd4ee367
Step 5 : RUN cpanm JSON::XS
---> Using cache
---> 89378a425f0b
Step 6 : RUN cpanm LWP::UserAgent
---> Using cache
---> 252fe329cf22
Step 7 : RUN cpanm LWP::Protocol::http::SocketUnixAlt
---> Using cache
---> a77d289faf19
Step 8 : RUN cpanm URI
---> Using cache
---> 6804b418778d
Step 9 : RUN cpanm AnyEvent
---> Using cache
---> c595f66bcf73
Step 10 : RUN cpanm AnyEvent::HTTP
---> Using cache
---> 31b25b2da3c4
Step 11 : RUN cpanm IO::String
---> Using cache
---> e54cd3d01988
Step 12 : COPY . /docker-perl
---> 4d4801209a79
Removing intermediate container c42897136186
Step 13 : WORKDIR /docker-perl
---> Running in 36575a59e465
---> 7042c67cf1b7
Removing intermediate container 36575a59e465
Step 14 : RUN cpanm --installdeps .
---> Running in c1b5cbb75c4a
--> Working on .
Configuring Net-Docker-0.002005 ... OK
<== Installed dependencies for .. Finishing.
---> 071f9caca472
Removing intermediate container c1b5cbb75c4a
Step 15 : RUN perl Build.PL
---> Running in fae9bbce142f
Creating new 'Build' script for 'Net-Docker' version '0.002005'
---> 2800182bd0ff
Removing intermediate container fae9bbce142f
Step 16 : RUN ./Build build
---> Running in a98cb6c7a808
cp lib/Net/Docker.pm blib/lib/Net/Docker.pm
cp script/docker.pl blib/script/docker.pl
---> f5ba5be85f9d
Removing intermediate container a98cb6c7a808
Step 17 : ENV DOCKER_HOST http://10.10.10.4:2375
---> Running in 1e8b3273974c
---> fffb42d69011
Removing intermediate container 1e8b3273974c
Step 18 : RUN ./Build test
---> Running in 3baacccbf17e
t/docker-api.t .... ok
t/docker-start.t .. ok
All tests successful.
Files=2, Tests=41, 5 wallclock secs ( 0.02 usr 0.02 sys + 0.26 cusr 0.06 csys = 0.36 CPU)
Result: PASS
---> f5d371cdc1fa
Removing intermediate container 3baacccbf17e
Step 19 : RUN ./Build install
---> Running in 60cd90714e02
Installing /usr/local/lib/perl5/site_perl/5.20.0/Net/Docker.pm
Installing /usr/local/bin/docker.pl
---> 62c6368a2fb0
Removing intermediate container 60cd90714e02
Step 20 : CMD ["docker.pl", "ps"]
---> Running in cb5ade11e146
---> 94984ed5756d
Removing intermediate container cb5ade11e146
Successfully built 94984ed5756d
```

So that I can use it:

```
bash-3.2$ docker run --rm -it docker-perl
ID IMAGE COMMAND CREATED STATUS PORTS
e619112eae2f 10.10.10.2:5001/sve bash 1411104597 Up 7 days ARRAY(0x2b84a48)
363ec1c45841 10.10.10.2:5001/sve bash 1411104470 Up 7 days ARRAY(0x29bae20)
```

You can also run the container with `bash – docker run --rm -it docker-perl bash` so you can do some more testing, or try out more complex examples.

In this case, the `./Build test` step probably needs to happen in the `docker run` phase, as it needs access to a working Docker daemon – this issue will be true for modules that talk to external resources.

I’ve made a [pull request](https://github.com/pstuifzand/docker-perl/pull/8) for the tiny changes to get me this far. Perhaps Dockerfiles like this could be a gateway into the world of contributing quick fixes for open source libraries.

---

Original source: [Speeding up CPAN module contributions using the Docker language stack images](http://fosiki.com/blog/2014/09/26/cpan-contributions-docker-perl-language-stack-image/)