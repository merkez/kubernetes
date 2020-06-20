---
layout: default
title: C10. Services 
published: true
nav_order: 9
parent: Introduction To Kubernetes- edX
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

# Introduction 



Although the microservices driven architecture aims to decouple the components of an application, microservices still need agents to logically tie or group them together and to load balance traffic to the ones that are part of such a logical set.

In this chapter, we will learn about **Services**, used to group Pods to provide common access points from the external world to the containerized applications. We will learn about the **kube-proxy** daemon, which runs on each worker node to provide access to services. We will also discuss **service discovery** and **service types**, which decide the access scope of a service.




# Connection Users to Pods 

To access the application, a user/client needs to connect to the Pods. As Pods are ephemeral in nature, resources like IP addresses allocated to it cannot be static. Pods could be terminated abruptly or be rescheduled based on existing requirements.

Let's take, for example, a scenario in which a user/client is connected to a Pod using its IP address.

![A Scenario Where a User Is Connected to a Pod via Its IP Address](../../../assets/images/intro-kubernetes/service-1.png)


Unexpectedly, the Pod to which the user/client is connected is terminated, and a new Pod is created by the controller. The new Pod will have a new IP address, which will not be known automatically to the user/client of the earlier Pod.


![A New Pod Is Created After the Old One Terminated Unexpectedly](../../../assets/images/intro-kubernetes/service-2.png)

To overcome this situation, Kubernetes provides a higher-level abstraction called Service, which logically groups Pods and defines a policy to access them. This grouping is achieved via **Labels** and **Selectors**. 

# Services 

In the following graphical representation, `app` is the Label key, `frontend` and `db` are Label values for different Pods.




![Grouping of Pods using Labels and Selectors](../../../assets/images/intro-kubernetes/service1.png)


Using the selectors `app==frontend` and `app==db`, we group Pods into two logical sets: one with 3 Pods, and one with a single Pod.

We assign a name to the logical grouping, referred to as a **Service**. In our example, we create two Services, `frontend-svc`, and `db-svc`, and they have the `app==frontend `and the `app==db` Selectors, respectively.

![Grouping of Pods using the Service object](../../../assets/images/intro-kubernetes/Services2.png)


Services can expose single Pods, ReplicaSets, Deployments, DaemonSets, and StatefulSets.

## Service Object Example

The following is an example of a Service object definition:

```bash

kind: Service
apiVersion: v1
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
```

In this example, we are creating a `frontend-svc` Service by selecting all the Pods that have the Label key=`app` set to value=`frontend`. By default, each Service receives an IP address routable only inside the cluster, known as `ClusterIP`. In our example, we have `172.17.0.4` and `172.17.0.5` as ClusterIPs assigned to our `frontend-svc` and `db-svc` Services, respectively. 


![Accessing the Pods using Service Object](../../../assets/images/intro-kubernetes/SOE.png)



The user/client now connects to a Service via its `ClusterIP`, which forwards traffic to one of the Pods attached to it. A Service provides load balancing by default while selecting the Pods for traffic forwarding.

While the Service forwards traffic to Pods, we can select the `targetPort` on the Pod which receives the traffic. In our example, the `frontend-svc` Service receives requests from the user/client on `port 80` and then forwards these requests to one of the attached Pods on the `targetPort 5000`. If the `targetPort` is not defined explicitly, then traffic will be forwarded to Pods on the `port` on which the Service receives traffic.

A logical set of a Pod's IP address, along with the `targetPort` is referred to as a `Service endpoint`. In our example, the `frontend-svc` Service has 3 endpoints: `10.0.1.3:5000`, `10.0.1.4:5000`, and `10.0.1.5:5000`. Endpoints are created and managed automatically by the Service, not by the Kubernetes cluster administrator. 

# kube-proxy 

All worker nodes run a daemon called [kube-proxy](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies), which watches the API server on the master node for the addition and removal of Services and endpoints. In the example below, for each new Service, on each node, **kube-proxy** configures **iptables** rules to capture the traffic for its ClusterIP and forwards it to one of the Service's endpoints. Therefore any node can receive the external traffic and then route it internally in the cluster based on the **iptables**  rules. When the Service is removed,**kube-proxy** removes the corresponding **iptables rules** on all nodes as well.



![kube-proxy, Services, and Endpoints](../../../assets/images/intro-kubernetes/kubeproxy.png)


