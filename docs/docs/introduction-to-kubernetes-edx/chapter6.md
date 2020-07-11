---
layout: default
title: C6. Minikube 
published: true
nav_order: 5
parent: Introduction To Kubernetes- edX
---


⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__


- [Introduction](#introduction)
  - [Requirements for Running Minikube](#requirements-for-running-minikube)
- [Installing Minikube on Linux](#installing-minikube-on-linux)
  - [Install Minikube](#install-minikube)
  - [Start Minikube](#start-minikube)
  - [Check the status](#check-the-status)
  - [Stop minikube](#stop-minikube)
- [Installing Minikube on macOS](#installing-minikube-on-macos)
  - [Install Minikube](#install-minikube-1)
  - [Start Minikube](#start-minikube-1)
  - [Check the status](#check-the-status-1)
  - [Stop minikube](#stop-minikube-1)
- [Minikube CRI-O](#minikube-cri-o)
- [Installation Video](#installation-video)

# Introduction

As we mentioned in the previous chapter, Minikube is the easiest and most recommended way to run an all-in-one Kubernetes cluster locally on our workstations. In this chapter, we will explore the requirements to install Minikube locally on our workstation, together with the installation instructions to set it up on local Linux, macOS, and Windows operating systems.


## Requirements for Running Minikube

Minikube is installed and runs directly on a local Linux, macOS, or Windows workstation. However, in order to fully take advantage of all the features Minikube has to offer, a Type-2 Hypervisor should be installed on the local workstation, to run in conjunction with Minikube. This does not mean that we need to create any VMs with guest operating systems with this Hypervisor.


Minikube builds all its infrastructure as long as the Type-2 Hypervisor is installed on our workstation. Minikube invokes the Hypervisor to create a single VM which then hosts a single-node Kubernetes cluster. Thus we need to make sure that we have the necessary hardware and software required by Minikube to build its environment. Below we outline the requirements to run Minikube on our local workstation:


- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

kubectl is a binary used to access and manage any Kubernetes cluster. It is installed separately from Minikube. Since we will install kubectl after the Minikube installation, we may see warnings during the Minikube initialization - safe to disregard for the time being, but do keep in mind that we will have to install kubectl to be able to manage the Kubernetes cluster. We will explore kubectl in more detail in future chapters.

- Type-2 Hypervisor

  - On Linux [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [KVM](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver)
  - On macOS [VirtualBox](https://www.virtualbox.org/wiki/Downloads), [HyperKit](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver), or [VMware Fusion](http://www.vmware.com/products/fusion.html)
  - On Windows [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [Hyper-V](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperv-driver)

   __NOTE:__ Minikube supports a --vm-driver=none option that runs the Kubernetes components directly on the host OS and not inside a VM. With this option a Docker installation is required and a Linux OS on the local workstation, but no hypervisor installation. If you use --vm-driver=none, be sure to specify a bridge network for Docker. Otherwise, it might change between network restarts, causing loss of connectivity to your cluster.

  - VT-x/AMD-v virtualization must be enabled on the local workstation in BIOS
  - Internet connection on first Minikube run - to download packages, dependencies, updates and pull images needed to initialize the Minikube Kubernetes cluster. Subsequent runs will require an internet connection only when new Docker images need to be pulled from a container repository or when deployed containerized applications need it. Once an image has been pulled it can be reused without an internet connection.

  - In this chapter, we use VirtualBox as hypervisor on all three operating systems - Linux, macOS, and Windows, to allow Minikube to provision the VM which hosts the single-node Kubernetes cluster.

  - Read more about Minikube from the official [Kubernetes documentation](https://kubernetes.io/docs/setup/minikube/) or [GitHub](https://github.com/kubernetes/minikube).


# Installing Minikube on Linux

Let's learn how to install Minikube v1.0.1 on Ubuntu Linux 18.04 LTS with VirtualBox v6.0 specifically.

__NOTE: For other versions, the installation steps may vary! Check the [Minikube installation](https://kubernetes.io/docs/tasks/tools/install-minikube/)!__

Install the [VirtualBox](https://www.virtualbox.org/wiki/Linux_Downloads) hypervisor

```bash
$ sudo bash -c 'echo "deb https://download.virtualbox.org/virtualbox/debian bionic contrib" >> /etc/apt/sources.list'
$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install -y virtualbox-6.0
```

## Install Minikube

We can download the latest release from the [Minikube release page](https://github.com/kubernetes/minikube/releases). At the time the course was written, the latest Minikube release was v1.0.1. Once downloaded, we need to make it executable and add it to our __PATH__:

```bash 
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.0.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

**NOTE: Replacing /v1.0.1/ with /latest/ will always download the latest version.**

## Start Minikube

We can start Minikube with the `minikube start` command (disregard "Unable to read.../docker/config..." and "No matching credentials..." warnings):

```bash 
$ minikube start
minikube v1.0.1 on linux (amd64)
Downloading Minikube ISO ...
142.88 MB / 142.88 MB [============================================] 100.00% 0s
Downloading Kubernetes v1.14.1 images in the background ...
Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
"minikube" IP address is 192.168.99.100
Configuring Docker as the container runtime ...
Version of container runtime is 18.06.3-ce
Waiting for image downloads to complete ...
Preparing Kubernetes environment ...
Downloading kubeadm v1.14.1
Downloading kubelet v1.14.1
Pulling images required by Kubernetes v1.14.1 ...
Launching Kubernetes v1.14.1 using kubeadm ... 
Waiting for pods: apiserver proxy etcd scheduler controller dns
Configuring cluster permissions ...
Verifying component health .....
kubectl is now configured to use "minikube"
For best results, install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
Done! Thank you for using minikube!

```

## Check the status

With the `minikube status` command, we display the status of Minikube:

```bash 
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

## Stop minikube

 With the `minikube stop` command, we can stop Minikube:

 ```bash 
$ minikube stop
Stopping "minikube" in virtualbox ...
"minikube" stopped.
 ```


# Installing Minikube on macOS

Let's learn how to install Minikube v1.0.1 on Mac OS X with VirtualBox v6.0 specifically.

 __NOTE: For other versions, the installation steps may vary! Check the [Minikube installation](https://kubernetes.io/docs/tasks/tools/install-minikube/)!__


Although VirtualBox is the default hypervisor for Minikube, on Mac OS X we can configure Minikube at startup to use another hypervisor, with the `--vm-driver=xhyve` or `=hyperkit start` option.

Install the [VirtualBox](https://www.virtualbox.org/wiki/Downloads) hypervisor for OS X hosts
Download and install the .dmg package.


## Install Minikube

 We can download the latest release from the [Minikube release](https://github.com/kubernetes/minikube/releases) page. At the time the course was written, the latest Minikube release was v1.0.1. Once downloaded, we need to make it executable and add it to our PATH:

 ```bash 

$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.0.1/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

 ```
## Start Minikube

We can start Minikube with the `minikube start` command (disregard "Unable to read.../docker/config..." and "No matching credentials..." warnings):

```bash 

$ minikube start
minikube v1.0.1 on darwin (amd64)
Downloading Kubernetes v1.14.1 images in the background ...
Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
Downloading Minikube ISO ...
142.88 MB / 142.88 MB [============================================] 100.00% 0s
"minikube" IP address is 192.168.99.100
Configuring Docker as the container runtime ...
Version of container runtime is 18.06.3-ce
Waiting for image downloads to complete ...
Preparing Kubernetes environment ...
Downloading kubeadm v1.14.1
Downloading kubelet v1.14.1
Pulling images required by Kubernetes v1.14.1 ...
Launching Kubernetes v1.14.1 using kubeadm ... 
Waiting for pods: apiserver proxy etcd scheduler controller dns
Configuring cluster permissions ...
Verifying component health .....
kubectl is now configured to use "minikube"
For best results, install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
Done! Thank you for using minikube!

```

## Check the status 

With the `minikube status` command, we display the status of Minikube:

```bash 

$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

## Stop minikube

With the `minikube stop` command, we can stop Minikube:

```bash 
$ minikube stop
Stopping "minikube" in virtualbox ...
"minikube" stopped.
```


# Minikube CRI-O

According to the CRI-O website,

__"CRI-O is an implementation of the Kubernetes CRI (Container Runtime Interface) to enable using OCI (Open Container Initiative) compatible runtimes."__

Start Minikube with CRI-O as container runtime, instead of Docker, with the following command:

```bash 

$ minikube start --container-runtime=cri-o
minikube v1.0.1 on linux (amd64)
Downloading Kubernetes v1.14.1 images in the background ...
Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
Restarting existing virtualbox VM for "minikube" ...
Waiting for SSH access ...
"minikube" IP address is 192.168.99.100
Configuring CRI-O as the container runtime ...
Version of container runtime is 1.13.5
Waiting for image downloads to complete ...
Preparing Kubernetes environment ...
Pulling images required by Kubernetes v1.14.1 ...
Relaunching Kubernetes v1.14.1 using kubeadm ... 
Waiting for pods: apiserver etcd scheduler controller
Updating kube-proxy configuration ...
Verifying component health ......
kubectl is now configured to use "minikube"
For best results, install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
Done! Thank you for using minikube!

```

__Let's login via ssh into the Minikube's VM:__

```bash

$ minikube ssh

                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ _

```

__NOTE: If you try to list containers using the docker command, it will not produce any results, because Docker is not running containers:__


```bash 
$ sudo docker container ls
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

__List the containers created via CRI-O container runtime with the following command:__

```bash 
$ sudo runc list
ID                                                                 PID         STATUS      BUNDLE                                                                                                                 CREATED                          OWNER 
1090869caeea44cb179d31b70ba5b6de96f10a8a5f4286536af5dac1c4312030   3661        running     /run/containers/storage/overlay-containers/1090869caeea44cb179d31b70ba5b6de96f10a8a5f4286536af5dac1c4312030/userdata   2019-04-18T20:03:02.199284303Z   root
1e9f8dce6d535b67822e744204098060ff92e574780a1809adbda48ad8605d06   3614        running     /run/containers/storage/overlay-containers/1e9f8dce6d535b67822e744204098060ff92e574780a1809adbda48ad8605d06/userdata   2019-04-18T20:03:02.129881761Z   root
1edcfc78bca52be153cc9f525d9fc64be75ccea478897004a5032f37c6c4c9dc   3812        running     /run/containers/storage/overlay-containers/1edcfc78bca52be153cc9f525d9fc64be75ccea478897004a5032f37c6c4c9dc/userdata   2019-04-18T20:03:02.740669541Z   root
```


# Installation Video 

<iframe width="736" height="450" src="https://www.youtube-nocookie.com/embed/M1DMukMBjz8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>