# Azure-DDC-CICD
Setting up a CI/CD pipeline using Jenkins with Docker Datacenter for Azure 

#DOCKER DATA CENTER:

Docker Datacenter is an integrated solution including open source and commercial software, the integrations between them, full Docker API support, validated configurations and commercial support for your Docker Datacenter environment. A pluggable architecture allows flexibility in compute, networking and storage providers used in your CaaS infrastructure without disrupting the application code. Enterprises can leverage existing technology investments with Docker Datacenter. The open APIs allow Docker Datacenter CaaS to easily integrate into your existing systems like LDAP/AD, monitoring, logging and more.

You can find more information here: https://www.docker.com/products/docker-datacenter

![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/DDC-Arch.png?raw=true "DDC Components")

Docker Data Center consists for 3 components:

#Docker Univeral Control Plane (UCP):
UCP is an enterprise-grade cluster management solution from Docker that helps you manage your cluster using a single plane of glass. It is architected on top of swarm that comes with Docker engine. The UCP cluster consists of controllers ( masters) and nodes (workers).

# Docker Trusted Registry (DTR):
DTR is the enterprise-grade image storage solution from Docker that helps you can securely store and manage the Docker images you use in your applications. DTR is made of DTR replicas only that are deployed on UCP nodes.

#Commercially Supported Engine (CS Engine):
The CS engine adds support to the existing Docker engine. This becomes very useful when patches and fixes will have to be backported to engines running in production instead of updating the entire engine.

#DDC for Azure
Setting up DDC for Azure is very simple. A simple search for "Docker DataCenter" in the Azure marketplace will bring the template that will guide you through the process of setting it up. After a simple few clicks you can have DDC up and running in approximately 30 min. Please see here for detailed instructions on setting it up : https://success.docker.com/Datacenter/Apply/Docker_Datacenter_on_Azure

Thanks to @uday-shetty for the above writeup on setting up DDC for Azure.

Once you have DDC for Azure set up, you will have a cluster with UCP and DTR in High Availablity mode. 

# DDC for Azure Architecture

![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/DDC-Azure-Arch.png?raw=true "DDC Azure Architecture ")

As of this writing the Docker Datacenter on Azure Marketplace (template version 1.0.7) is based on the following stack:

1. Ubuntu 14.04.4 LTS
2. Docker UCP 1.1.2
3. Docker Trusted Registry 2.0.2
4. Docker CS Engine 1.11.2-cs3

#Continuous Integration and Continuous Deployment 

We will now walk through the steps for setting up a Continuous Integration, Continuous Deployment workflow using Jenkins and DDC. It is important you have DDC installed and running for this. If you don't go back up , follow the instructions and make sure you have DDC up and running before proceeding.

# CI/CD Architecture
We are going to create an environment from which demos of the Docker CICD use case along with using DTR and UCP can be done. Jenkins will run as a container and handle the building of Docker images.
![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/CI-CD.png?raw=true "CI/CD Architecture")

# Step 1: Setting up Jenkins
SSH to one of the worker nodes in the cluster and deploy Jenkins as a container

## SSH to worker node
ssh ucpadmin@ <ip-workernode-lb> -p 2200

## Run Jenkins as a Container:

```
docker run -d --restart=always -p 8080 --name jenkins \
-l interlock.hostname=harish2ucplb \
-l interlock.domain=eastus.cloudapp.azure.com \
-e constraint:node==harish20-ucpctrl \
-e DTR_URL=harish2dtrlb.eastus.cloudapp.azure.com \
-e DEMO_MASTER=10.0.0.4 \
-e DOMAIN_NAME=harish2ucplb.eastus.cloudapp.azure.com \
-e GITHUB_USERNAME=mbentley \
-v ucp-node-certs:/etc/docker:ro \
dockersolutions/jenkins
```
Launch your browser with the <ip-workernode-lb> :8080. You should see the Jenkins screen with a login and password page
Use credentials (u) demo and (p) docker123 to login.

