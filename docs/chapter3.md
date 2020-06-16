---
layout: default
title: C3. Kubernetes
nav_order: 2
---

__Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Kubernetes](#kubernetes)
  - [From Borg to Kubernetes](#from-borg-to-kubernetes)
  - [Kubernetes Features I](#kubernetes-features-i)
  - [Kubernetes Features II](#kubernetes-features-ii)
  - [Why Use Kubernetes?](#why-use-kubernetes)
  - [Cloud Native Computing Foundation (CNCF)](#cloud-native-computing-foundation-cncf)


# Kubernetes 

Definition from [Kubernetes website](https://kubernetes.io/):

> "Kubernetes is an open-source system for automating deployment scaling, and management of containerized applications."


## From Borg to Kubernetes

According to the abstract of Google's [Borg paper](https://research.google.com/pubs/pub43438.html), published in 2015,

> "Google's Borg system is a cluster manager that runs hundreds of thousands of jobs, from many thousands of different applications, across a number of clusters each with up to tens of thousands of machines".

For more than a decade, Borg has been Google's secret, running its worldwide containerized workloads in production. Services we use from Google, such as Gmail, Drive, Maps, Docs, etc., they are all serviced using Borg. 

Some of the initial authors of Kubernetes were Google employees who have used Borg and developed it in the past. They poured in their valuable knowledge and experience while designing Kubernetes. Some of the features/objects of Kubernetes that can be traced back to Borg, or to lessons learned from it, are:

- API servers
- Pods
- IP-per-Pod
- Services
- Labels.

## Kubernetes Features I

Kubernetes offers a very rich set of features for container orchestration. Some of its fully supported features are:

- __Automatic bin packing__
    Kubernetes automatically schedules containers based on resource needs and constraints, to maximize utilization without sacrificing availability.

- __Self-healing__
    Kubernetes automatically replaces and reschedules containers from failed nodes. It kills and restarts containers unresponsive to health checks, based on existing rules/policy. It also prevents traffic from being routed to unresponsive containers.

- __Horizontal scaling__
    With Kubernetes applications are scaled manually or automatically based on CPU or custom metrics utilization.

- __Service discovery and Load balancing__
    Containers receive their own IP addresses from Kubernetes, while it assigns a single Domain Name System (DNS) name to a set of containers to aid in load-balancing requests across the containers of the set.

## Kubernetes Features II

- __Automated rollouts and rollbacks__
    Kubernetes seamlessly rolls out and rolls back application updates and configuration changes, constantly monitoring the application's health to prevent any downtime.

- __Secret and configuration management__
    Kubernetes manages secrets and configuration details for an application separately from the container image, in order to avoid a re-build of the respective image. Secrets consist of confidential information passed to the application without revealing the sensitive content to the stack configuration, like on GitHub.

- __Storage orchestration__
    Kubernetes automatically mounts software-defined storage (SDS) solutions to containers from local storage, external cloud providers, or network storage systems.

- __Batch execution__
    Kubernetes supports batch execution, long-running jobs, and replaces failed containers.


## Why Use Kubernetes?

In addition to its fully-supported features, Kubernetes is also portable and extensible. It can be deployed in many environments such as local or remote Virtual Machines, bare metal, or in public/private/hybrid/multi-cloud setups. It supports and it is supported by many 3rd party open source tools which enhance Kubernetes' capabilities and provide a feature-rich experience to its users.

Kubernetes' architecture is modular and pluggable. Not only that it orchestrates modular, decoupled microservices type applications, but also its architecture follows decoupled microservices patterns. Kubernetes' functionality can be extended by writing custom resources, operators, custom APIs, scheduling rules or plugins.

## Cloud Native Computing Foundation (CNCF)

The [Cloud Native Computing Foundation](https://www.cncf.io/) (CNCF) is one of the projects hosted by the [Linux Foundation](https://www.linuxfoundation.org/). CNCF aims to accelerate the adoption of containers, microservices, and cloud-native applications.

You can observe all graduated and incubating projects from [https://www.cncf.io/projects/](https://www.cncf.io/projects/)