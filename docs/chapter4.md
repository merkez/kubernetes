---
layout: default
title: C4. Kubernetes Architecture
nav_order: 3
---


⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Kubernetes Architecture](#kubernetes-architecture)
  - [Master Node](#master-node)
    - [Master Node Components: API Server](#master-node-components-api-server)
    - [Master Node Components: Scheduler](#master-node-components-scheduler)
    - [Master Node Components: Controller Managers](#master-node-components-controller-managers)
    - [Master Node Components: etcd](#master-node-components-etcd)
  - [Worker Node](#worker-node)
    - [Worker Node Components: Container Runtime](#worker-node-components-container-runtime)
    - [Worker Node Components: kubelet](#worker-node-components-kubelet)
    - [Worker Node Components: kubelet - CRI shims](#worker-node-components-kubelet---cri-shims)
    - [Worker Node Components: kube-proxy](#worker-node-components-kube-proxy)
    - [Worker Node Components: Addons](#worker-node-components-addons)
  - [Networking Challenges](#networking-challenges)
  - [Container-to-Container Communication Inside Pods](#container-to-container-communication-inside-pods)
  - [Pod-to-Pod Communication Across Nodes](#pod-to-pod-communication-across-nodes)
  - [Pod-to-External World Communication](#pod-to-external-world-communication)

# Kubernetes Architecture

At a very high level, Kubernetes has the following main components:

- One or more __master nodes__
- One or more __worker nodes__
- Distributed key-value store, such as [etcd](https://etcd.io/).

![Kubernetes Architecture](../../assets/images/intro-kubernetes/kube-arch.png)

## Master Node

The __master node__ provides a running environment for the control plane responsible for managing the state of a Kubernetes cluster, and it is the brain behind all operations inside the cluster. The control plane components are agents with very distinct roles in the cluster's management. In order to communicate with the Kubernetes cluster, users send requests to the master node via a Command Line Interface (CLI) tool, a Web User-Interface (Web UI) Dashboard, or Application Programming Interface (API).


![Kubernetes Master Node](../../assets/images/intro-kubernetes/master_node.png)

It is important to keep the control plane running at all costs. Losing the control plane may introduce downtimes, causing service disruption to clients, with possible loss of business. To ensure the control plane's fault tolerance, master node replicas are added to the cluster, configured in High-Availability (HA) mode. While only one of the master node replicas actively manages the cluster, the control plane components stay in sync across the master node replicas. This type of configuration adds resiliency to the cluster's control plane, should the active master node replica fail.

To persist the Kubernetes cluster's state, all cluster configuration data is saved to [etcd](https://github.com/etcd-io). However, __etcd__ is a distributed key-value store which only holds cluster state related data, no client workload data. etcd is configured on the master node ([stacked](https://kubernetes.io/docs/setup/independent/ha-topology/#stacked-etcd-topology)) or on its dedicated host ([external](https://kubernetes.io/docs/setup/independent/ha-topology/#external-etcd-topology)) to reduce the chances of data store loss by decoupling it from the control plane agents.

When stacked, HA master node replicas ensure etcd resiliency as well. Unfortunately, that is not the case of external etcds, when the etcd hosts have to be separately replicated for HA mode configuration.

A master node has the following components:

- API server
- Scheduler
- Controller managers
- etcd.

### Master Node Components: API Server

All the administrative tasks are coordinated by the __kube-apiserver__, a central control plane component running on the master node. The API server intercepts RESTful calls from users, operators and external agents, then validates and processes them. During processing the API server reads the Kubernetes cluster's current state from the etcd, and after a call's execution, the resulting state of the Kubernetes cluster is saved in the distributed key-value data store for persistence. The API server is the only master plane component to talk to the etcd data store, both to read and to save Kubernetes cluster state information from/to it - acting as a middle-man interface for any other control plane agent requiring to access the cluster's data store.

The API server is highly configurable and customizable. It also supports the addition of custom API servers, when the primary API server becomes a proxy to all secondary custom API servers and routes all incoming RESTful calls to them based on custom defined rules.

### Master Node Components: Scheduler

The role of the __kube-scheduler__ is to assign new objects, such as pods, to nodes. During the scheduling process, decisions are made based on current Kubernetes cluster state and new object's requirements. The scheduler obtains from etcd, via the API server, resource usage data for each worker node in the cluster. The scheduler also receives from the API server the new object's requirements which are part of its configuration data. Requirements may include constraints that users and operators set, such as scheduling work on a node labeled with __disk==ssd__ key/value pair. The scheduler also takes into account Quality of Service (QoS) requirements, data locality, affinity, anti-affinity, taints, toleration, etc.

The scheduler is highly configurable and customizable. Additional custom schedulers are supported, then the object's configuration data should include the name of the custom scheduler expected to make the scheduling decision for that particular object; if no such data is included, the default scheduler is selected instead.

A scheduler is extremely important and quite complex in a multi-node Kubernetes cluster. In a single-node Kubernetes cluster, such as the one explored later in this course, the scheduler's job is quite simple.

### Master Node Components: Controller Managers

- **controller managers**

    The __controller managers__ are control plane components on the master node running controllers to regulate the state of the Kubernetes cluster. Controllers are watch-loops continuously running and comparing the cluster's desired state (provided by objects' configuration data) with its current state (obtained from etcd data store via the API server). In case of a mismatch corrective action is taken in the cluster until its current state matches the desired state.

- **kube-controller-manager**

    The __kube-controller-manager__ runs controllers responsible to act when nodes become unavailable, to ensure pod counts are as expected, to create endpoints, service accounts, and API access tokens.

- **cloud-controller-manager**

    The __cloud-controller-manager__ runs controllers responsible to interact with the underlying infrastructure of a cloud provider when nodes become unavailable, to manage storage volumes when provided by a cloud service, and to manage load balancing and routing.

### Master Node Components: etcd

**etcd** is a distributed key-value data store used to persist a Kubernetes cluster's state. New data is written to the data store only by appending to it, data is never replaced in the data store. Obsolete data is compacted periodically to minimize the size of the data store.

Out of all the control plane components, only the API server is able to communicate with the etcd data store.

etcd's CLI management tool provides backup, snapshot, and restore capabilities which come in handy especially for a single etcd instance Kubernetes cluster - common in Development and learning environments. However, in Stage and Production environments, it is extremely important to replicate the data stores in HA mode, for cluster configuration data resiliency.

etcd is based on the [Raft Consensus Algorithm](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14) which allows a collection of machines to work as a coherent group that can survive the failures of some of its members. At any given time, one of the nodes in the group will be the master, and the rest of them will be the followers. Any node can be treated as a master.

![Master and Followers](../../assets/images/intro-kubernetes/master-followers.png)


**etcd** is written in the **Go programming language**. In Kubernetes, besides storing the cluster state, etcd is also used to store configuration details such as subnets, ConfigMaps, Secrets, etc.

## Worker Node

Could be useful to check out [**POD**](https://kubernetes.io/docs/concepts/workloads/pods/pod/) definition.

A worker node provides a running environment for client applications. Though containerized microservices, these applications are encapsulated in Pods, controlled by the cluster control plane agents running on the master node. Pods are scheduled on worker nodes, where they find required compute, memory and storage resources to run, and networking to talk to each other and the outside world. A Pod is the smallest scheduling unit in Kubernetes. It is a logical collection of one or more containers scheduled together

![Worker Node](../../assets/images/intro-kubernetes/worker_node.png)

Also, to access the applications from the external world, we connect to worker nodes and not to the master node

A worker node has the following components:

- Container runtime
- kubelet
- kube-proxy
- Addons for DNS, Dashboard, cluster-level monitoring and logging.

### Worker Node Components: Container Runtime

Although Kubernetes is described as a "container orchestration engine", it does not have the capability to directly handle containers. In order to run and manage a container's lifecycle, Kubernetes requires a **container runtime** on the node where a Pod and its containers are to be scheduled. Kubernetes supports many container runtimes:

- [**Docker**](https://www.docker.com/) - although a container platform which uses containerd as a container runtime, it is the most widely used container runtime with Kubernetes

- [**CRI-O**](https://cri-o.io/) - a lightweight container runtime for Kubernetes, it also supports Docker image registries

- [**containerd**](https://containerd.io/) - a simple and portable container runtime providing robustness

- [**rkt**](https://github.com/rkt/rkt) - a pod-native container engine, it also runs Docker images

- [**rktlet**](https://github.com/kubernetes-incubator/rktlet) - a Kubernetes [Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) (CRI) implementation using rkt

### Worker Node Components: kubelet

The **kubelet** is an agent running on each node and communicates with the control plane components from the master node. It receives Pod definitions, primarily from the API server, and interacts with the container runtime on the node to run containers associated with the Pod. It also monitors the health of the Pod's running containers.

The kubelet connects to the container runtime using [Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) (CRI). CRI consists of protocol buffers, gRPC API, and libraries.

![Container Runtime Interface ](../../assets/images/intro-kubernetes/cri.png)

As shown above, the kubelet acting as grpc client connects to the CRI shim acting as grpc server to perform container and image operations. CRI implements two services: **ImageService** and **RuntimeService**. The **ImageService** is responsible for all the image-related operations, while the **RuntimeService** is responsible for all the Pod and container-related operations.

Container runtimes used to be hard-coded in Kubernetes, but with the development of CRI, Kubernetes is more flexible now and uses different container runtimes without the need to recompile. Any container runtime that implements CRI can be used by Kubernetes to manage Pods, containers, and container images.

### Worker Node Components: kubelet - CRI shims


Below you will find some examples of CRI shims:

- **dockershim**

    With dockershim, containers are created using Docker installed on the worker nodes. Internally, Docker uses containerd to create and manage containers.

    ![dockershim](../../assets/images/intro-kubernetes/dockershim.png)

- **cri-containerd**

    With cri-containerd, we can directly use Docker's smaller offspring containerd to create and manage containers.

    ![cri-containerd](../../assets/images/intro-kubernetes/cri-containerd.png)

- **CRI-O**

    CRI-O enables using any Open Container Initiative (OCI) compatible runtimes with Kubernetes. At the time this course was created, CRI-O supported runC and Clear Containers as container runtimes. However, in principle, any OCI-compliant runtime can be plugged-in.

    ![CRI-O](../../assets/images/intro-kubernetes/crio.png)

### Worker Node Components: kube-proxy

The **kube-proxy** is the network agent which runs on each node responsible for dynamic updates and maintenance of all networking rules on the node. It abstracts the details of Pods networking and forwards connection requests to Pods.


### Worker Node Components: Addons

**Addons** are cluster features and functionality not yet available in Kubernetes, therefore implemented through 3rd-party pods and services.


- **DNS** - cluster DNS is a DNS server required to assign DNS records to Kubernetes objects and resources
- **Dashboard** - a general purposed web-based user interface for cluster management
- **Monitoring** - collects cluster-level container metrics and saves them to a central data store
- **Logging** - collects cluster-level container logs and saves them to a central log store for analysis.

## Networking Challenges

Decoupled microservices based applications rely heavily on networking in order to mimic the tight-coupling once available in the monolithic era. Networking, in general, is not the easiest to understand and implement. Kubernetes is no exception - as a containerized microservices orchestrator is needs to address 4 distinct networking challenges:

- Container-to-container communication inside Pods
- Pod-to-Pod communication on the same node and across cluster nodes
- Pod-to-Service communication within the same namespace and across cluster namespaces
- External-to-Service communication for clients to access applications in a cluster.

All these networking challenges must be addressed before deploying a Kubernetes cluster.

## Container-to-Container Communication Inside Pods

Making use of the underlying host operating system's kernel features, a container runtime creates an isolated network space for each container it starts. On Linux, that isolated network space is referred to as a network namespace. A **network namespace** is shared across containers, or with the host operating system.

When a Pod is started, a network namespace is created inside the Pod, and all containers running inside the Pod will share that network namespace so that they can talk to each other via localhost.


## Pod-to-Pod Communication Across Nodes

In a Kubernetes cluster Pods are scheduled on nodes randomly. Regardless of their host node, Pods are expected to be able to communicate with all other Pods in the cluster, all this without the implementation of Network Address Translation (NAT). This is a fundamental requirement of any networking implementation in Kubernetes.

The Kubernetes network model aims to reduce complexity, and it treats Pods as VMs on a network, where each VM receives an IP address - thus each Pod receiving an IP address. This model is called **"IP-per-Pod"** and ensures Pod-to-Pod communication, just as VMs are able to communicate with each other.


Let's not forget about containers though. They share the Pod's network namespace and must coordinate ports assignment inside the Pod just as applications would on a VM, all while being able to communicate with each other on **localhost** - inside the Pod. 

However, containers are integrated with the overall Kubernetes networking model through the use of the [Container Network Interface (CNI)](https://github.com/containernetworking/cni) supported by [CNI plugins](https://github.com/containernetworking/cni#3rd-party-plugins).

CNI is a set of a specification and libraries which allow plugins to configure the networking for containers. While there are a few [core plugins](https://github.com/containernetworking/plugins#plugins), most CNI plugins are 3rd-party Software Defined Networking (SDN) solutions implementing the Kubernetes networking model. In addition to addressing the fundamental requirement of the networking model, some networking solutions offer support for Network Policies. [Flannel](https://github.com/coreos/flannel/), [Weave](https://www.weave.works/oss/net/), [Calico](https://www.projectcalico.org/) are only a few of the SDN solutions available for Kubernetes clusters.

![Container Network Interface CNI](../../assets/images/intro-kubernetes/cni.png)


The container runtime offloads the IP assignment to CNI, which connects to the underlying configured plugin, such as Bridge or MACvlan, to get the IP address. Once the IP address is given by the respective plugin, CNI forwards it back to the requested container runtime.

For more details, you can explore the [Kubernetes documentation](https://kubernetes.io/docs/concepts/cluster-administration/networking/).


## Pod-to-External World Communication

For a successfully deployed containerized applications running in Pods inside a Kubernetes cluster, it requires accessibility from the outside world. Kubernetes enables external accessibility through **services**, complex constructs which encapsulate networking rules definitions on cluster nodes. By exposing services to the external world with __kube-proxy__, applications become accessible from outside the cluster over a virtual IP.

