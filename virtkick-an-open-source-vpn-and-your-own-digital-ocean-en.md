# VirtKick: an Open Source VPN and Your Own Digital Ocean

----

Author: Lucas Carlson

---

[VirtKick](https://www.virtkick.io/) wraps awkard virtualization technology in a convenient, usable panel. VirtKick does to open source what DigitalOcean did to SaaS. Simplicity and privacy are their core goals. The project is currently [alpha and the code is on GitHub](https://github.com/virtkick).

Today we are talking to [Damian Nowak](https://www.nowaker.net/), the CEO of VirtKick.

### [00:45] Can you explain to everybody what VirtKick does?

VirtKick is, essentially, a self-hosted Digital Ocean. We enable almost anyone to manage their clouds without having to buy the VPS from any company we just let people order a dedicated server and then manage their clouds themselves.

### [01:24] So where do they get the dedicated server?

They can get a dedicated server from anywhere, it can be for example SoftLayer or OVH in Europe, you can get them from anywhere. Since dedicated servers are a bit different from VPS the security and the privacy you get is much better than with VPS. Thanks to this, if they would like to snoop on you they would need to physically get to the server and read the disks. Whereas with a VPS providers can read it without you noticing it, at any time. They are able to just perform a live snapshot of the system and pass it to the authorities.

Privacy is one of our concerns. The other concern that we are targeting is building competition in the VPS provider business. We will not only have the virtualization features that are essential for running the cloud but VirtKick will also ship with features usable by VPS providers. That means that payments, customer management, and a small ticket system will be shipped with VirtKick this means that many new VPS providers will be able to start their business, or existing providers will be able to switch to VirtKick and improve the user experience.

As you know, Digital Ocean revolutionized the business by doing this panel super reliable, super easy, and great. We are building the same thing for open source.

### [03:28] Can you explain the installation process and what you do to get it up and running?

At the moment we are at alpha stage but already the instructions are very very simple. There are only five commands to execute, one of which is 'git pull' and another is 'virtkick start'. So its already very simple however we are going to simplify it even more and package it so you just have a ddm package, an rpm package so it very simple to use. Unlike many usual Linux packages VirtKick will not need any manual install procedures, like editing some configuration files. We will get everything configured from the network, the tool storage pools, and things like that. Currently, VirtKick downloads all the ISOs so when you start VirtKick you can already pick Ubuntu and the image will just load up and you will have to install it in the VNC console.

The next step is providing virtual appliances so that you don't even have to manually install Ubuntu and other distros on the hard disk. You will just get your SSH keys injected.

### [05:12] How does it scale past a single server?

We only support one hypervisor, however we have plans on our short term to add support for many hypervisors and it will be very very easy to be the VirtKick administrator who goes to the panel and hit a button to add a hypervisor. You will provide SSH connection details and VirtKick will do everything on the remote hypervisor to make it available and after a while you will be able to create a new VM right on this server.

Another problem we have on our long term plan is to order a new dedicated server from SoftLayer or OVH or any other provider. So you don't even need to go to the VPS provider to order it we'll just do it for you without even leaving the VirtKick interface.

### [07:13] What is your background, and how did you come up with this idea?

A few years ago I had a few servers that were unused whereas my friends wanted some VPS servers, this was when Digital Ocean did not exist. I was trying to set up something for my friends and concluded there was really nothing interesting that was simple, easy and provides a good experience to these users. And there was, obviously, nothing available open source.

OpenStack, for example, is a great project however I believe it is great for big infrastructures, big corporate infrastructures where people expect more that to just have a VM. They expect to have some complicated networking infrastucture, failover redundancy, and things like that. These are things the neither Digital Ocean or Linode provide and they are fine with this. So there are many more people that just want the VM rather than people who want sophisticated configuration.

So four years ago there wasn't anything like that, just a conclusion that it was not possible. A year ago I starting think a bit on how I could do something to improve it. Three years had passed and nothing had happened in the open source community so I decided to do something about it. Then presented the idea to Rush (Damian Kaczmarek) who concluded this is a good idea and off we go.

### [09:53] What is the technology behind VirtKick?

The very first thing that you see is the UI. The UI is built with Rails and AngularJS, some parts are generated on the backend and some parts with more interaction are AngularJS. Our temporary backend that we are using is WebVirtMgr.

WebVirtMgr is another open source application that lets users control their virtualization with libvirt. We wrapped it into VirtKick, added JSON API on top of it, and use it as a JSON API to libvirt, where libvirt is another abstraction to KVM that we are using for virtualization. However, KVM is our first step, the next step will be OpenVZ, and the next step will be Docker.

[11:24] One of the things that you've been doing that is kind of interesting is that you are crowd-funding this, so its an open source project but it is also crowd-funded. How's that going, what's the goal there and what are you going to do with the money?

We started our crowd-funding some twenty days ago and the crowd-funding isn't going exactly well. In my understanding that is because people think it is too technical for them, whereas we aim for VirtKick to be super usable so anyone can kickstart their virtualization with one click. The problem we have in this virtualization business is that people are already used to this being very very hard. Hard to start, hard to use, people think this is too technical for it to benefit them.

However, we have been accepted to a startup accelerator program and we will receive some money that will boost our development. I believe we will be able to show people in three or four months from now that virtualization doesn't need to be that hard as it used to be with technologies that were already there.

### [13:23] On your Indiegogo page I think its really interesting that one of the big goals listed is a federated VPS for developers. Can you talk about what that means a little bit more?

Federated VPS for developers is something you can think like, people building open source projects need to run some sophisticated infrastructure tests. They currently do it through Vagrant which connects to Digital Ocean and they test all the stuff in Digital Ocean. What we are thinking of doing is after VirtKick is super popular amongst home users and small businesses that they would be able to donate their resources on servers, or even desktop computers, to the VirtKick federation. Open source developers will be able to run these infrastructure tests in virtual machines that will be created somewhere in the federation. For example, on your computer.

Thanks to that, open source developers won't need to pay money for these VMs spinning and running tests. For example, testing Docker files. You need a VM for that because it is currently not possible to do it in TravisCI and that is what Digital Ocean is using. We are planning on federating all these home users and providing something free and cool to software developers.

### [15:27] Could you run VirtKick on your Mac, use VirtualBox as the virtualization, and be able to have your Mac hosted virtual machines? Is that kind of the idea there?

Yes, that is also the idea. Of course, our first step will be to be able to donate these resources that are Linux boxes, however we have been thinking of supporting VirtualBox so that Mac users can join us. In the long term we will support VirtualBox and Mac users because this is essential for this to succeed.

### [16:22] If you join the crowd-funding campaign at the $199 level you get "copyright" instead of "copyleft". Can you explain the difference and what the benefit is?

Let's begin with what "copyleft" is. Copyleft is a way to ensure that the software and the source code belongs to people, belongs to the users, instead of to the corporation. That is the case for VirtKick. We license it against the AGPL license, which means that not only do they have to keep it still open but also share the source code with their users and visitors. For example, if a company implements some new extra feature in VirtKick, like an integration with BitCoin, they have to share it with their users so that we developers will be able to merge it back into the project.

This is how AGPL works and that is good for the community to benefit from companies that do something extra on VirtKick. However, some companies may need to develop something and they don't want to share this with others. So we let them do this as long as they pay something to us to boost our development. Even Richard Stallman (rms) said this is acceptable and OK so, if he says so, I am happy to do the same. I myself like the GPL approach whereas the other co-founder (Rush) is rather an MIT guy.

### [18:50] Is this how you are going to make money in the future? How do you guys plan to make money?

The most important thing in startups and earning money is that first you have to focus on users, and find those users, and the ways to monetize what you do are always to be found. We are currently not sure how to make money on it, maybe it will be paid support, maybe it will be licensing, or maybe it will be something else. For now we are just doing something that we know it cool, that we are doing something that comes out of our passion, something for the open source community. If it kick some money it will automatically go to us and our investors. We are hoping for good.

### [19:50] Since you have a billing system built in are you also thinking of charging a portion of how much you are making if you are a VirtKick user selling servers?

Yes, that is an assumption that has grown from this being open source. With this being open source anyone can run it and we have no means to get some cut of this, that's not possible. Anyone who sells these VPS servers to their customers has got to register their account with Stripe or Paypal so we won't get anything from that. However, if we add some links in VirtKick and they click on these links at least we will get some money by referring people to this.

Indirectly there is a way to get some little money on the referrals, however there is no way to get the percentage of sales of other companies. Frankly we don't even need something like this, we don't want… We want them to sell these VPS servers and make VirtKick even more popular.

### [21:26] Are you right out of college, are you right out of a job, what is your employment background?

Both of us co-founders were previously working in some daily job and at the same time were doing VirtKick. I was working for Zerigo, a cloud hosting provider, for one year and I was working on the source code of the DNS offer and the VPS offer.

At the same time it has been almost four years I have been working on my small business that provided hosting of Atlassian applications, Jira and Confluence, and at the same time we are offering VPS servers to our friends. Virtualization is something I have been working with for four years now.

Rush is a Linux guy. He knows how C bindings work, how NodeJS calls interact with Linux, so he is essential in the project because he will be the guy who will contribute to the libvirt project, who does things like wrapping VNC connections with NodeJS, HTTP servers. He used to work on some imbedded code on some ARM processors.

### [23:33] Are you both doing this full time, is this all you do now?

For the last two months we have been doing it full time because we saw the potential and we decided that this is something that we have to focus because we have to keep up with this and make it finally. So that it is not something that is announced and never developed. We plan to get the 1.0 version somewhere in January.

### [24:15] What's the future, where is this all going, what would be your wildest dream?

The wildest dream would be that when people when they want to have a new VPS server they will have to think about it for a minute because there will be so many great VPS providers they won't be able to choose the best one because all of them will be very good.

### [24:50] Any other thoughts, anything that you wanted to add?

The one thing I would like to ask is for supporting our open source project and our Indiegogo campaign. [https://www.indiegogo.com/projects/virtkick-take-cloud-back]

### [25:10] If anyone finds this interesting, make sure you contribute before Christmas 2014. If you check out the GitHub/VirtKick website they have a great open source repository. The installation instructions, as he mentioned, are really really simple, there's even a button, a one click install on Digital Ocean. How does that work?

We haven't tested it, but someone who tested it added it so we just accepted the pull request. I know that Digital Ocean currently supports nested virtualization so its possible to create VirtKick in Digital Ocean and start a virtual machine.

The install on Digital Ocean button is a very cool initiative. Someone thought they could use these new features, metadata API feature from Digital Ocean and implement something like this.

---

Original source: [VirtKick: an Open Source VPN and Your Own Digital Ocean](http://www.centurylinklabs.com/interviews/virtkick-an-open-source-vpn-and-your-own-digital-ocean/)