# Wrapping Desktop Apps with Docker

---

##### Author: Adrian Mouat

---

The core purpose of Docker is to create portable, isolated containers holding a process (or possibly a set of processes) and its dependencies. Typically, the process represents a service such as an Apache Webserver or a Postgres database. However, there is no reason why it can’t also be a traditional desktop application such as Eclipse or Open Office. This post looks at how I used Docker to create [dim](https://github.com/amouat/dim), a customised version of Vim designed for “distraction free” editing of text files.

## Configuring Vim

Most of you have probably heard about editors such as [Writeroom](http://www.hogbaysoftware.com/products/writeroom) and [iA Writer](http://www.iawriter.com/), which are intentionally designed to be minimalist and sparse to encourage authors to focus on writing and ignore distractions.

At first, it sounds like it would be trivial to customise Vim to recreate a Writeroom style environment. However, it turns out many people have tried and achieved different levels of success, leaving a [bewildering](http://www.vim.org/scripts/script.php?script_id=4357) [sea](http://developerminutes.com/2014/02/16/write-mode-for-vim/) [of](https://github.com/jamestomasino/vim-writeroom) [blog](http://amix.dk/blog/post/19744#zenroom-for-Vim-Focsuing-only-on-the-essential) [posts](http://astrails.com/blog/2013/8/12/writing-markdown-with-style-in-vim) [and](https://mikewest.github.io/vimroom/) [plug-ins](http://bilalquadri.com/blog/2013/11/27/removing-distractions-from-vim/), each with different sets of features and problems.

In order to evaluate all the possible options, I had to make sweeping changes, installing various plug-ins and making heavy vimrc changes. Many of the plug-ins and suggested Vim settings were designed to work only with various other plug-ins and combinations of settings, resulting in bugs or unexpected behaviour outside of the author’s environment. For example, to provide a truly distraction free environment you need to turn off powerline, line-numbering and various other plug-ins and settings that the user may or may not have turned on. To make things easier, I wanted a known quantity – a fresh, default Vim environment. To achieve this, you could point Vim to a different config file by using the -u flag on the command line, but I’m not sure how you would go about pointing to a different set of plug-ins. It’s far easier just to use a Docker container to provide a clean Vim environment and not worry about it.

![alt](http://resource.docker.cn/gdim.png)
gDim in action

After a considerable amount of time evaluating plug-ins and hacking at Vim settings, I ended up using the following plug-ins:

- [Vundle](https://github.com/gmarik/Vundle.vim), a plug-in manager;
- [Goyo](https://github.com/junegunn/goyo.vim), which does most of the work, especially regarding centring text and;
- [Limelight](https://github.com/junegunn/limelight.vim), which dims all text except the current paragraph being worked on

I also made various tweaks to vimrc to handle wrapping etc and installed the [solarized](https://github.com/altercation/vim-colors-solarized) colour scheme.

## Launching Docker Containers as an Application

Dim is launched with a simple shell script which just runs a Docker container with the file to be edited mapped in as a volume. This means the user doesn’t need to worry about dim accidentally breaking their Vim set-up or anything else about their environment (with the exception of the file being edited). This is one of the advantages to using Docker to run applications – the application is sandboxed so the user can have more confidence the software isn’t doing anything nasty (like modifying the web browser home page or looking for personal files).

Dim exists in both a gVim version and a plain Vim version. The gVim version requires X, which means a bit more work is needed. To run X apps from Docker, we have a few choices. I could have used VNC or ssh with X forwarding, but it’s simpler and more efficient to just mount the X socket as a volume. This wouldn’t work if the Docker container was running on a remote host, but I assume this isn’t the case. Another issue is that details of other X window events from applications running on the host can be leaked through the socket, so it pokes a hole in the aforementioned isolation. To see how I’ve mapped the X socket, take a look at [gdim.sh](https://github.com/amouat/dim/blob/master/gdim.sh).

## Getting Dim

The other major advantage to using Docker is that it makes it very simple to install and distribute. Assuming you have Docker installed, it should just be case of downloading and running [gdim.sh](https://github.com/amouat/dim/blob/master/gdim.sh) or [dim.sh](https://raw.githubusercontent.com/amouat/dim/master/dim.sh). This will then go and grab the appropriate Docker image from the hub and launch it. Note that you need to provide a single argument which is the name of the file to be edited.

## Conclusion

There are some drawbacks and issues with dim. Most notably you have to exit the editor in order to edit a new file, which is a side-effect of the isolation I previously praised. There are also a few rough edges including the need to issue two quit commands (I think this is to do with how the text is centred).

Overall, I’m pretty happy with how this project has turned out – I’m currently editing this blog post using dim and not worrying about what changes I’ll need to make to my vimrc when I go back to editing code. The spartan editing environment takes some getting used to, but does encourage focusing on writing. Using Docker to bundle applications provides some clear benefits to both developer and end user – the developer can make assumptions about environment and dependencies and the end user can quickly get started with the software in a sandboxed environment which provides significant protection against malware.

I will be using to use this system for editing text files and markdown in the future, so expect to see minor updates and tweaks to the project.

If you want to learn more about dim or see how it works, have a look a the github project [amouat/dim](https://github.com/amouat/dim).

---

Original source: [Wrapping Desktop Apps with Docker](http://container-solutions.com/2014/10/wrapping-desktop-apps-docker/)