---
layout: default
title: C15. Advanced Topics
published: true
nav_order: 14
parent: Introduction To Kubernetes- edX
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Introduction](#introduction)
- [Annotations](#annotations)
- [Jobs and CronJobs](#jobs-and-cronjobs)
- [Quota Management](#quota-management)
- [Autoscaling](#autoscaling)
- [DaemonSets](#daemonsets)
- [StatefulSets](#statefulsets)
- [Kubernetes Federation](#kubernetes-federation)
- [Custom Resources](#custom-resources)
- [Helm](#helm)
- [Security Contexts and Pod Security Policies](#security-contexts-and-pod-security-policies)
- [Network Policies](#network-policies)
- [Monitoring and Logging](#monitoring-and-logging)

# Introduction

So far, in this course, we have spent most of our time understanding the basic Kubernetes concepts and simple workflows to build a solid foundation. To support enterprise-class production workloads, Kubernetes also supports auto-scaling, rolling updates, rollbacks, quota management, authorization through RBAC, package management, network and security policies, etc. In this chapter, we will briefly cover a limited number of such advanced topics, but diving into details would be out of scope for this course.

# Annotations

With [Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/), we can attach arbitrary non-identifying metadata to any objects, in a key-value format:

```json
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```

Unlike Labels, annotations are not used to identify and select objects. Annotations can be used to:

- Store build/release IDs, PR numbers, git branch, etc.
- Phone/pager numbers of people responsible, or directory entries specifying where such information can be found
- Pointers to logging, monitoring, analytics, audit repositories, debugging tools, etc.
- Etc.

For example, while creating a Deployment, we can add a description as seen below:

```yaml 

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver
  annotations:
    description: Deployment based PoC dates 2nd May'2019
....
```

Annotations are displayed while describing an object:

```bash 
$ kubectl describe deployment webserver
Name:                webserver
Namespace:           default
CreationTimestamp:   Fri, 03 May 2019 05:10:38 +0530
Labels:              app=webserver
Annotations:         deployment.kubernetes.io/revision=1
                     description=Deployment based PoC dates 2nd May2019
...
```

# Jobs and CronJobs

A [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) creates one or more Pods to perform a given task. The Job object takes the responsibility of Pod failures. It makes sure that the given task is completed successfully. Once the task is complete, all the Pods have terminated automatically. Job configuration options include:

- **parallelism** - to set the number of pods allowed to run in parallel;
- **completions** - to set the number of expected completions;
- **activeDeadlineSeconds** - to set the duration of the Job;
- **backoffLimit** - to set the number of retries before Job is marked as failed;
- **ttlSecondsAfterFinished** - to delay the clean up of the finished Jobs.


Starting with the Kubernetes 1.4 release, we can also perform Jobs at scheduled times/dates with CronJobs, where a new Job object is created about once per each execution cycle. The [CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/) configuration options include:

- **startingDeadlineSeconds** - to set the deadline to start a Job if scheduled time was missed;
- **concurrencyPolicy** - to allow or forbid concurrent Jobs or to replace old Jobs with new ones. 

# Quota Management

When there are many users sharing a given Kubernetes cluster, there is always a concern for fair usage. A user should not take undue advantage. To address this concern, administrators can use the [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/) API resource, which provides constraints that limit aggregate resource consumption per Namespace.

We can set the following types of quotas per Namespace:

- **Compute Resource Quota**
  
  We can limit the total sum of compute resources (CPU, memory, etc.) that can be requested in a given Namespace.

- **Storage Resource Quota**

  We can limit the total sum of storage resources (PersistentVolumeClaims, requests.storage, etc.) that can be requested.

- **Object Count Quota**
  
  We can restrict the number of objects of a given type (pods, ConfigMaps, PersistentVolumeClaims, ReplicationControllers, Services, Secrets, etc.)


# Autoscaling

While it is fairly easy to manually scale a few Kubernetes objects, this may not be a practical solution for a production-ready cluster where hundreds or thousands of objects are deployed. We need a dynamic scaling solution which adds or removes objects from the cluster based on resource utilization, availability, and requirements. 

Autoscaling can be implemented in a Kubernetes cluster via controllers which periodically adjust the number of running objects based on single, multiple, or custom metrics. There are various types of autoscalers available in Kubernetes which can be implemented individually or combined for a more robust autoscaling solution:

- [Horizontal Pod Autoscaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details)
  
 HPA is an algorithm based controller [API resource](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/autoscaling/horizontal-pod-autoscaler.md#horizontalpodautoscaler-object) which automatically adjusts the number of replicas in a ReplicaSet, Deployment or Replication Controller based on CPU utilization.
              
- [Vertical Pod Autoscaler (VPA)](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/autoscaling/vertical-pod-autoscaler.md)
            
 VPA automatically sets Container resource requirements (CPU and memory) in a Pod and dynamically adjusts them in runtime, based on historical utilization data, current resource availability and real-time events.

- [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

 Cluster Autoscaler automatically re-sizes the Kubernetes cluster when there are insufficient resources available for new Pods expecting to be scheduled or when there are underutilized nodes in the cluster.

# DaemonSets

In cases when we need to collect monitoring data from all nodes, or to run a storage daemon on all nodes, then we need a specific type of Pod running on all nodes at all times. A [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) is the object that allows us to do just that. It is a critical controller API resource for multi-node Kubernetes clusters. The `kube-proxy` agent running as a Pod on every single node in the cluster is managed by a `DaemonSet`.

Whenever a node is added to the cluster, a Pod from a given DaemonSet is automatically created on it. Although it ensures an automated process, the DaemonSet's Pods are placed on nodes by the cluster's default Scheduler. When the node dies or it is removed from the cluster, the respective Pods are garbage collected. If a DaemonSet is deleted, all Pods it created are deleted as well.

A newer feature of the DaemonSet resource allows for its Pods to be scheduled only on specific nodes by configuring **nodeSelectors** and node **affinity** rules. Similar to Deployment resources, DaemonSets support rolling updates and rollbacks. 

# StatefulSets

The [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) controller is used for stateful applications which require a unique identity, such as name, network identifications, strict ordering, etc. For example, `MySQL cluster, etcd cluster`.

The `StatefulSet` controller provides identity and guaranteed ordering of deployment and scaling to Pods. Similar to Deployments, StatefulSets use ReplicaSets as intermediary Pod controllers and support rolling updates and rollbacks. 


# Kubernetes Federation

With [Kubernetes Cluster Federation](https://kubernetes.io/docs/concepts/cluster-administration/federation/) we can manage multiple Kubernetes clusters from a single control plane. We can sync resources across the federated clusters and have cross-cluster discovery. This allows us to perform Deployments across regions, access them using a global DNS record, and achieve High Availability. 

Although still an Alpha feature, the Federation is very useful when we want to build a hybrid solution, in which we can have one cluster running inside our private datacenter and another one in the public cloud, allowing us to avoid provider lock-in. We can also assign weights for each cluster in the Federation, to distribute the load based on custom rules. 

# Custom Resources

In Kubernetes, a **resource** is an API endpoint which stores a collection of API objects. For example, a Pod resource contains all the Pod objects.

Although in most cases existing Kubernetes resources are sufficient to fulfill our requirements, we can also create new resources using **custom resources**. With custom resources, we don't have to modify the Kubernetes source.

Custom resources are dynamic in nature, and they can appear and disappear in an already running cluster at any time.

To make a resource declarative, we must create and install a **custom controller**, which can interpret the resource structure and perform the required actions. Custom controllers can be deployed and managed in an already running cluster.

There are two ways to add custom resources:



- [Custom Resource Definitions (CRDs)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

This is the easiest way to add custom resources and it does not require any programming knowledge. However, building the custom controller would require some programming.
            
- [API Aggregation](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)
            
 For more fine-grained control, we can write API Aggregators. They are subordinate API servers which sit behind the primary API server. The primary API server acts as a proxy for all incoming API requests - it handles the ones based on its capabilities and proxies over the other requests meant for the subordinate API servers.

# Helm

To deploy an application, we use different Kubernetes manifests, such as Deployments, Services, Volume Claims, Ingress, etc. Sometimes, it can be tiresome to deploy them one by one. We can bundle all those manifests after templatizing them into a well-defined format, along with other metadata. Such a bundle is referred to as Chart. These Charts can then be served via repositories, such as those that we have for **rpm** and **deb** packages. 

[Helm](https://helm.sh/) is a package manager (analogous to yum and apt for Linux) for Kubernetes, which can install/update/delete those Charts in the Kubernetes cluster.

Helm has two components:

- A client called **helm**, which runs on your user's workstation
- A server called **tiller**, which runs inside your Kubernetes cluster.

The client helm connects to the server tiller to manage Charts. Charts submitted for Kubernetes are available [here](https://github.com/helm/charts).

# Security Contexts and Pod Security Policies

At times we need to define specific privileges and access control settings for Pods and Containers. [Security Contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) allow us to set Discretionary Access Control for object access permissions, privileged running, capabilities, security labels, etc. However, their effect is limited to the individual Pods and Containers where such context configuration settings are incorporated in the `spec` section.

In order to apply security settings to multiple Pods and Containers cluster-wide, we can define [Pod Security Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/). They allow more fine-grained security settings to control the usage of the host namespace, host networking and ports, file system groups, usage of volume types, enforce Container user and group ID, root privilege escalation, etc. 

# Network Policies

Kubernetes was designed to allow all Pods to communicate freely, without restrictions, with all other Pods in cluster Namespaces. In time it became clear that it was not an ideal design, and mechanisms needed to be put in place in order to restrict communication between certain Pods and applications in the cluster Namespace. [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) are sets of rules which define how Pods are allowed to talk to other Pods and resources inside and outside the cluster. Pods not covered by any **Network Policy** will continue to receive unrestricted traffic from any endpoint. 

**Network Policies** are very similar to typical Firewalls. They are designed to protect mostly assets located inside the Firewall but can restrict outgoing traffic as well based on sets of rules and policies.

The **Network Policy** API resource specifies `podSelectors`, `Ingress` and/or `Egress` **policyTypes**, and rules based on source and destination **ipBlocks** and **ports**. Very simplistic default allow or default deny policies can be defined as well. As a good practice, it is recommended to define a default deny policy to block all traffic to and from the Namespace, and then define sets of rules for specific traffic to be allowed in and out of the Namespace. 

Let's keep in mind that not all the networking solutions available for Kubernetes support Network Policies. Review the Pod-to-Pod Communication section from the Kubernetes Architecture chapter if needed. By default, **Network Policies** are namespaced API resources, but certain network plugins provide additional features so that Network Policies can be applied cluster-wide. 

# Monitoring and Logging

In Kubernetes, we have to collect resource usage data by Pods, Services, nodes, etc., to understand the overall resource consumption and to make decisions for scaling a given application. Two popular Kubernetes monitoring solutions are the Kubernetes Metrics Server and Prometheus.


- **Metrics Server**
- [Metrics Server](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#metrics-server) is a cluster-wide aggregator of resource usage data - a relatively new feature in Kubernetes. 
            
- [Prometheus](https://prometheus.io/)
 
  Prometheus, now part of [CNCF](https://www.cncf.io/) (Cloud Native Computing Foundation), can also be used to scrape the resource usage from different Kubernetes components and objects. Using its client libraries, we can also instrument the code of our application.


Another important aspect for troubleshooting and debugging is Logging, in which we collect the logs from different components of a given system. In Kubernetes, we can collect logs from different cluster components, objects, nodes, etc. Unfortunately, however, Kubernetes does not provide cluster-wide logging by default, therefore third party tools are required to centralize and aggregate cluster logs. The most common way to collect the logs is using [Elasticsearch](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/), which uses [fluentd](http://www.fluentd.org/) with custom configuration as an agent on the nodes. **fluentd** is an open source data collector, which is also part of CNCF.