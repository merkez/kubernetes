---
layout: default
title: Installation
nav_order: 3
has_children: true
permalink: /docs/installation
---

- [Setting Up Nodes](#setting-up-nodes)
  - [Enable Root Access](#enable-root-access)
  - [Modify local DNS record](#modify-local-dns-record)
  - [Copy SSH key-pairs to nodes](#copy-ssh-key-pairs-to-nodes)
  - [Install Paralel SSH](#install-paralel-ssh)
  - [Copy node IPs to `/etc/hosts` for all nodes](#copy-node-ips-to-etchosts-for-all-nodes)
  - [Install container runtime (Docker)](#install-container-runtime-docker)
  - [Content of Installation Script](#content-of-installation-script)

# Setting Up Nodes 

In this installation tutorial an example of an Kubernetes cluster mechanism will be installed. Kubernetes cluster will have following nodes with specified features below.

Note that all nodes are running **ubuntu18.04**

<style type="text/css">
.tg  {border-collapse:collapse;border-color:#9ABAD9;border-spacing:0;}
.tg td{background-color:#EBF5FF;border-color:#9ABAD9;border-style:solid;border-width:1px;color:#444;
  font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{background-color:#409cff;border-color:#9ABAD9;border-style:solid;border-width:1px;color:#fff;
  font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-pwe8{background-color:#fe0000;color:#ffffff;font-weight:bold;text-align:left;vertical-align:top}
.tg .tg-0lax{text-align:left;vertical-align:top}
.tg .tg-amwm{font-weight:bold;text-align:center;vertical-align:top}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-0lax"></th>
    <th class="tg-amwm">Master Nodes</th>
    <th class="tg-amwm">Specs</th>
    <th class="tg-amwm">Worker Nodes</th>
    <th class="tg-amwm">Specs<br></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-pwe8">M/W1<br></td>
    <td class="tg-0lax">10.0.128.90</td>
    <td class="tg-0lax">2 VCPU<br>16 GB RAM<br></td>
    <td class="tg-0lax">10.0.128.193</td>
    <td class="tg-0lax">2 VCPU<br>16 GB RAM</td>
  </tr>
  <tr>
    <td class="tg-pwe8">M/W2</td>
    <td class="tg-0lax">10.0.128.138</td>
    <td class="tg-0lax">2 VCPU<br>16 GB RAM</td>
    <td class="tg-0lax">10.0.128.149</td>
    <td class="tg-0lax">2 VCPU<br>16 GB RAM</td>
  </tr>
  <tr>
    <td class="tg-pwe8">M/W3</td>
    <td class="tg-0lax">10.0.128.9</td>
    <td class="tg-0lax">4 VCPU<br>32 GB RAM</td>
    <td class="tg-0lax">10.0.128.163</td>
    <td class="tg-0lax">2 VCPU<br>16 GB RAM</td>
  </tr>
</tbody>
</table>


The nodes could be spinned up through any cloud providers or on your own servers. For initial configuration make sure that you have enabled ssh connection to at least one master node with external ip address. It can enable to ssh other servers which are closed to outside world _(no external ip adress)_. 


## Enable Root Access 

It is practical to enable ROOT access through SSH connection, to enable  that functionality if it is not enabled already, follow steps below. 

* Enable following values on `/etc/ssh/sshd_config` file

```bash 
 
$ echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
$ sed -i -E 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config

```

* Restart ssh service 

```bash 
$ systemctl restart sshd 
```

Assumed that ROOT user has password, if not create with; 

```bash 
$ passwd
```

Once this process is done, we will be able to ssh to any master or worker node from current master node (which has external ip.)

Only one master node has external IP address, other worker and master nodes do not have any particular external IP address which is bound to them. Hence, we connect to any worker and master node through main master node. 

## Modify local DNS record

*M1* has external IP address and all installation process will be taken from that point to other machines. 

Before starting installatoin process, it would be handful to import all master and worker nodes to `/etc/hosts` file, this way we can use short and clear name when dealing with nodes. 

Append following lines to `/etc/hosts` in each node. To do that, first append all IP adresses of master and worker nodes in current machine. 


```conf 
10.0.128.90 master1
10.0.128.9 master3
10.0.128.138 master2
10.0.128.193 worker1
10.0.128.149 worker2
10.0.128.183 worker3
```

Once IP addresses are appended to `/etc/hosts` file, each machine can recognize itself with name. `/etc/hosts` is holding DNS records in local communication. 

## Copy SSH key-pairs to nodes

- Create default SSH key-pair

```bash 
$ ssh-keygen
```

- Create list of nodes, file called `nodes`

```bash 
$ cat nodes
master1
master3
master2
worker2
worker3
worker1
```

- Copy default ssh key to nodes 

```bash 
$ for i in $(cat nodes); ssh-copy-id -f $i; done
```

Test connection by; 

```bash 
$ ssh master3 # when username of users on nodes same 
```

## Install Paralel SSH

- Install parallel-ssh on main master

```bash

$ apt install pssh
```

- From `/usr/share/doc/pssh/README.Debian`

```
To avoid any conflicts with the putty package, all of the programs have been renamed.
- parallel-ssh is pssh 
- parallel-scp is pscp 
- parallel-rsync is prsync 
- parallel-nuke is pnuke 
- parallel-slurp is pslurp 
```

## Copy node IPs to `/etc/hosts` for all nodes

- Copy `/etc/hosts` file to all nodes 

```bash 
$ for i in $(cat nodes);do scp /etc/hosts root@$i:/etc/hosts; done
```
Now, each node can reach each other with their name. 


## Install container runtime (Docker)

```bash 
# Install curl first to execute sh script from URL 
$ parallel-ssh -h nodes -i "apt-get install curl"

# run installation script from Github Gist 

$ parallel-ssh -h nodes -i " curl -sL https://gist.githubusercontent.com/mrturkmencom/74160d9d47a838dd3d60be036a5ea704/raw/d6d15afcfae596767ec74b5224f16de02498d855/docker-runtime-installation.sh | bash -"

```

After execution of installation script, all nodes will have Docker as container runtime application. Next step is to setup Kubernetes with similar approach. 


## Content of Installation Script

The content of the installation script is taken from [installation of container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) from official Kubernetes website. 

```bash 
#!/bin/sh

# Versions will be updated periodically 
# Last Update : 2020-07-03

# taken from https://kubernetes.io/docs/setup/production-environment/container-runtimes/
# run as ROOT 

apt-get update
apt-get upgrade -y
# (Install Docker CE)
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2  

# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
# Add the Docker apt repository:
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# Install Docker CE
apt-get update && apt-get install -y containerd.io=1.2.13-2 docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)

# Set up the Docker daemon
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d  

# Restart Docker
systemctl daemon-reload
systemctl restart docker

# Enable Docker
systemctl enable docker
```

Note that installation of Docker container runtime made by specifiying version, script will be updated respect to updates on official Kubernetes instructions. 

