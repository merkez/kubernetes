---
layout: default
title: C8. Kubernetes Building Blocks
published: true
nav_order: 7
parent: Introduction To Kubernetes- edX
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__


- [Introduction](#introduction)
  - [Kubernetes Object Model](#kubernetes-object-model)
- [Pods](#pods)
- [Labels](#labels)
- [Label Selectors](#label-selectors)
- [ReplicationControllers](#replicationcontrollers)
- [ReplicaSets I](#replicasets-i)
- [ReplicaSets II](#replicasets-ii)
- [ReplicaSets III](#replicasets-iii)
- [Deployments I](#deployments-i)
- [Deployments II](#deployments-ii)
- [Deployments III](#deployments-iii)
- [Namespaces](#namespaces)
  - [Deployment Rolling Update and Rollback](#deployment-rolling-update-and-rollback)

# Introduction 

In this chapter, we will explore the Kubernetes object model and discuss some of its fundamental building blocks, such as **Pods, ReplicaSets, Deployments, Namespaces,** etc. We will also discuss the essential role Labels and Selectors play in a microservices driven architecture as they group decoupled objects together.

## Kubernetes Object Model

Kubernetes has a very rich object model, representing different persistent entities in the Kubernetes cluster. Those entities describe:

- What containerized applications we are running and on which node
- Application resource consumption
- Different policies attached to applications, like restart/upgrade policies, fault tolerance, etc.

With each object, we declare our intent spec section. The Kubernetes system manages the `status` section for objects, where it records the actual state of the object. At any given point in time, the Kubernetes Control Plane tries to match the object's actual state to the object's desired state.

Examples of Kubernetes objects are Pods, ReplicaSets, Deployments, Namespaces, etc. We will explore them next.

When creating an object, the object's configuration data section from below the spec field has to be submitted to the Kubernetes API server. The spec section describes the desired state, along with some basic information, such as the object's name. The API request to create an object must have the spec section, as well as other details. Although the API server accepts object definition files in a JSON format, most often we provide such files in a YAML format which is converted by kubectl in a JSON payload and sent to the API server.

Below is an example of a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) object's configuration in YAML format:

```bash 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.11
        ports:
        - containerPort: 80
```

The `apiVersion` field is the first required field, and it specifies the API endpoint on the API server which we want to connect to; it must match an existing version for the object type defined. The second required field is kind, specifying the object type - in our case it is Deployment, but it can be Pod, Replicaset, Namespace, Service, etc. The third required field metadata, holds the object's basic information, such as name, labels, namespace, etc. Our example shows two spec fields (spec and spec.template.spec). The fourth required field spec marks the beginning of the block defining the desired state of the Deployment object. In our example, we want to make sure that 3 Pods are running at any given time. The Pods are created using the Pods Template defined in spec.template. A nested object, such as the Pod being part of a Deployment, retains its `metadata` and `spec` and loses the `apiVersion` and `kind` - both being replaced by template. In `spec.template.spec`, we define the desired state of the Pod. Our Pod creates a single container running the `nginx:1.15.11` image from [Docker Hub](https://hub.docker.com/_/nginx).

# Pods 

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) is the smallest and simplest Kubernetes object. It is the unit of deployment in Kubernetes, which represents a single instance of the application. A Pod is a logical collection of one or more containers, which:

- Are scheduled together on the same host with the Pod
- Share the same network namespace
- Have access to mount the same external storage (volumes).

![Pods](/assets/images/intro-kubernetes/Pods.png)

Pods are ephemeral in nature, and they do not have the capability to self-heal by themselves. That is the reason they are used with controllers which handle Pods' replication, fault tolerance, self-healing, etc. Examples of controllers are Deployments, ReplicaSets, ReplicationControllers, etc. We attach a nested Pod's specification to a controller object using the Pod Template, as we have seen in the previous section.

Below is an example of a Pod object's configuration in `YAML` format:

```bash 
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.11
    ports:
    - containerPort: 80
```
The `apiVersion` field must specify `v1` for the `Pod` object definition. The second required field is `kind` specifying the `Pod` object type. The third required field `metadata`, holds the object's name and label. The fourth required field `spec` marks the beginning of the block defining the desired state of the Pod object - also named the PodSpec. Our Pod creates a single container running the `nginx:1.15.11` image from [Docker Hub](https://hub.docker.com/_/nginx).


# Labels

[Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) are key-value pairs attached to Kubernetes objects (e.g. Pods, ReplicaSets). Labels are used to organize and select a subset of objects, based on the requirements in place. Many objects can have the same Label(s). Labels do not provide uniqueness to objects. Controllers use Labels to logically group together decoupled objects, rather than using objects' names or IDs.

![Labels](/assets/images/intro-kubernetes/Labels.png)


In the image above, we have used two Label keys: `app` and `env`. Based on our requirements, we have given different values to our four Pods. The Label env=dev logically selects and groups the top two Pods, while the Label `app=frontend` logically selects and groups the left two Pods. We can select one of the four Pods - bottom left, by selecting two Labels: app=frontend and env=qa.

# Label Selectors

Controllers use Label Selectors to select a subset of objects. Kubernetes supports two types of Selectors:

- **Equality-Based Selectors**
  Equality-Based Selectors allow filtering of objects based on Label keys and values. Matching is achieved using the =, == (equals, used interchangeably), or != (not equals) operators. For example, with env==dev or env=dev we are selecting the objects where the env Label key is set to value dev. 

- **Set-Based Selectors**
  Set-Based Selectors allow filtering of objects based on a set of values. We can use in, notin operators for Label values, and exist/does not exist operators for Label keys. For example, with env in (dev,qa) we are selecting objects where the env Label is set to either dev or qa; with !app we select objects with no Label key app.


![SELECTORS](/assets/images/intro-kubernetes/Selector.png)

# ReplicationControllers  

Although no longer a recommended method, a [ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) is a controller that ensures a specified number of replicas of a Pod is running at any given time. If there are more Pods than the desired count, a replication controller would terminate the extra Pods, and, if there are fewer Pods, then the replication controller would create more Pods to match the desired count. Generally, we don't deploy a Pod independently, as it would not be able to re-start itself if terminated in error. The recommended method is to use some type of replication controllers to create and manage Pods. 


The default controller is a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) which configures a [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) to manage Pods' lifecycle.


# ReplicaSets I

A [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is the next-generation ReplicationController. ReplicaSets support both equality- and set-based selectors, whereas ReplicationControllers only support equality-based Selectors. Currently, this is the only difference.

With the help of the ReplicaSet, we can scale the number of Pods running a specific container application image. Scaling can be accomplished manually or through the use of an autoscaler.

Next, you can see a graphical representation of a ReplicaSet, where we have set the replica count to 3 for a Pod.

![ReplicaSet](/assets/images/intro-kubernetes/Replica_Set1.png)


# ReplicaSets II

Now, let's continue with the same example and assume that one of the Pods is forced to terminate (due to insufficient resources, timeout, etc.), and the current state is no longer matching the desired state.

![ReplicaSet2](/assets/images/intro-kubernetes/Replica_Set2.png)

**Current State and Desired State Are Different**

# ReplicaSets III

The ReplicaSet will detect that the current state is no longer matching the desired state. The ReplicaSet will create an additional Pod, thus ensuring that the current state matches the desired state.

![ReplicaSet3](/assets/images/intro-kubernetes/ReplicaSet3.png)


ReplicaSets can be used independently as Pod controllers but they only offer a limited set of features. A set of complementary features are provided by Deployments, the recommended controllers for the orchestration of Pods. Deployments manage the creation, deletion, and updates of Pods. A Deployment automatically creates a ReplicaSet, which then creates a Pod. There is no need to manage ReplicaSets and Pods separately, the Deployment will manage them on our behalf.


# Deployments I

Deployment objects provide declarative updates to Pods and ReplicaSets. The DeploymentController is part of the master node's controller manager, and it ensures that the current state always matches the desired state. It allows for seamless application updates and downgrades through **rollouts** and **rollbacks**, and it directly manages its ReplicaSets for application scaling. 


In the following example, a new **Deployment** creates **ReplicaSet A** which then creates **3 Pods**, with each Pod Template configured to run one **nginx:1.7.9** container image. In this case, the **ReplicaSet A** is associated with **nginx:1.7.9** representing a state of the **Deployment**. This particular state is recorded as **Revision 1**.

![Deployment Updates](/assets/images/intro-kubernetes/Deployment_Updated.png)


# Deployments II

Now, in the Deployment, we change the Pods' Template and we update the container image from **nginx:1.7.9** to **nginx:1.9.1**. The **Deployment** triggers a new **ReplicaSet B** for the new container image versioned **1.9.1** and this association represents a new recorded state of the **Deployment, Revision 2**. 

The seamless transition between the two ReplicaSets, from **ReplicaSet A** with 3 Pods versioned **1.7.9** to the new ReplicaSet B with 3 new Pods versioned 1.9.1, or from Revision 1 to Revision 2, is a **Deployment rolling update**. 

A **rolling update** is triggered when we update the Pods Template for a deployment. Operations like scaling or labeling the deployment do not trigger a rolling update, thus do not change the Revision number.


Once the rolling update has completed, the **Deployment** will show both **ReplicaSets** A and B, where A is scaled to 0 (zero) Pods, and B is scaled to 3 Pods. This is how the Deployment records its prior state configuration settings, as Revisions. 

![Replika Set](/assets/images/intro-kubernetes/ReplikaSet_B.png)


# Deployments III

Once __ReplicaSet B__ and its 3 Pods versioned 1.9.1 are ready, the **Deployment** starts actively managing them. However, the Deployment keeps its prior configuration states saved as Revisions which play a key factor in the **rollback** capability of the Deployment - returning to a prior known configuration state. In our example, if the performance of the new nginx:1.9.1 is not satisfactory, the Deployment can be rolled back to a prior Revision, in this case from Revision 2 back to Revision 1 running nginx:1.7.9. 

![Deployment Points to ReplikaSet](/assets/images/intro-kubernetes/Deployment_points_to_ReplikaSet.png)


# Namespaces

If multiple users and teams use the same Kubernetes cluster we can partition the cluster into virtual sub-clusters using Namespaces. The names of the resources/objects created inside a Namespace are unique, but not across Namespaces in the cluster.

To list all the Namespaces, we can run the following command:

```bash 
$ kubectl get namespaces
NAME              STATUS       AGE
default           Active       11h
kube-node-lease   Active       11h
kube-public       Active       11h
kube-system       Active       11h
```

Generally, Kubernetes creates four default Namespaces: **kube-system, kube-public, kube-node-lease**, and default. The **kube-system** Namespace contains the objects created by the Kubernetes system, mostly the control plane agents. The default Namespace contains the objects and resources created by administrators and developers. By default, we connect to the default Namespace. kube-public is a special Namespace, which is unsecured and readable by anyone, used for special purposes such as exposing public (non-sensitive) information about the cluster. The newest Namespace is kube-node-lease which holds node lease objects used for node heartbeat data. Good practice, however, is to create more Namespaces to virtualize the cluster for users and developer teams.

With [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/), we can divide the cluster resources within Namespaces. We will briefly cover resource quotas in one of the future chapters.


## Deployment Rolling Update and Rollback

<iframe width="736" height="450" src="https://www.youtube-nocookie.com/embed/KaclgYWv-04" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>