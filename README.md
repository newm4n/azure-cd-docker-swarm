# Azure DevOps CD into On-Premise Docker Swarm Cluster

This documentation explain about how to setup a production environment using Docker Swarm. Docker Swarm is still widely used architecture choosen by many corporates. Other PaaS platform such as Kubernetes are definitely better compared to Docker Swarm but Kubernete's advanced capability also comes with it's own complexity to setup and to maintain.

For companies already have their own on-premise infrastructure containings many services and not yet ready to move to PaaS on the cloud using Kubernetes, Docker Swarm will definitely an option and could become a stepping stone should they ready to move to the cloud.

Further more, we also cover Continuous Deployment using Azure DevOps into a production environment running Docker Swarm Cluster.

## Topology

First, for the entire documentation, we going to have a sample topology of our deployment scenario. You can always adjust this to fit your current situation.

The diagram bellow shows our server and service formations.

![Deployment Topology](DockerSwarm-DeploymentSchema-Topology.png)

Please note that we assume that all host machines are running Ubuntu Linux.

### I. Docker Swarm Cluster

We going to setup multiple machine to form our docker swarm cluster. In this example we have total 23 hosts for the cluster, each of these hosts are assigned with static IPs. This hosts can be a bare metal (physical) computer, a Virtual machine, Virtual Private server or any kind of machine.

The cluster will have the following composition

- 1 Swarm Manager Master
- 2 Swarm Manager Slave
- 20 Swarm Worker

As you can see, 3 Swarm Manager is the advised production. As specified in this [Raft Consensus](https://docs.docker.com/engine/swarm/raft/), Docker is employing the following consensus : *"Raft tolerates up to (N-1)/2 failures and requires a majority or quorum of (N/2)+1 members to agree on values proposed to the cluster. "*

Thus the fault tolerance is becoming as follow

| Manager nodes | Failures tolerated |
| ------------- | ------------------ |
|    1          |       0            |
|    3          |       1            |
|    5          |       2            |
|    7          |       3            |

Thus if we have 3 Swarm Manager, only 1 of them are allowed to fail for the Manager to still working, This will enable the minimum HA (High-Availability) requirement.

You will also notice that Swarm Manager is operating in a Master-Slave model.

### II. Docker Registry

Docker Registry functions as a storage of docker images. This is the location to put docker images produced by deployment pipeline and also the location where those images were released into docker enabled host (our docker swarm).

There are public docker registry used by corporate around the world. One of the is [docker-hub](https://hub.docker.com/). You can setup your corporate account there and have all image published under the account to be private.

### III. CI / CD

Its a best practice to have an automated CI/CD process for build and deployment process. In this documentation, we use Azure DevOps for CI/CD

### IV. GIT Repository

GIT Repository will contains of the development code for your services.

## Deployment Process

To gain a better understanding on the deployment process, please observe the following diagram.

![Deployment Topology](DockerSwarm-DeploymentSchema-Process.png)

## Setup Everything

### Network Prerequisite

Within the server farm, its important that every hosts are able to *see* each other. 

All of the hosts must allow incoming connection to the following ports :

- TCP port 2377 for cluster management communications
- TCP and UDP port 7946 for communication among nodes
- UDP port 4789 for overlay network traffic

### Docker Prerequisite

For all host within the server farm, the latest **Docker Engine** should be installed in all of them. The complete manual to install docker in ubuntu can be accessed [here](https://docs.docker.com/engine/install/ubuntu/)

#### 1. Uninstall Old Docker
Please note that your ubuntu host might already have an old docker installed. 

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

#### 2. Install Docker Engine

1. Update `apt` and install necessary apt repository

```bash
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

$ sudo apt-get update
```

2. Install Docker Engine

```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

3. Test your docker

```bash
$ sudo docker run hello-world
```

Now lets prepare them one-by-one in correct order as shown in the following diagram.

![Deployment Topology](DockerSwarm-DeploymentSchema-Setup.png)

### I. Configure the Master Swarm Manager

### II. Configure the Slave Swarm Managers

### III. Configure the Swarm Workers

### IV. Configure the Docker Registry

### V. COnfigure the Firewall

### VI. Configure the Azure DevOps

### VII. Configure the Deployment Pipeline