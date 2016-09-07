# Azure-DDC-CICD
Setting up a CI/CD pipeline using Jenkins with Docker Datacenter for Azure

# DOCKER DATA CENTER:

Docker Datacenter is an integrated solution including open source and commercial software, the integrations between them, full Docker API support, validated configurations and commercial support for your Docker Datacenter environment. A pluggable architecture allows flexibility in compute, networking and storage providers used in your CaaS infrastructure without disrupting the application code. Enterprises can leverage existing technology investments with Docker Datacenter. The open APIs allow Docker Datacenter CaaS to easily integrate into your existing systems like LDAP/AD, monitoring, logging and more.

You can find more information here: https://www.docker.com/products/docker-datacenter

![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/DDC-Arch.png?raw=true "DDC Components")

Docker Data Center consists for 3 components:

# Docker Univeral Control Plane (UCP):
UCP is an enterprise-grade cluster management solution from Docker that helps you manage your cluster using a single plane of glass. It is architected on top of swarm that comes with Docker engine. The UCP cluster consists of controllers ( masters) and nodes (workers).

# Docker Trusted Registry (DTR):
DTR is the enterprise-grade image storage solution from Docker that helps you can securely store and manage the Docker images you use in your applications. DTR is made of DTR replicas only that are deployed on UCP nodes.

# Commercially Supported Engine (CS Engine):
The CS engine adds support to the existing Docker engine. This becomes very useful when patches and fixes will have to be backported to engines running in production instead of updating the entire engine.

# DDC for Azure
Setting up DDC for Azure is very simple. A simple search for "Docker DataCenter" in the Azure marketplace will bring the template that will guide you through the process of setting it up. After a simple few clicks you can have DDC up and running in approximately 30 min. Please see here for detailed instructions on setting it up : https://success.docker.com/Datacenter/Apply/Docker_Datacenter_on_Azure

Thanks to @[uday-shetty](https://github.com/uday-shetty) for the above writeup on setting up DDC for Azure.

Once you have DDC for Azure set up, you will have a cluster with UCP and DTR in High Availablity mode.

# DDC for Azure Architecture

![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/DDC-Azure-Arch.png?raw=true "DDC Azure Architecture ")

As of this writing the Docker Datacenter on Azure Marketplace (template version 1.0.7) is based on the following stack:

1. Ubuntu 14.04.4 LTS
2. Docker UCP 1.1.2
3. Docker Trusted Registry 2.0.2
4. Docker CS Engine 1.11.2-cs3

# Continuous Integration and Continuous Deployment

We will now walk through the steps for setting up a Continuous Integration, Continuous Deployment workflow using Jenkins and DDC. It is important you have DDC installed and running for this. If you don't go back up , follow the instructions and make sure you have DDC up and running before proceeding.

# CI/CD Architecture
We are going to create an environment from which demos of the Docker CICD use case along with using DTR and UCP can be done. Jenkins will run as a container and handle the building of Docker images.
![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/CI-CD.png?raw=true "CI/CD Architecture")

# Step 1: Set up a wildcard DNS entry for applications
In order to deploy applications (including Jenkins) using the methods specified in the DDC reference architecture with Interlock, a wildcard DNS entry should be created to facilitate the easy of deploying applications.  Typically this should be a subdomain like `*.demo.example.com` where the DNS entry is a CNAME to the `ucplb` that was configured by the ARM template.  For example, `test.demo.example.com.  3600    IN      CNAME   harish2ucplb.eastus.cloudapp.azure.com.`.  Otherwise, individual DNS entries will need to be created for your application or your system's `hosts` file will need to be modified to fake DNS.

# Step 2: Setting up Jenkins
Using a client bundle from UCP, run Jenkins as a container.

## Run Jenkins as a Container:
Many of the parameters in the `docker run` command need to be updated for your specific environment.

 * `interlock.hostname` - hostname Jenkins will be available at (e.g. - `jenkins`)
 * `interlock.domain` - domain name Jenkins will be available at (e.g. - `demo.example.com`)
 * `constraint:node` - UCP node name to run Jenkins on; must be a UCP controller (e.g. - `harish20-ucpctrl`); can be found in the UCP `Nodes` UI
 * `DTR_URL` - FQDN to DTR (e.g. - harish2dtrlb.eastus.cloudapp.azure.com)
 * `DEMO_MASTER` - private IP address of the UCP controller you're running Jenkins on; can be found in the UCP `Nodes` UI
 * `DOMAIN_NAME` - domain name where a wildcard DNS entry has been added (e.g. - `demo.example.com`)
 * `GITHUB_USERNAME` - GitHub username where `https://github.com/harishjayakumar/Week2-PythonComposeSwarm-project` has been forked (e.g. - `harishjayakumar`)

Example:

```
docker run -d --restart=always -p 8080 --name jenkins \
  -l interlock.hostname=jenkins \
  -l interlock.domain=demo.example.com \
  -e constraint:node==harish20-ucpctrl \
  -e DTR_URL=harish2dtrlb.eastus.cloudapp.azure.com \
  -e DEMO_MASTER=10.0.0.4 \
  -e DOMAIN_NAME=demo.example.com \
  -e GITHUB_USERNAME=harishjayakumar \
  -v ucp-node-certs:/etc/docker:ro \
  -v jenkins-data:/var/lib/jenkins \
  dockersolutions/jenkins:latest
```
Launch your browser with FQDN to Jenkins specified by `interlock.hostname` and `interlock.domain` (e.g. - http://jenkins.demo.example.com). You should see the Jenkins screen with a login and password page
Use credentials (u) demo and (p) docker123 to login.

# Step 3: Ensure nodes trust DTR
In order to be able to pull/push between the cluster nodes and DTR we have to ensure that DTR trusts those nodes. 

## SSH to UCP-Controller nodes

UCP-Controller -1``` ssh ucpadmin@<<clbpip ip-address>> -p 2200```

UCP-Controller -2``` ssh ucpadmin@<<clbpip ip-address>> -p 2201```

UCP-Controller -3``` ssh ucpadmin@<<clbpip ip-address>> -p 2202```

## Copy Certificates

As seen above we just use the clb's (UCP Controller load balancer) ip address and change the port ids to ssh into the individual nodes. Once you are in use the following commands for DTR to trust the nodes:

```$ export DOMAIN_NAME=dtr.yourdomain.com```

Replace dtr.yourdomain.com with the DNS name of dlbip ( DTR load balancer)

```$ openssl s_client -connect $DOMAIN_NAME:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/$DOMAIN_NAME.crt```

```$ sudo update-ca-certificates```
    
```$ sudo service docker restart```

Repeat the steps above to SSH and trust the DTR for the DTR nodes and the worker nodes.

DTR nodes : ``` ssh ucpadmin@<<dlbpip ip-address>> -p 2200``` , 2201 and 2202 ports

Worker nodes: ``` ssh ucpadmin@<<nlbpip ip-address>> -p 2200```, 2201 

# Step 4: Setting up Github

Next, We want to make sure we set up your Github repository to trigger a jenkins build when a change is committed. 

From your repository in Github.. Click on Settings -> Webhooks and Services -> Add Service and search for "Jenkins github" plugin and add it

![Alt text](https://github.com/harishjayakumar/Azure-DDC-CICD/blob/master/Github-Set.png?raw=true "Github Setup")



