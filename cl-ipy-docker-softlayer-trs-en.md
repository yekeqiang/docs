# Use notebooks in the cloud for great data science

---

Author: Peter Parente

---



[IPython Notebook](http://ipython.org/) is a web-based environment for interactive computing and reproducible research. In a [previous tutorial](http://www.ibm.com/developerworks/cloud/library/cl-ipython-app/index.html), I explained how to run IPython Notebook on [IBM® Bluemix™](http://www.bluemix.net/?cm_mmc=dw-_-bluemix-_-cl-ipy-docker-softlayer-trs-_-article). I noted there that although deploying a stock notebook server to Bluemix is quick and easy, the result has its limitations (such as temporary storage and uncontrolled restarts).

You can overcome those limitations by deploying an IPython Notebook server to SoftLayer. In this tutorial, you'll:

1. Provision a SoftLayer virtual machine (VM) with [Docker](https://docker.com/), an application container engine, on the VM.

2. Pull an official IPython repository from the [Docker Hub](https://hub.docker.com/) and run the IPython Notebook server in a container.

3. Upload and play with a sample notebook that explores airline on-time performance data from [Data.gov](https://data.gov/), the U.S. government's open data site: 

![alt](http://resource.docker.cn/image001.jpg)

## What you'll need

 
- A SoftLayer account. You can get a free 30-day [SoftLayer trial](http://www.softlayer.com/info/free-cloud/dwfreetrial).

> “The command for creating a SoftLayer VM instance takes a handful of arguments, which you can copy and paste from this example into your own terminal.”

## Step 1. Start a SoftLayer VM
 
To get started, provision a SoftLayer VM with Docker on it. You can do this in one of two ways. If you prefer working at the command line and have Python installed, you can order a VM by using the [SoftLayer CLI](https://softlayer-api-python-client.readthedocs.org/en/latest/cli/). Alternatively, if you would rather work in a graphical environment, you can order a VM by using the [SoftLayer customer portal](https://manage.softlayer.com/Sales/orderHourlyComputingInstance) in your web browser.

I'll describe the command-line approach here. For instructions on ordering via the portal, see the "Provision a VM" section of this [blog post](http://mindtrove.info/docker-registry-softlayer-object-storage/).

To start your VM from the command line, install the SoftLayer CLI by using one of the supported methods listed in the [SoftLayer Python Client documentation](https://softlayer-api-python-client.readthedocs.org/en/latest/install/). For example, if you have pip installed and have root access on your local PC, run:

```
sudo pip install SoftLayer
```

Next, configure the CLI with your credentials by entering sl config setup and responding to the prompts:

```
sl config setup
Username: parente
API Key or Password: ******
Endpoint (public|private|custom): public
```

Your credentials are retained locally in ~/.softlayer.
Now, create an SSH keypair by entering ssh-keygen -t rsa and responding to all of the prompts. Save the key under the name dw-ipy in your ~/.ssh folder:

```
ssh-keygen -t rsa
Generating public/private rsa key pair.
# Enter file in which to save the key (/Users/parente/.ssh/id_rsa): /Users/parente/.ssh/dw-ipy
Enter passphrase (empty for no passphrase): ******
# Enter same passphrase again: ******
# Your identification has been saved in /Users/parente/.ssh/dw-ipy.
# Your public key has been saved in /Users/parente/.ssh/dw-ipy.pub.
# The key fingerprint is:
86:f4:c8:84:2f:15:52:74:b4:25:79:49:75:0e:9f:30 parente@localhost.local
```

Store your public key in your SoftLayer account under the label dw-ipy:

```
sl sshkey add dw-ipy -f ~/.ssh/dw-ipy.pub
SSH key added: 86:f4:c8:84:2f:15:52:74:b4:25:79:49:75:0e:9f:30
```

Now, create your VM instance. The creation command takes a handful of arguments, which you can copy and paste from this example into your own terminal:

```
sl vs create --hostname=ipython --domain=dw.ibm.com --key=dw-ipy --cpu=1 --memory=1024 --os=UBUNTU_14_64 --hourly --d sea01 --postinstall=https://bit.ly/1l2xaWE --wait=600
```

In the command arguments:

- `--hostname` and `--domain` are human-readable identifiers for your VM. They do not map to true DNS entries. Set them to any values you want.
- `--key` must be the label you assigned to the SSH public key that you stored earlier.
- `--cpu` and `--memory` dictate the resources assigned to your VM.
- `--os=UBUNTU_14_64` installs Ubuntu LTS 14.04 to your VM instance.
- `--hourly` indicates that the VM should be billed on an hourly (versus monthly) basis.
- `--d sea01` states that the VM should be created in the Seattle data center. (Run `sl vs create-options` for a list of data centers closest to you.)
- `--postinstall` points to a simple bash script that uses apt-get to install the latest version of Docker from Docker, Inc. (Of course, you can and should review the content of that script before trusting it.)
- `--wait` blocks the command for the specified number of seconds before returning.

When the command returns, it prints the ID of the VM along with its ready status:

```
:.........:......................................:
:    name : value                                :
:.........:......................................:
:      id : 6229756                              :
: created : 2014-09-17T21:04:38-05:00            :
:    guid : 506e3de8-6b86-47cc-8830-17e27141424c :
:   ready : True                                 :
:.........:......................................:
```

Query the VM to ensure that it's ready and to get the public IP address for connecting to it:

```
sl vs detail VM ID
```

For example:

```
sl vs detail 6229756
```

```
: ...................:...............................:
:               Name : Value                         :
:....................:...............................:
:                 id : 6229756                       :
:           hostname : ipython.dw.ibm.com            :

:             status : Active                        :
: active_transaction : -                             :
:              state : Running                       :
:         datacenter : sea01                         :
:                 os : Ubuntu                        :
:         os_version : 14.04-64 Minimal for VSI      :
:              cores : 1                             :
:             memory : 1G                            :
:          public_ip : 50.23.141.114                 :
:         private_ip : 10.28.164.13                  :
:       private_only : False                         :
:        private_cpu : False                         :
:            created : 2014-09-17T21:04:38-05:00     :
:           modified : 2014-09-17T21:06:23-05:00     :
:              vlans : :.........:........:........: :
:                    : :   type  : number :   id   : :
:                    : :.........:........:........: :
:                    : :  PUBLIC :  782   : 605768 : :
:                    : : PRIVATE :  961   : 605770 : :
:                    : :.........:........:........: :
:....................:...............................:
```

## Step 2. Pull the Docker image

When your VM is ready, use SSH to connect to its public IP and authenticate as root either with the private key you generated earlier or the password assigned by SoftLayer to the VM. For example, to use your private key, run:

```
ssh -i ~/.ssh/dw-ipy root@50.23.141.114
```

After you connect, run docker ps to check that the docker CLI is installed and the Docker daemon is running.

If you receive an error, the VM postinstall script might still be running. Wait a bit and try again.

When docker is available, pull the official [ipython/scipyserver Docker repository](https://registry.hub.docker.com/u/ipython/scipyserver/) from Docker Hub:

```
docker pull ipython/scipyserver
```

When the repository layers finish downloading, launch a container instance:

```
docker run -d -p 443:8888 -e PASSWORD=$( read -p "Password: " -s PASSWORD && echo $PASSWORD ) --restart always ipython/scipyserver
```

The arguments to the run command do the following:

- `-d` runs the container in the background.
- `-p` maps host port 443 to port 8888 in the container.
- `-e` sets the PASSWORD environment variable to a password that you enter.
- `--restart` tells the Docker daemon to restart the container instance whenever it stops (for example, on an unexpected crash or on a VM reboot).
- `ipython/scipyserver` states the image to use as the starting point for the container.

Now visit the address of your VM in your web browser using the HTTPS protocol (for example, https://50.23.141.114). Because you are using a self-signed SSL certificate, your browser will warn you that the server identity cannot be verified. For the purposes of this tutorial, you can proceed without worry. If you plan to use your SoftLayer notebook server in earnest, you should get a real SSL certificate.
Enter your credentials when prompted. Then confirm that you see the IPython Notebook dashboard:

![alt](http://resource.docker.cn/image002.jpg)


## Step 3. Download the sample notebook
 
To help you exercise your new notebook server, I've created a sample notebook that posits three questions about the on-time performance of U.S. air flights in the month of June 2014:

- What is the distribution by state of departure delays of at least 15 minutes? Arrival delays?
- Is there a tendency of flights from one state to another to experience a delay of 15 minutes or more on the arriving end?
- How does arrival delay vary day by day?

The notebook connects to a Cloudant database that I've populated with relevant data from Data.gov. It then proceeds to use a handful of the Python scientific computing packages installed in the ipython/scipyserver image to transform and visualize the data. Along the way, it captures my thinking about the data and results in Markdown-formatted comments that render as HTML.

To get the notebook:

1. In a new browser tab or window, open a read-only copy of the notebook on the IPython NBViewer site by visiting [this page](http://nbviewer.ipython.org/gist/parente/7cf287e32dfa49cd6664).
2. Click the **Download Notebook** icon on the top right of the page.
3. Return to the dashboard page of your IPython Notebook server.
4. Find where the page says **To import a notebook, drag the file onto the listing below or click here**. and click where it says **click here**.
5. Locate where the notebook downloaded on your local PC and select it for upload.
6. Click the **Exploration of Airline On-Time Performance.ipynb** notebook when you see it.

The notebook should open in a new tab in your browser. In this tab, you can navigate, execute, and edit the notebook cells using the menu options, toolbar buttons, or keyboard shortcuts listed in the Help menu. If the airline data interests you or you find the IPython environment intriguing, take time to play with the notebook and explore the data on your own.

## Go further

In this tutorial, you ran an IPython Notebook server in a Docker container on SoftLayer. Here are some ideas if you want to carry this exercise further.

- The sample notebook includes various suggestions for further exploration of the airline on-time performance data. You could tackle one or more of these and share your work as a new notebook.

- IPython has built-in support for parallel and distributed computing. For example, if you visit the Running tab in the dashboard of your notebook, you can launch multiple worker processes. If you run a container instance on a VM with multiple CPUs, you can use this feature to parallelize your work in a notebook.

- Your Docker container has the IPython Notebook server running as root by default. If you plan to run many container instances on the same VM or perhaps in the same SoftLayer account, you should consider running the server as non-root or otherwise, addressing the fact that [containers do not contain, yet](https://www.youtube.com/watch?v=zWGFqMuEHdw&list=UU76AVf2JkrwjxNKMuPpscHQ).

- The `docker run` command that you used stores your notebooks in the container. If you start doing serious work in your notebook instance, you should consider a backup strategy for your work, perhaps storing your notebooks in the [SoftLayer Object Store](https://github.com/rgbkrk/bookstore).

---

Original source: [Use notebooks in the cloud for great data science](http://www.ibm.com/developerworks/cloud/library/cl-ipy-docker-softlayer-trs/index.html)