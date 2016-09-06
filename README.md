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

Once you have DDC for Azure set up, you will have a cluster with UCP and DTR in High Availablity mode. 

# DDC for Azure Architecture

![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/DDC-Azure-Arch.png?raw=true "DDC Azure Architecture ")




