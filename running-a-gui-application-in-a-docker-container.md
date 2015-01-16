# Running a GUI application in a Docker container

---

##### Author: Johan Seferidis

---

This guide will show you how to run a GUI application **headless** in a Docker container and even more specific scenarios involving running Firefox and Chrome. If you are not interested about those then you can just stop in the middle of this tutorial.

## What the hell is X?

X is a program that sits on a Linux machine with a monitor. X’s job is to talk to the Linux kernel in behalf of GUI programs. So if you are playing a game for example, the game (that is, the application) is constantly sending drawing commands to the X server like “draw me a rectangle here”. X forwards all this to the Kernel which will further forward the information to the GPU to render it on the monitor.
X can even receive commands from the keyboard or mouse. When you click to shoot on your game for example, the command “click at 466,333″ is sent from your mouse to the kernel, from the kernel to the X and from X to the game. That way the game can have a clue on what is happening!
You will often hear X being called a server and the reason for that is simply because the way the applications send commands to X is through sockets. For that reason the applications are also referred to as clients many times.
If you are reading this then the X is running on your PC. Let’s prove it:

```
> ps aux | grep X
root      1436  3.2  0.7 687868 94444 tty7     Ssl+ 09:47   2:50 /usr/bin/X -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
```

We can see that X is running as root and has PID 1436. An other important thing is to notice the :0 which is called display in X jargon. A display is essentially:

- A monitor
- A mouse
- A keyboard
- 
And this is the bigger picture of how it all looks together:

