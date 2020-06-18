---
layout: default
title: C7. Accessing Minikube
published: true
nav_order: 6
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Introduction](#introduction)
- [Accessing Minikube](#accessing-minikube)
  - [Accessing Minikube: Command Line Interface (CLI)](#accessing-minikube-command-line-interface-cli)

# Introduction


In this chapter, we will study different methods of accessing a Kubernetes cluster. There are a variety of external clients or custom scripts that provide cluster access for administration purposes. We will use `kubectl` as a CLI tool to access the Minikube Kubernetes cluster, the __Kubernetes Dashboard__ as a web-based user interface to interact with the cluster, and the `curl` command with the right credentials to access the cluster via APIs.

# Accessing Minikube

Any healthy running Kubernetes cluster can be accessed via any one of the following methods:

- Command Line Interface (CLI) tools and scripts
- Web-based User Interface (Web UI) from a web browser
- APIs from CLI or programmatically

These methods are applicable to all Kubernetes clusters. 

## Accessing Minikube: Command Line Interface (CLI)



[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) is the Kubernetes Command Line Interface (CLI) client to manage cluster resources and applications. It can be used standalone, or part of scripts and automation tools. Once all required credentials and cluster access points have been configured for kubectl it can be used remotely from anywhere to access a cluster. 

In later chapters, we will be using kubectl to deploy applications, manage and configure Kubernetes resources.

