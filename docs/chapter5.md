---
layout: default
title: C5. Installing Kubernetes 
published: true
nav_order: 4
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Kubernetes Configuration](#kubernetes-configuration)
  - [Infrastructure for Kubernetes Installation](#infrastructure-for-kubernetes-installation)
  - [Localhost Installation](#localhost-installation)
  - [On-Premise Installation](#on-premise-installation)
  - [Cloud Installation](#cloud-installation)
  - [Kubernetes Installation Tools/Resources](#kubernetes-installation-toolsresources)

# Kubernetes Configuration 

- **All-in-One Single-Node Installation**
  
  In this setup, all the master and worker components are installed and running on a single-node. While it is useful for learning, development, and testing, it should not be used in production. Minikube is one such example, and we are going to explore it in future chapters.

- **Single-Node etcd, Single-Master and Multi-Worker Installation**

  In this setup, we have a single-master node, which also runs a single-node etcd instance. Multiple worker nodes are connected to the master node.

- **Single-Node etcd, Multi-Master and Multi-Worker Installation**

   In this setup, we have multiple-master nodes configured in HA mode, but we have a single-node etcd instance. Multiple worker nodes are connected to the master nodes.

- **Multi-Node etcd, Multi-Master and Multi-Worker Installation**

   In this mode, etcd is configured in clustered HA mode, the master nodes are all configured in HA mode, connecting to multiple worker nodes. This is the most advanced and recommended production setup.


## Infrastructure for Kubernetes Installation

Once we decide on the installation type, we also need to make some infrastructure-related decisions, such as:

- Should we set up Kubernetes on bare metal, public cloud, or private cloud?
- Which underlying OS should we use? Should we choose RHEL, CoreOS, CentOS, or something else?
- Which networking solution should we use?

Explore the [Kubernetes documentation](https://kubernetes.io/docs/setup/) for details on choosing the right solution. Next, we will take a closer look at these solutions.

## Localhost Installation

These are only a few localhost installation options available to deploy single- or multi-node Kubernetes clusters on our workstation/laptop:

- [Minikube](https://kubernetes.io/docs/getting-started-guides/minikube/) - single-node local Kubernetes cluster
- [Docker Desktop](https://www.docker.com/products/docker-desktop)-  single-node local Kubernetes cluster for Windows and Mac
- [CDK on LXD](https://www.ubuntu.com/kubernetes/docs/install-local) -  multi-node local cluster with LXD containers.

**Minikube** is the preferred and recommended way to create an all-in-one Kubernetes setup locally. We will be using it extensively in this course.

## On-Premise Installation

Kubernetes can be installed on-premise on VMs and bare metal.

- **On-Premise VMs**

Kubernetes can be installed on VMs created via Vagrant, VMware vSphere, KVM, or another Configuration Management (CM) tool in conjunction with a hypervisor software. There are different tools available to automate the installation, such as [Ansible](https://www.ansible.com/) or [kubeadm](https://github.com/kubernetes/kubeadm).

- **On-Premise Bare Metal**
- 
Kubernetes can be installed on on-premise bare metal, on top of different operating systems, like RHEL, CoreOS, CentOS, Fedora, Ubuntu, etc. Most of the tools used to install Kubernetes on VMs can be used with bare metal installations as well.

## Cloud Installation

Kubernetes can be installed and managed on almost any cloud environment:

- **Hosted Solutions**
With Hosted Solutions, any given software is completely managed by the provider. The user pays hosting and management charges. Some of the vendors providing hosted solutions for Kubernetes are:

   <ul>
    <li><a href="https://cloud.google.com/kubernetes-engine/" target="_blank">Google Kubernetes Engine</a> (GKE)</li>
    <li><a href="https://azure.microsoft.com/en-us/services/kubernetes-service/" target="_blank">Azure Kubernetes Service</a> (AKS)</li>
    <li><a href="https://aws.amazon.com/eks/" target="_blank">Amazon Elastic Container Service for Kubernetes</a> (EKS)</li>
    <li><a href="https://www.digitalocean.com/products/kubernetes/" target="_blank">DigitalOcean Kubernetes</a></li>
    <li><a href="https://www.openshift.com/products/dedicated/" target="_blank">OpenShift Dedicated</a></li>
    <li><a href="https://platform9.com/managed-kubernetes/" target="_blank">Platform9</a></li>
    <li><a href="https://cloud.ibm.com/docs/containers?topic=containers-getting-started#container_index" target="_blank">IBM Cloud Kubernetes Service</a>.</li>
    </ul>
<br>

- **Turnkey Cloud Solutions**
Below are only a few of the Turnkey Cloud Solutions, to install Kubernetes with just a few commands on an underlying IaaS platform, such as:

   <ul>
    <li><a href="https://cloud.google.com/gke-on-prem/" target="_blank">GKE On-Prem</a> by Google Cloud</li>
    <li><a href="https://www.ibm.com/cloud/private" target="_blank">IBM Cloud Private</a></li>
    <li><a href="https://www.openshift.com/products/container-platform/" target="_blank">OpenShift Container Platform</a> by Red Hat.</li>
    </ul>
<br>

## Kubernetes Installation Tools/Resources

While discussing installation configuration and the underlying infrastructure, let's take a look at some useful tools/resources available:

<ul>
    <ul>
        <li><strong>kubeadm</strong><br><a href="https://github.com/kubernetes/kubeadm" target="_blank">kubeadm</a> is a first-class citizen on the Kubernetes ecosystem. It is a secure and recommended way to bootstrap a single- or multi-node Kubernetes cluster. It has a set of building blocks to setup the cluster, but it is easily extendable to add more features. Please note that kubeadm does not support the provisioning of hosts.</li>
        <li><strong>kubespray</strong><br>With <a href="https://kubernetes.io/docs/setup/custom-cloud/kubespray/" target="_blank">kubespray</a> (formerly known as kargo), we can install Highly Available Kubernetes clusters on AWS, GCE, Azure, OpenStack, or bare metal. Kubespray is based on Ansible, and is available on most Linux distributions. It is a <a href="https://github.com/kubernetes-sigs/kubespray" target="_blank">Kubernetes Incubator</a> project.</li>
        <li><strong>kops</strong><br>With <a href="https://kubernetes.io/docs/setup/custom-cloud/kops/" target="_blank">kops</a>, we can create, destroy, upgrade, and maintain production-grade, highly-available Kubernetes clusters from the command line. It can provision the machines as well. Currently, AWS is officially supported. Support for GCE is in beta, and VMware vSphere in alpha stage, and other platforms are planned for the future. Explore the&nbsp;<a href="https://github.com/kubernetes/kops" target="_blank">kops project</a> for more details.</li>
        <li><strong>kube-aws</strong><br>With <a href="https://github.com/kubernetes-incubator/kube-aws" target="_blank">kube-aws</a> we can create, upgrade and destroy Kubernetes clusters on AWS from the command line. Kube-aws is also a Kubernetes Incubator project.</li>
    </ul>
</ul>


It is worth checking out the [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) GitHub project by Kelsey Hightower, which shares the manual steps involved in bootstrapping a Kubernetes cluster.