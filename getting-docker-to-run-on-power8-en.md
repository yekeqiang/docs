# Getting Docker to run on Power8

---

##### Author: Jean-Tiare Le Bigot

---

Last Week-End, I wanted to play around with [Docker on a Power8](http://en.wikipedia.org/wiki/POWER8) processor. Unfortunately, there no “ready-to-use” build available (yet) and Go support is still quite rough. Anyway, I love challenges and the process was eased a lot by the work of [Dave Cheney](http://dave.cheney.net/) from Canonical who did the hard work of [porting the go command line to Power8](http://go-talks.appspot.com/github.com/davecheney/gosyd/gccgo.slide#1).

Power8 is the name of a 64bits RISC processor micro-architecture of the same family as the G5 for example. This was the processor powering the venerable Mac G5. It is extremely parallel with up to 8 threads per core. This makes it especially good at running databases. Notably, [Stewart Smith tuned MySQL 7 to get up to 1M request per seconds](https://www.flamingspork.com/blog/2014/06/03/1-million-sql-queries-per-second-mysql-5-7-on-power8/). This is just amazing!

Docker is a tool helping developers to build, ship and run code anywhere just like containers helps shipping anything anywhere. It is increasingly used in production to cleanly isolate processes on a same physical machine without the overhead of a Virtual Machine.

So, let’s get started. My goal was to get docker running and, if possible the latest version (it turns out it actually **is** the latest version). The goal was not to make it the shiniest way. That’s for later.

Here is the state of the art:

- Docker depends on Go and cgo 1.2.1 until version 1.1.1
- Docker depends on Go and cgo 1.3+ after then
- gccgo 4.9, shipped with Ubuntu 14.04 supports go 1.2.1 but lacks some reflexivity implementation for Power8 and Elf parsing for Power8 in libcgo
- gccgo trunk supports go 1.4 (yes), fixes the reflexivity but still lacks the Elf parsing
- golang 1.3 has no support for Power8
- golang dev.power64 is still very work in progress but supports ELF parsing for Power8 (hint, hint)

As you can see, this is not attempting to square the circle but not so close.

It is also worth noting that gccgo is only the compiler parts. It brings no support for the “go” command line itself (which is written in pure go) neither for cgo (which bridges the gap between Go and C worlds). Fortunately, Dave Cheney, of Canonical, did the hard work of getting “go” to build with gccgo and in turn seamlessly work with gccgo backend by default. His work is now available through ‘apt-get’. He also did a great presentation of his work which is available online http://go-talks.appspot.com/github.com/davecheney/gosyd/gccgo.slide. And, honestly, after a full week-end battling to get it right, I totally share his opinions when he writes “ʕ╯◔ϖ◔ʔ╯︵ ┻━┻”.

Among the discarded, aborted, failed attempts: cross compile from my laptop, find ready to use instructions, use stock gcc 4.9, build dev.power64 Go branch (it’s completely broken / Work in progress), fly a unicorn.

Anyway, let’s start over. What we’ll do:

1. get a Power8 machine. No cross build sorry.
2. grab latest version of GCC from trunk (SVN, that’s 1 VCS)
3. grab latest WIP version of Power8 from dev.power64 (Mercurial, that’s a 2nd VCS)
4. copy required bits from go to gccgo, namely the ELF parser of libcgo
5. patch, build and install gccgo in /opt/gcc-trunk
6. build “go” and “cgo” commands to use our updated libgo.so.6 instead of libgo.so.5
7. grab lastest version of Docker from master (Git, that’s a 3rd VCS)
8. patch, build, install Docker
9. celebrate


### 1. Get a Power8 Machine

The easiest way to get one is to [join RunAbove’s public beta](http://labs.runabove.com/power8/) which comes with a $32 Voucher. That’s one month worth of Power8.

Common setup:

```
sudo locale-gen
sudo apt-get -y update
sudo apt-get -y install subversion mercurial git build-essential gccgo-go
```

### 2. Grab GCC

```
cd
svn checkout svn://gcc.gnu.org/svn/gcc/trunk gcc
# Be *very* patient
```

### 3. Grab Go dev.power64

```
cd
hg clone -u release https://code.google.com/p/go
cd go
hg update dev.power64
```

### 4. Patch GCC

GCC’s libcgo implementation lakes elf parsing supporting for PPC64 instruction set. As this is required by cgo, we’ll get it from Go itself.

```
cd
cp go/src/debug/elf/file.go gcc/libgo/go/debug/elf/
cp go/src/debug/elf/elf.go gcc/libgo/go/debug/elf/
```

It also lacks some termios related symbols required to build docker command line interface. They’re easily added with this patch (extracted from `svn diff`):

```
--- libgo/mksysinfo.sh  (revision 216693)
+++ libgo/mksysinfo.sh  (working copy)
@@ -174,6 +174,15 @@
 #ifdef TIOCGWINSZ
   TIOCGWINSZ_val = TIOCGWINSZ,
 #endif
+#ifdef TIOCSWINSZ
+  TIOCSWINSZ_val = TIOCSWINSZ,
+#endif
+#ifdef TCGETS
+  TCGETS_val = TCGETS,
+#endif
+#ifdef TCSETS
+  TCSETS_val = TCSETS,
+#endif
 #ifdef TIOCNOTTY
   TIOCNOTTY_val = TIOCNOTTY,
 #endif
@@ -790,6 +799,21 @@
     echo 'const TIOCGWINSZ = _TIOCGWINSZ_val' >> ${OUT}
   fi
 fi
+if ! grep '^const TIOCSWINSZ' ${OUT} >/dev/null 2>&1; then
+  if grep '^const _TIOCSWINSZ_val' ${OUT} >/dev/null 2>&1; then
+    echo 'const TIOCSWINSZ = _TIOCSWINSZ_val' >> ${OUT}
+  fi
+fi
+if ! grep '^const TCGETS' ${OUT} >/dev/null 2>&1; then
+  if grep '^const _TCGETS_val' ${OUT} >/dev/null 2>&1; then
+    echo 'const TCGETS = _TCGETS_val' >> ${OUT}
+  fi
+fi
+if ! grep '^const TCSETS' ${OUT} >/dev/null 2>&1; then
+  if grep '^const _TCSETS_val' ${OUT} >/dev/null 2>&1; then
+    echo 'const TCSETS = _TCSETS_val' >> ${OUT}
+  fi
+fi
 if ! grep '^const TIOCNOTTY' ${OUT} >/dev/null 2>&1; then
   if grep '^const _TIOCNOTTY_val' ${OUT} >/dev/null 2>&1; then
     echo 'const TIOCNOTTY = _TIOCNOTTY_val' >> ${OUT}
```

If you’re planning on making a break, just wait one more minute. We’ll launch GCC’s build…

### 5. Build GCC

As usual, except that we built it out of tree.

```
cd
mkdir build-gcc
cd build-gcc
sudo apt-get install -y libgmp-dev libmpfr-dev libmpc-dev install flex bison
../gcc/configure --enable-languages=go --disable-multilib --prefix=/opt/gcc-trunk
make -j200 # if using the big instance
sudo make install
```

Be patient, read a book, watch a movie, go visit friends… It takes a while. On the ‘S’ instance, it took me around 98 minutes.

Once done, we have some additional setup:

```
export PATH=/opt/gcc-trunk/bin:$PATH
echo "/opt/gcc-trunk/lib64" | sudo tee /etc/ld.so.conf.d/gcc-trunk.conf
sudo ldconfig
```

### 6. Build (and install) CGO

Cgo is the component bridging the gap between Go and C world. It is notably required to build the devmapper driver of Docker.

```
cd go/src/cmd/cgo
go build
```

You can now install it. It’s hackish but it does the job. But I still can’t figure out why I needed to copy the source files to `/usr/src/cmd/cgo`. Anyway, it’s working.

```
sudo mkdir -p /usr/pkg/tool/linux_ppc64
sudo mkdir -p /usr/src/cmd/cgo
sudo cp cgo /usr/pkg/tool/linux_ppc64/cgo
sudo cp * /usr/src/cmd/cgo
```

One more thing: to let `go build` know we prepared to using cgo, we need to switch `CGO_ENABLED` environment variable on.

```
export CGO_ENABLED=1
```

### 7. Grab Docker 1.3.0

This is the last stable release at the time of writing. Let’s use it.

```
cd
git clone
cd docker
git checkout v1.3.0
```

We’ll also need to prepare a little the build environment:

```
mkdir /go
sudo mkdir -p /go/src/github.com/docker/
sudo ln -s $HOME/docker /go/src/github.com/docker/docker
export PATH=/opt/gcc-trunk/bin/:$PATH
export GOPATH=/go:/go/src/github.com/docker/docker/vendor
```

### 8. Build Docker

Just issue ‘docker build’. I’m kidding.

This is the trickiest part of the job as all the full build systems assumes a working docker environment. So we’ll mostly emulate it.

First, let’s apply a couple of patches.

Remove a runtime (?!) check preventing Docker to run on non amd64 platforms:

````
diff --git a/daemon/daemon.go b/daemon/daemon.go
index 235788c..b75a94e 100644
--- a/daemon/daemon.go
+++ b/daemon/daemon.go
@@ -1104,9 +1104,9 @@ func (daemon *Daemon) ImageGetCached(imgID string, config *runconfig.Config) (*i
  
 func checkKernelAndArch() error {
    // Check for unsupported architectures
-   if runtime.GOARCH != "amd64" {
-       return fmt.Errorf("The Docker runtime currently only supports amd64 (not %s). This will change in the future. Aborting.", runtime.GOARCH)
-   }
+   //if runtime.GOARCH != "amd64" {
+   //  return fmt.Errorf("The Docker runtime currently only supports amd64 (not %s). This will change in the future. Aborting.", runtime.GOARCH)
+   //}
    // Check for unsupported kernel versions
    // FIXME: it would be cleaner to not test for specific versions, but rather
    // test for specific functionalities.
```

Next, we need to workaround hard-coded references to official go compiler:

```
diff --git a/vendor/src/github.com/kr/pty/pty_linux.go b/vendor/src/github.com/kr/pty/pty_linux.go
index 6e5a042..8525f80 100644
--- a/vendor/src/github.com/kr/pty/pty_linux.go
+++ b/vendor/src/github.com/kr/pty/pty_linux.go
@@ -7,6 +7,11 @@ import (
    "unsafe"
 )
  
+type (
+        _C_int  int32
+        _C_uint uint32
+)
+
 var (
    ioctl_TIOCGPTN   = _IOR('T', 0x30, unsafe.Sizeof(_C_uint(0))) /* Get Pty Number (of pty-mux device) */
    ioctl_TIOCSPTLCK = _IOW('T', 0x31, unsafe.Sizeof(_C_int(0)))  /* Lock/unlock Pty */
```

And, finally, change the link flags. Note that for some reason `-static` breaks network communication. It seems to be related to name resolution but I did not investigate further as dynamic linking works just fine.

```
diff --git a/hack/make/binary b/hack/make/binary
index b97069a..f5398ae 100755
--- a/hack/make/binary
+++ b/hack/make/binary
@@ -6,9 +6,8 @@ DEST=$1
 go build \
    -o "$DEST/docker-$VERSION" \
    "${BUILDFLAGS[@]}" \
-   -ldflags "
-       $LDFLAGS
-       $LDFLAGS_STATIC_DOCKER
+   -gccgoflags "
+       -static-libgo -static-libgcc
    " \
    ./docker
 echo "Created binary: $DEST/docker-$VERSION"
```

Make sure you have the the ldconfig, PATH and CGO_ENABLED tricks then:

```
./hack/make.sh binary
sudo cp /home/admin/docker/bundles/1.3.0/binary/docker-1.3.0 /usr/bin/docker
```

And we’re done !

---

Original source: [Getting Docker to run on Power8](https://blog.jtlebi.fr/2014/10/28/getting-docker-to-run-on-power8/)