![alt](http://resource.docker.cn/user-workstation.png)

Now there is a variable in Linux that is used whenever we run a GUI program. That variable is surprisingly called DISPLAY. The syntax of the DISPLAY variable is

```
<hostname>:<display>.<monitor>
```

. Let’s check the DISPLAY on our computer:

```
> echo $DISPLAY
:0
```

I get :0, which means we use display 0. Notice however that this says nothing about which monitor we use. This makes sense since if you are running 2 or more monitors on your Linux you still have the same environment variables in both of them. It wouldn’t make sense that an environment variable changes just because you echo it from a different screen, would it? For that reason we get the display and not the monitor so that we get the same output on both. As about the hostname, since there is no info about it, the local host is assumed.

On a notice, if you have multiple monitors you can still specify which monitor to run an application by simply typing the full display variable you want. So if you have a monitor 0 and a monitor 1 on the current display, I can run firefox on monitor 1 with:

DISPLAY=:0.1 firefox

## Creating a virtual monitor

Instead of running X, we can run a different version of it that can create virtual displays. Xvfb (virtual framebuffer – whatever the hell that means) will create a virtual monitor for us.

So let’s make a new monitor (I assume you have installed xvfb):

```
Xvfb :1 -screen 0 1024x768x16
```

This will start the Xvfb server with a display 1 and a virtual screen(monitor) 0. We can access this by simply typing DISPLAY=:1.0 before running our graphical program. In this case the program will start in the virtual screen instead of our monitor.

Let’s make sure that the screen is still running:

```
> ps aux | grep X
root      1436  3.1  0.7 684580 91144 tty7     Ssl+ 09:47   3:31 /usr/bin/X -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
manos    22018  0.0  0.1 164960 20756 pts/27   Sl+  11:37   0:00 Xvfb :1 -screen 0 1024x768x16
```

We see we have the normal display 0. (A way to tell it is the default screen is to see that it runs as root.) We can also see the second display :1 and screen 0 with resolution 1024×768. So what if we want to use it?

Open a new terminal and type:

```
> DISPLAY=:1.0 firefox
..
```

This will start firefox at the given display. The reason I use the DISPLAY at the same line is to make sure that the subprocess inherits the variable DISPLAY. An other way to do this is to type:

```
> DISPLAY=:1.0
> export DISPLAY
> firefox
..
```

## Run a GUI program in a Docker container

We will now create a virtual screen inside a docker container.

```
> docker run -it ubuntu bash
root@660ddd5cc806:/# apt-get update
root@660ddd5cc806:/# apt-get install xvfb
root@660ddd5cc806:/# Xvfb :1 -screen 0 1024x768x16 &> xvfb.log  &
root@660ddd5cc806:/# ps aux | grep X
root        11  0.0  0.1 169356 20676 ?        Sl   10:49   0:00 Xvfb :1 -screen 0 1024x768x16
```

So now we are sure that we are running the virtual screen. Let’s access it and run something graphical on it. In this case I will run Firefox and Python+Selenium just as a proof of concept of what is happening.

First I put my display variable and use export to assure that any sub-shells or sub-processes use the same display (with export, they inherit the variable DISPLAY!):

```
root@660ddd5cc806:/# DISPLAY=:1.0
root@660ddd5cc806:/# export DISPLAY
```

Now we can simply run a browser

```
root@660ddd5cc806:/# firefox
(process:14967): GLib-CRITICAL **: g_slice_set_config: assertion 'sys_page_size == 0' failed
Xlib:  extension "RANDR" missing on display ":99.0".
(firefox:14967): GConf-WARNING **: Client failed to connect to the D-BUS daemon:
//bin/dbus-launch terminated abnormally without any error message
..
```

The errors don’t mean anything. But we can’t be sure, can we? I mean, since we can’t see what’s happening it’s really hard to tell. There are two things we can do, either use ImageMagick to take a snapshot and send it to our host via a socket or we can simply use Selenium. I will do that since most people probably want to achieve all this for testing purposes anyway.

```
root@0e395f0ef30a:/# apt-get install python-pip
root@0e395f0ef30a:/# pip install selenium
root@0e395f0ef30a:/# python
>>> from selenium import webdriver
>>> browser=webdriver.Firefox()
>>> browser.get("http://www.google.com")
>>> browser.page_source
```

If you get a bunch of HTML, then we have succeeded!

## The Chrome issue

If you try and run Chrome in a Docker container, it won’t work even if you have setup everything correctly. The reason is that Chrome uses something called sandboxing. Reading [this](http://www.google.com/googlebooks/chrome/med_26.html) I could not let but notice the word jail. Apparently it seems that Chrome uses Linux containers (the same that Docker uses). For this reason you have to put a bit of extra effort to solve this issue since because of technical difficulties it’s not possible to run containers in containers.

There are two workarounds:

1. Use my [docker-enter](https://github.com/Pithikos/docker-enter)
2. Use –privileged when running the container

The second solution is probably the best one. However while testing things, there’s nothing wrong with the first one.

So to make things work (notice I run everything from the start):

```
> docker run -it --privileged ubuntu bash
root@7dd2c07cb8cb:/# apt-get update
root@7dd2c07cb8cb:/# apt-get install wget python-pip xvfb
root@7dd2c07cb8cb:/# wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
OK
root@7dd2c07cb8cb:/# echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list
root@7dd2c07cb8cb:/# apt-get update
root@7dd2c07cb8cb:/# apt-get install google-chrome-stable
root@7dd2c07cb8cb:/# pip install selenium
```

I have now installed Selenium, Chrome and Xvfb. Now I am going to make make a virtual monitor and run Chrome:

```
root@7dd2c07cb8cb:/# Xvfb :99 -screen 0 1024x768x16 &> xvfb.log &
[1] 6729
root@7dd2c07cb8cb:/# DISPLAY=:99.0
root@7dd2c07cb8cb:/# export DISPLAY
root@7dd2c07cb8cb:/# google-chrome
Xlib:  extension "RANDR" missing on display ":99.0".
Xlib:  extension "RANDR" missing on display ":99.0".
[6736:6736:1017/143449:ERROR:desktop_window_tree_host_x11.cc(802)] Not implemented reached in virtual void views::DesktopWindowTreeHostX11::InitModalType(ui::ModalType)
ATTENTION: default value of option force_s3tc_enable overridden by environment.
failed to create drawable
[6775:6775:1017/143449:ERROR:gl_surface_glx.cc(633)] glXCreatePbuffer failed.
[6775:6775:1017/143449:ERROR:gpu_info_collector.cc(27)] gfx::GLContext::CreateOffscreenGLSurface failed
[6775:6775:1017/143449:ERROR:gpu_info_collector.cc(89)] Could not create surface for info collection.
[6775:6775:1017/143449:ERROR:gpu_main.cc(402)] gpu::CollectGraphicsInfo failed (fatal).
[6775:6775:1017/143449:ERROR:sandbox_linux.cc(305)] InitializeSandbox() called with multiple threads in process gpu-process
[6775:6775:1017/143449:ERROR:gpu_child_thread.cc(143)] Exiting GPU process due to errors during initialization
[6736:6736:1017/143449:ERROR:gpu_process_transport_factory.cc(418)] Failed to establish GPU channel.
```

It seems that it works. It’s normal that we get the gpu errors since we don’t have a gpu! However I don’t like gambling so we will take it a step further to check that the browser actually works. However for this I will need to download the webdriver for Google Chrome.

```
root@7dd2c07cb8cb:/# apt-get install curl unzip
root@7dd2c07cb8cb:/# cpu_arch=$(lscpu | grep Architecture | sed "s/^.*_//")
root@7dd2c07cb8cb:/# version=$(curl 'http://chromedriver.storage.googleapis.com/LATEST_RELEASE' 2> /dev/null)
root@7dd2c07cb8cb:/# url_file="chromedriver_linux${cpu_arch}.zip"
root@7dd2c07cb8cb:/# url_base="http://chromedriver.storage.googleapis.com"
root@7dd2c07cb8cb:/# url="${url_base}/${version}/${url_file}"
root@7dd2c07cb8cb:/# wget "$url"
root@7dd2c07cb8cb:/# unzip chromedriver_*.zip -d tmp
root@7dd2c07cb8cb:/# mv tmp/chromedriver usr/bin/
```

Now (FINALLY!) we can test with Selenium:

```
root@7dd2c07cb8cb:/# python
>>> from selenium import webdriver
>>> browser=webdriver.Chrome()
>>> browser.get("http://en.wikipedia.org/wiki/Open_source")
>>> browser.page_source
```

You should get a bunch of HTML code. So there we go!

## Common errors

```
.. Gtk: cannot open display:
```

DISPLAY has wrong value or you forgot to export it!

## References

- http://www.x.org/wiki/Development/Documentation/HowVideoCardsWork/
- http://www.x.org/archive/X11R7.7/doc/man/man1/Xvfb.1.xhtml
- http://blog.mecheye.net/2012/06/the-linux-graphics-stack/
- http://people.freedesktop.org/~marcheu/linuxgraphicsdrivers.pdf
- http://www.tldp.org/HOWTO/Framebuffer-HOWTO/
- http://en.wikipedia.org/wiki/X_Window_System
- http://en.wikipedia.org/wiki/Framebuffer
- http://en.wikipedia.org/wiki/X.Org_Server
- http://en.wikipedia.org/wiki/Display_server
- http://www.google.com/googlebooks/chrome/med_26.html
- http://linux.die.net/man/1/xvfb
- https://www.freebsd.org/doc/handbook/x-understanding.html

---

Original source: [Running a GUI application in a Docker container](http://linuxmeerkat.wordpress.com/2014/10/17/running-a-gui-application-in-a-docker-container/)