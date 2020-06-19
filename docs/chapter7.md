---
layout: default
title: C7. Accessing Minikube
published: true
nav_order: 6
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Introduction](#introduction)
- [Accessing Minikube](#accessing-minikube)
  - [Command Line Interface (CLI)](#command-line-interface-cli)
  - [Web-based User Interface (Web UI)](#web-based-user-interface-web-ui)
  - [APIs](#apis)
  - [kubectl](#kubectl)
  - [Installing kubectl on Linux](#installing-kubectl-on-linux)
  - [Installing kubectl on macOS](#installing-kubectl-on-macos)
- [kubectl Configuration File](#kubectl-configuration-file)
  - [Installing kubectl CLI Client](#installing-kubectl-cli-client)
  - [Kubernetes Dashboard](#kubernetes-dashboard)
  - [The 'kubectl proxy' Command](#the-kubectl-proxy-command)
- [APIs - with 'kubectl proxy'](#apis---with-kubectl-proxy)
- [APIs - without 'kubectl proxy'](#apis---without-kubectl-proxy)
- [Accessing the Cluster with Dashboard and Query APIs with CLI(Demo)](#accessing-the-cluster-with-dashboard-and-query-apis-with-clidemo)

# Introduction


In this chapter, we will study different methods of accessing a Kubernetes cluster. There are a variety of external clients or custom scripts that provide cluster access for administration purposes. We will use `kubectl` as a CLI tool to access the Minikube Kubernetes cluster, the __Kubernetes Dashboard__ as a web-based user interface to interact with the cluster, and the `curl` command with the right credentials to access the cluster via APIs.

# Accessing Minikube

Any healthy running Kubernetes cluster can be accessed via any one of the following methods:

- Command Line Interface (CLI) tools and scripts
- Web-based User Interface (Web UI) from a web browser
- APIs from CLI or programmatically

These methods are applicable to all Kubernetes clusters. 

## Command Line Interface (CLI)

[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) is the Kubernetes Command Line Interface (CLI) client to manage cluster resources and applications. It can be used standalone, or part of scripts and automation tools. Once all required credentials and cluster access points have been configured for kubectl it can be used remotely from anywhere to access a cluster. 

## Web-based User Interface (Web UI)

The [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) provides a __Web-Based User Interface (Web UI)__ to interact with a Kubernetes cluster to manage resources and containerized applications. In one of the later chapters, we will be using it to deploy a containerized application.

## APIs

As we know, Kubernetes has the __API server__, and operators/users connect to it from the external world to interact with the cluster. Using both CLI and Web UI, we can connect to the API server running on the master node to perform different operations. We can directly connect to the API server using its API endpoints and send commands to it, as long as we can access the master node and have the right credentials.

Below, we can see a part of the HTTP API space of Kubernetes:

![API Server Space](/assets/images/intro-kubernetes/api-server-space_.jpg)


HTTP API space of Kubernetes can be divided into three independent groups:

- **Core Group (/api/v1)**
This group includes objects such as Pods, Services, nodes, namespaces, configmaps, secrets, etc.
- **Named Group**
This group includes objects in /apis/$NAME/$VERSION format. These different API versions imply different levels of stability and support:
_Alpha level_ - it may be dropped at any point in time, without notice. For example, /apis/batch/v2alpha1.
_Beta level_ - it is well-tested, but the semantics of objects may change in incompatible ways in a subsequent beta or stable release. For example, /apis/certificates.k8s.io/v1beta1.
_Stable level_ - appears in released software for many subsequent versions. For example, /apis/networking.k8s.io/v1.
- **System-wide**
This group consists of system-wide API endpoints, like /healthz, /logs, /metrics, /ui, etc.

We can either connect to an API server directly via calling the respective API endpoints or via the CLI/Web UI.

## kubectl

`kubectl` is generally installed before installing Minikube, but we can also install it after. Once installed, `kubectl` receives its configuration automatically for Minikube Kubernetes cluster access. However, in other Kubernetes cluster setups, we may need to configure the cluster access points and certificates required by `kubectl` to access the cluster.


There are different methods that can be used to install `kubectl`, which are mentioned in the Kubernetes documentation. For best results, it is recommended to keep `kubectl` at the same version with the Kubernetes run by Minikube - at the time the course was written the latest stable release was v1.14.1. Next, we will look at a few steps to install it on Linux, macOS, and Windows systems.

## Installing kubectl on Linux

To install `kubectl` on Linux, follow the instruction below:

```bash 
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

__NOTE:__ To download and setup a specific version of **kubectl** (such as v1.14.1), issue the following command:

```bash 
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

## Installing kubectl on macOS

There are two ways to install `kubectl` on macOS: manually and using the Homebrew package manager. Next, we will provide instructions for both methods.

To manually install `kubectl`, download the latest stable kubectl binary, make it executable and move it to the PATH with the following command:

```bash 

$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/

```
NOTE: To download and setup a specific version of kubectl (such as v1.14.1), issue the following command:

```bash 

$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.1/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

To install kubectl with [Homebrew package manager](https://brew.sh/), issue the following command:

```bash 

$ brew install kubernetes-cli
```

# kubectl Configuration File

To access the Kubernetes cluster, the `kubectl` client needs the master node endpoint and appropriate credentials to be able to interact with the API server running on the master node. While starting Minikube, the startup process creates, by default, a configuration file, config, inside the.kube directory (often referred to as the `dot-kube-config` file), which resides in the user's home directory. The configuration file has all the connection details required by kubectl. By default, the kubectl binary parses this file to find the master node's connection endpoint, along with credentials. To look at the connection details, we can either see the content of the `~/.kube/config` file (on Linux) or run the following command:

```bash 
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/student/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/student/.minikube/client.crt
    client-key: /home/student/.minikube/client.key
```

Once kubectl is installed, we can get information about the Minikube cluster with the `kubectl cluster-info `command: 

```bash 

$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443//api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

You can find more details about the `kubectl` command line options [here](https://kubernetes.io/docs/reference/kubectl/overview/).

Although for the Kubernetes cluster installed by Minikube the `~/.kube/config` file gets created automatically, this is not the case for Kubernetes clusters installed by other tools. In other cases, the config file has to be created manually and sometimes re-configured to suit various networking and client/server setups.


## Installing kubectl CLI Client


<iframe width="736" height="450" src="https://www.youtube-nocookie.com/embed/HHnHfA3cDGc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



## Kubernetes Dashboard

As mentioned earlier, the [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) provides a web-based user interface for Kubernetes cluster management. To access the dashboard from Minikube, we can use the minikube dashboard command, which opens a new tab on our web browser, displaying the Kubernetes Dashboard:

```bash 
$ minikube dashboard
```

![Kubernetes Dashboard](/assets/images/intro-kubernetes/kubernetes_dashboard.png)


**NOTE:** In case the browser is not opening another tab and does not display the Dashboard as expected, verify the output in your terminal as it may display a link for the Dashboard (together with some Error messages). Copy and paste that link in a new tab of your browser. Depending on your terminal's features you may be able to just click or right-click the link to open directly in the browser. The link may look similar to:

http://127.0.0.1:37751/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/

Chances are that the only difference is the PORT number, which above is 37751. Your port number may be different.

After a logout/login or a reboot of your workstation the normal behavior should be expected (where the minikube dashboard command directly opens a new tab in your browser displaying the Dashboard).

## The 'kubectl proxy' Command

Issuing the `kubectl proxy` command, kubectl authenticates with the API server on the master node and makes the Dashboard available on a slightly different URL than the one earlier, this time through the proxy port 8001.

First, we issue the `kubectl proxy` command:

```bash 
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

It locks the terminal for as long as the proxy is running. With the proxy running we can access the Dashboard over the new URL (just click on it below - it should work on your workstation). Once we stop the proxy (with CTRL + C) the Dashboard is no longer accessible.

```bash 
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy/#!/overview?namespace=default 

```


![Dashboard](/assets/images/intro-kubernetes/kub-dash-8001.png)


# APIs - with 'kubectl proxy'

When `kubectl proxy` is running, we can send requests to the API over the localhost on the proxy port 8001 (from another terminal, since the proxy locks the first terminal):

```bash 
$ curl http://localhost:8001/
{
 "paths": [
   "/api",
   "/api/v1",
   "/apis",
   "/apis/apps",
   ......
   ......
   "/logs",
   "/metrics",
   "/openapi/v2",
   "/version"
 ]
}
```

With the above `curl` request, we requested all the API endpoints from the API server. Clicking on the link above (in the curl command), it will open the same listing output in a browser tab.

We can explore every single path combination with `curl` or in a browser, such as:

```raw
- http://localhost:8001/api/v1
- http://localhost:8001/apis/apps/v1
- http://localhost:8001/healthz
- http://localhost:8001/metrics
```


# APIs - without 'kubectl proxy'


When not using the `kubectl proxy, we need to authenticate to the API server when sending API requests. We can authenticate by providing a **Bearer Token** when issuing a curl, or by providing a set of **keys** and **certificates**.

A **Bearer Token** is an access token which is generated by the authentication server (the API server on the master node) and given back to the client. Using that token, the client can connect back to the Kubernetes API server without providing further authentication details, and then, access resources.

- Get the token:

```bash 
$ TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")
```

- **Get the API server endpoint:**

```bash 
$ APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
```

**Confirm that the `APISERVER` stored the same IP as the Kubernetes master IP by issuing the following 2 commands and comparing their outputs:**

```bash 
$ echo $APISERVER
https://192.168.99.100:8443

$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443 ..
```

**Access the API server using the curl command, as shown below:**

```bash 

$ curl $APISERVER --header "Authorization: Bearer $TOKEN" --insecure
{
 "paths": [
   "/api",
   "/api/v1",
   "/apis",
   "/apis/apps",
   ......
   ......
   "/logs",
   "/metrics",
   "/openapi/v2",
   "/version"
 ]
}

```

Instead of the `access token`, we can extract the client certificate, client key, and certificate authority data from the `.kube/config` file. Once extracted, they are encoded and then passed with a curl command for authentication. The new curl command looks similar to:

```bash 
$ curl $APISERVER --cert encoded-cert --key encoded-key --cacert encoded-ca

```
# Accessing the Cluster with Dashboard and Query APIs with CLI(Demo)

<iframe width="736" height="450" src="https://www.youtube-nocookie.com/embed/ovypydMNAXA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>