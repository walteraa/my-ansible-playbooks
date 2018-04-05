# Deploy OpenStack cluster

First of all, it is highly recommended the read of this [article](https://medium.com/@arthur.souzamiranda/kubernetes-with-openstack-cloud-provider-current-state-and-upcoming-changes-part-1-of-2-48b161ea449a) with part 1 of running a kubernetes cluster with OpenStack as internal cloud provider, in it Arthur Miranda introduces core concepts that will be quickly explained in here.


##  Introduction

Kubernetes has been a buzz word in tech for a long time, this is a sympton of both something with high recommendation rates and low failure rates. Other important characteristics of Kubernetes such as high availability, scalability, etc. can be easily proven with a few days using the tool. In this article we want to focus on one of K8S greatest abilities: integration.
Now jumping to the second part of the title, we have OpenStack which is an Infrastructure as a Service (aka IaaS) that provides bare metal management using its multiples services: Nova to provision compute instances, Neutron for network management, etc. 
Many companies from research and industry are using OpenStack as Infra management: CERN, Volkswagen, RedHat, Mirantis, etc. Uniting the popularity of Kubernetes and the history of OpenStack is a big deal nowadays.


## Kubernetes Archtecture

We can explain a little bit of Kubernetes archtecture to explain its integration to multiple clouds. Here is a simple diagram explaining our main components here:
Cloud controller manager arcghtectureLet’s go through the main components:

![Cloud controller](https://i.imgur.com/uKm5Xxz.jpg)

**Kube Controller Manager**: this component encapsulates control loop over kubernetes resources. Almost all components (like Pods, Deployments, etc) have a controller attached to it that runs a loop checking the health of its underlying containers, or even other kubernetes objects, if any failure is detected its duty is to activate the scheduler to make the cluster in the “desired” state.
**Kube API Server**: this is the main API of the kubernetes system. All requests to create/destroy/edit objects come through this component. It also delegates specific requests to build an application to other components.
**Kubelet**: this component manages all resouces in the nodes joined in a kubernetes cluster, all docker requests are made through Kubelet as well as collecting memory, disk, cpu, etc. data.
**Note**: there are other components in a kubernetes archtecture that are not represented in the image or mentioned here, the main idea is to explain how components interact and how our highlighted component fits in the kubernetes cluster..
**Cloud Controller Manager**: this is our main component, it is responsible to control all requests made to a cloud provider…. But what’s a cloud provider?

### Cloud Provider

In the image we see multiple cloud pieces with names such as “Google Cloud Provider”, “OpenStack Cloud Provider”, etc. The main reason this explanation is separated from the kubernetes explanations above is that: it is not a kubernetes resource, right? The cloud provider are external systems that provides some kinds of services usually network, nodes, volumes, etc. Cloud providers may be of three kinds: Public (like Google, AWS, Azure), Private (Platform9, VMWare) and Hybrid (OpenStack). 


## Running

### Pre requisites

You need a set of hosts with at least one Ubuntu virtual machine in your openstack environment. After that, edit your inentory file in `/etc/ansible/hosts` with the ip addresses and other cconfiguration necessary, you can check the options in this [link](http://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html).

```
[k8s-masters]
<ip>
<ip>
.
.
.

[k8s-minions]
<ip>
<ip>
.
.
.
```

After installing all dependencies, you need to edit the file with OS configuration:

**cloud-config**:
```
[Global]
region=RegionOne
username=$OPENSTACK_LOGIN
password=$OPENSTACK_PASSWORD
auth-url=$KEYSTONE_ENDPOINT
tenant-id=$PROJECT_ID
domain-id=$DOMAIN_ID
[LoadBalancer]
subnet-id=$SERVICE_SUBNET_ID
floating-network-id=$FLOAT_IP_NETWORK_ID
[BlockStorage]
bs-version=v2
```
All this information can be found in Horizon or via your openrc file.


### Fire!

Run the following command to start the playbook:

```bash
ansible-playbook k8s_master_configure.yaml
```

It will run all tasks regarding the master conf.

After that, do the same for the minion:

```bash
ansible-playbook k8s_minion_configure.yaml
```

Wait for all tasks to complete, copy the KUBECONFIG file in the master node and enjoy your k8s cluster.