# Service Discovery

As Services are the primary mode of communication in Kubernetes, we need a way to discover them at runtime. Kubernetes supports two methods for discovering Services:

- **Environment Variables**

As soon as the Pod starts on any worker node, the **kubelet** daemon running on that node adds a set of environment variables in the Pod for all active Services. For example, if we have an active Service called **redis-master**, which exposes port **6379**, and its ClusterIP is **172.17.0.6**, then, on a newly created Pod, we can see the following environment variables:

```bash 
REDIS_MASTER_SERVICE_HOST=172.17.0.6
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://172.17.0.6:6379
REDIS_MASTER_PORT_6379_TCP=tcp://172.17.0.6:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=172.17.0.6s
```
With this solution, we need to be careful while ordering our Services, as the Pods will not have the environment variables set for Services which are created after the Pods are created.

- **DNS**

Kubernetes has an [add-on](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/README.md) for [DNS](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns), which creates a DNS record for each Service and its format is **my-svc.my-namespace.svc.cluster.local**. Services within the same Namespace find other Services just by their name. If we add a Service **redis-master** in **my-ns** Namespace, all Pods in the same Namespace lookup the Service just by its name, **redis-master**. Pods from other Namespaces lookup the same Service by adding the respective Namespace as a suffix, such as redis-master.my-ns. 

# ServiceType

While defining a Service, we can also choose its access scope. We can decide whether the Service:

- Is only accessible within the cluster
- Is accessible from within the cluster and the external world
- Maps to an entity which resides either inside or outside the cluster.

Access scope is decided by ServiceType, which can be configured when creating the Service.

## ServiceType: ClusterIP and NodePort



**ClusterIP** is the default _ServiceType_. A Service receives a Virtual IP address, known as its ClusterIP. This Virtual IP address is used for communicating with the Service and is accessible only within the cluster.

With the **NodePort** ServiceType, in addition to a ClusterIP, a high-port, dynamically picked from the default range **30000-32767**, is mapped to the respective Service, from all the worker nodes. For example, if the mapped NodePort is **32233** for the service frontend-svc, then, if we connect to any worker node on port 32233, the node would redirect all the traffic to the assigned ClusterIP - 172.17.0.4. If we prefer a specific high-port number instead, then we can assign that high-port number to the NodePort from the default range

![Node Port](../../../assets/images/intro-kubernetes/NodePort.png)

The **NodePort** _ServiceType_ is useful when we want to make our Services accessible from the external world. The end-user connects to any worker node on the specified high-port, which proxies the request internally to the ClusterIP of the Service, then the request is forwarded to the applications running inside the cluster. To access multiple applications from the external world, administrators can configure a reverse proxy - an ingress, and define rules that target Services within the cluster.

## ServiceType: LoadBalancer

With the **LoadBalancer** ServiceType:

- NodePort and ClusterIP are automatically created, and the external load balancer will route to them
- The Service is exposed at a static port on each worker node
- The Service is exposed externally using the underlying cloud provider's load balancer feature.

![Load Balancers](../../../assets/images/intro-kubernetes/LoadBalancer.png)

The LoadBalancer _ServiceType_ will only work if the underlying infrastructure supports the automatic creation of Load Balancers and have the respective support in Kubernetes, as is the case with the Google Cloud Platform and AWS. If no such feature is configured, the LoadBalancer IP address field is not populated, and the Service will work the same way as a NodePort type Service.

## ServiceType: ExternalIP

A Service can be mapped to an **ExternalIP** address if it can route to one or more of the worker nodes. Traffic that is ingressed into the cluster with the ExternalIP (as destination IP) on the Service port, gets routed to one of the Service endpoints. This type of service requires an external cloud provider such as Google Cloud Platform or AWS.

![External IP](../../../assets/images/intro-kubernetes/ExternalIP.png)

Please note that ExternalIPs are not managed by Kubernetes. The cluster administrator has to configure the routing which maps the ExternalIP address to one of the nodes.



## ServiceType: ExternalName

__ExternalName__ is a special ServiceType, that has no Selectors and does not define any endpoints. When accessed within the cluster, it returns a **CNAME** record of an externally configured Service.

The primary use case of this ServiceType is to make externally configured Services like **my-database.example.com** available to applications inside the cluster. If the externally defined Service resides within the same Namespace, using just the name **my-database**  would make it available to other applications and Services within that same Namespace.
