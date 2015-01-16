# PANAMAX ANNOUNCES REMOTE AGENT DEPLOYMENTS

---

Author: Laura Frank 

---

The [Panamax](http://panamax.io/) team is happy to announce the v.0.1 launch of the Panamax Remote Agent toolset, consisting of the Panamax Remote Agent and adapters for deploying to both [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) and to a [CoreOS cluster](CoreOS cluster) using Fleet. This marks the next step in creating a tool that simplifies Docker management for humans. Keep in mind that both Panamax and the Remote Agent are nowhere near 1.0.

As mentioned in a [previous post](https://docker.cn/p/panamax-status-update-remote-agent-and-adapters-en) about the evolutionary path of Panamax, the core team agreed that Panamax templates – the heart of what makes working with Panamax simple and great – should be agnostic to host, orchestration, and cluster management technology.

Previously, Panamax was able to deploy to a local, single-node configuration using CoreOS and Fleet. We knew that users would have other use cases, specifically wanting to deploy their applications to clusters using schedulers and orchestrators.

With the release of the Panamax Remote Agent, Panamax can now work seamlessly with different infrastructures.

## SET UP YOUR DEPLOYMENT ENVIRONMENT

Currently, the Panamax Remote Agent is able to deploy to Kubernetes or a CoreOS cluster. As more adapters are written by the Panamax core team or contributed by the community, this list will continue to grow. Stay tuned for an Adapter Developer Guide available on the [Panamax Wiki](https://github.com/CenturyLinkLabs/panamax-ui/wiki).

In order to deploy to a target, you must first set up a development environment using your preferred service. Below are some helpful links to get you started. More information can be found on the Remote Agent wiki page.

### KUBERNETES

- [Kubernetes Project](https://github.com/GoogleCloudPlatform/kubernetes/)
- [Getting Started on Google Compute Engine](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/gce.md)
- [What is Kubernetes?](http://www.centurylinklabs.com/what-is-kubernetes-and-how-to-use-it/)

### FLEET

- [Running CoreOS on Google Compute Engine](https://coreos.com/docs/running-coreos/cloud-providers/google-compute-engine/)
- [Running CoreOS on CenturyLink Cloud](https://t3n.zendesk.com/entries/47064274-Building-CoreOS-Server-Cluster-on-the-CenturyLink-Cloud)

## ADD THE PANAMAX REMOTE AGENT NODE TO YOUR ENVIRONMENT

The Panamax Remote Agent must be installed on a dedicated node. We recommend using CoreOS for simplicity, but the agent can run on Linux with Docker installed as well. Add the additional VM to your environment, and ensure that the node is on the same network as your deployment node or cluster and that it has a public IP address. Port 3001 (tcp) must be open on the Panamax Remote Agent node.

Before installing the agent, test that your agent node connects to the cluster. Kubernetes: `telnet <kubernetes_master_IP> 443` – connection should be successful Fleet: `telnet <public_exposed_fleet_node_IP> 4001` – connection should be successful

## INSTALLING THE PANAMAX REMOTE AGENT

If your remote agent node is running CoreOS, Docker will already be installed for you. However, if you choose to run Linux, make sure that [Docker is installed](https://docs.docker.com/installation/ubuntulinux/) before proceeding with the remote agent installation.

- SSH into the Panamax remote agent node created above

- Run `$ sudo su` to ensure your docker commands will run with correct privileges

- Run `$ sudo bash -c "$(curl http://download.panamax.io/agent/pmx-agent-install)"`

- Select `(1) - init` to begin the installation process

- Follow the prompts, including choosing your adapter, adding your API endpoint (typically a private IP) and Panamax Remote Agent endpoint (typically a public IP)

- Copy the displayed token (needed when adding the deployment endpoint to the client)

You can run `$ ./pmx-agent to reinstall`, upgrade and run other options for the remote agent. The folder is located under root home. Access the folder, following these steps:

1. `$ sudo su`
2. `$ cd ~/pmx-agent`
3. `$ ./pmx-agent`

### NOTES FOR KUBERNETES ADAPTER

The API endpoint url (kubernetes-master node) should include https:// and no port, such as https://10.x.x.x. This is typically a private IP. The default API username is admin and your password is located in $ ~/.kubernetes_auth.

### NOTES FOR FLEET ADAPTER

The API endpoint url, which can be of any Fleet enabled node running in the cluster, should include http:// and port 4001. For example: http://10.x.x.x:4001. This is typically a private IP.

### NOTES FOR RUNNING ON GOOGLE COMPUTE ENGINE

The API endpoint url (k8s-containercluster-master) should include https:// and no port. For example: https://10.x.x.x. This is typically a private IP.
The default API username is admin and your password can be discovered by running the following commands where you have gcloud installed:

`$ gcloud components update preview`

`$ gcloud preview container clusters list -z ZONE`

## ADDING YOUR REMOTE DEPLOYMENT ENDPOINT TO PANAMAX

The final step is to let your installation of Panamax know about your remote deployment target.


Click Manage to access your dashboard; then click Remote Deployment Targets. 


![alt](http://resource.docker.cn/dashboard.png)

1x1.trans Panamax Announces Remote Agent Deployments
Click the button to add a new target, give the target a name, and then paste in the token you copied earlier. Click Create Remote Deployment Target to save. Your target is now available for deployments for any type of Panamax Template or application. The same remote deployment target can be set up on multiple machines, so any colleagues can deploy to the same endpoint. 

![alt](http://resource.docker.cn/remote-deployment-targets.png)


## DEPLOYING TO A TARGET

Browse to the Search page, and find a template you would like to deploy. Now, you may choose to run this template locally, as was previously done with Panamax, or deploy it to a remote target.

If you have more than one target set up, select your desired one.

Next, you can specify any environment variables and configuration options for the deployment. 

![alt](http://resource.docker.cn/skitch1.png)


Behind the Options tab, you can indicate scaling information for each service. 

![alt](http://resource.docker.cn/skitch2.png)

If using the Fleet adapter, scaling has some caveats when dealing with dependencies defined via Docker linking. Read more about it on the [Fleet Adapter wiki page](https://github.com/CenturyLinkLabs/panamax-ui/wiki/Fleet-Adapter).

Select Deploy To Target and the remote agent will deploy your application for you. You’ve just deployed to a remote deployment environment with a few clicks of your mouse.

Stay tuned for subsequent posts explaining the architecture of the adapters.

## RESOURCES

- [Remote Installation Wiki](https://github.com/CenturyLinkLabs/panamax-ui/wiki/Panamax-Remote-Agent-Installation)

- [Fleet Adapter](https://github.com/CenturyLinkLabs/panamax-ui/wiki/Fleet-Adapter)

- [Kubernetes Adapter](https://github.com/CenturyLinkLabs/panamax-ui/wiki/Fleet-Adapter)


You can find the core team in the [Panamax Support Chat](https://www.hipchat.com/gUjLli7k5) room, on IRC at #panamax (freenode), and you can also log any issues on [GitHub](https://github.com/orgs/CenturyLinkLabs).

---

Original source: [PANAMAX ANNOUNCES REMOTE AGENT DEPLOYMENTS](http://www.centurylinklabs.com/panamax-announces-remote-agent-deployments/)