---
layout: default
title: C11. Deploying a Stand-Alone Application
published: true
nav_order: 10
parent: Introduction To Kubernetes- edX
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__


- [Introduction](#introduction)
- [Deploying an Application Using the Dashboard I](#deploying-an-application-using-the-dashboard-i)
- [Deploying an Application Using the Dashboard II](#deploying-an-application-using-the-dashboard-ii)
- [Deploying an Application Using the Dashboard III](#deploying-an-application-using-the-dashboard-iii)
  - [List the Deployments](#list-the-deployments)
  - [List the ReplicaSets](#list-the-replicasets)
  - [List the Pods](#list-the-pods)
- [Exploring Labels and Selectors I](#exploring-labels-and-selectors-i)
  - [Look at a Pod's Details](#look-at-a-pods-details)
  - [List the Pods, along with their attached Labels](#list-the-pods-along-with-their-attached-labels)
  - [Select the Pods with a given Label](#select-the-pods-with-a-given-label)
  - [Try using k8s-app=webserver1 as the Selector](#try-using-k8s-appwebserver1-as-the-selector)
  - [Delete the Deployment we created earlier](#delete-the-deployment-we-created-earlier)
- [Deploying an Application Using the CLI II](#deploying-an-application-using-the-cli-ii)
  - [Create a YAML configuration file with Deployment details](#create-a-yaml-configuration-file-with-deployment-details)
- [Exposing an Application I](#exposing-an-application-i)
  - [Create a webserver-svc.yaml file with the following content:](#create-a-webserver-svcyaml-file-with-the-following-content)
  - [Using kubectl, create the Service:](#using-kubectl-create-the-service)
  - [Expose a Deployment with the kubectl expose command:](#expose-a-deployment-with-the-kubectl-expose-command)
  - [List the Services:](#list-the-services)
- [Accessing an Application](#accessing-an-application)
- [Liveness and Readiness Probes](#liveness-and-readiness-probes)
  - [Liveness](#liveness)
  - [Liveness Command (Demo)](#liveness-command-demo)
  - [Liveness HTTP Request](#liveness-http-request)
  - [TCP Liveness Probe](#tcp-liveness-probe)
  - [Readiness Probes](#readiness-probes)
  - [Deploying a Containerized Application](#deploying-a-containerized-application)

# Introduction

In this chapter, we will learn how to deploy an application using the **Dashboard (Kubernetes WebUI)** and the **Command Line Interface (CLI)**. We will also expose the application with a NodePort type Service, and access it from the external world.

# Deploying an Application Using the Dashboard I

In the next few sections, we will learn how to deploy an nginx webserver using the `nginx:alpine` Docker image.

**Start Minikube and verify that it is running**

Run this command first:

```bash 
$ minikube start
```

Allow several minutes for Minikube to start, then verify Minikube status:

```bash 
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

**Start the Minikube Dashboard**

To access the Kubernetes Web IU, we need to run the following command:

```bash 
$ minikube dashboard
```

Running this command will open up a browser with the Kubernetes Web UI, which we can use to manage containerized applications. By default, the dashboard is connected to the default Namespace. So, all the operations that we will do in this chapter will be performed inside the **default** Namespace.

![Deploying an Application - Accessing the Dashboard](../../../assets/images/intro-kubernetes/Chapter_10_-_1_-_Deploying_an_Application_-_Accessing_the_Dashboard.png)


**Note**:  In case the browser is not opening another tab and does not display the Dashboard as expected, verify the output in your terminal as it may display a link for the Dashboard (together with some Error messages). Copy and paste that link in a new tab of your browser. Depending on your terminal's features you may be able to just click or right-click the link to open directly in the browser. The link may look similar to:

```bash 
http://127.0.0.1:37751/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```

Chances are that the only difference is the PORT number, which above is 37751. Your port number may be different.

After a logout/login or a reboot of your workstation the normal behavior should be expected (where the minikube dashboard command directly opens a new tab in your browser displaying the Dashboard).

# Deploying an Application Using the Dashboard II

**Deploy a webserver using the nginx:alpine image**


From the dashboard, click on the _+CREATE_ tab at the top right corner of the Dashboard. That will open the create interface as seen below:

![Deploy a Containerized Application Web Interface](../../../assets/images/intro-kubernetes/10_-_2_-_Deploy_a_Containerized_Application_Web_Interface.png)

From that, we can create an application using a valid YAML/JSON configuration data of file, or manually from the CREATE AN APP section. Click on the CREATE AN APP tab and provide the following application details:

- The application name is **webserver**
- The Docker image to use is `nginx:alpine`, where alpine is the image tag
- The replica count, or the number of Pods, is 3
- No Service, as we will be creating it later.


![Deploy a Containerized Application Web Interface](../../../assets/images/intro-kubernetes/Deploy_a_Containerized_Application_Web_Interface.png)

If we click on Show Advanced Options, we can specify options such as Labels, Namespace, Environment Variables, etc. By default, the app Label is set to the application name. In our example **k8s-app:webserver** Label is set to all objects created by this Deployment: Pods and Services (when exposed).

By clicking on the _Deploy_ button, we trigger the deployment. As expected, the Deployment **webserver** will create a ReplicaSet (**webserver-74d8bd488f**), which will eventually create three Pods (**webserver-74d8bd488f-xxxxx**).

![Deployment Details](../../../assets/images/intro-kubernetes/10_-_3__-_Deployment_Details.png)

NOTE: Add the full URL in the Container Image field **docker.io/library/nginx:alpine** if any issues are encountered with the simple **nginx:alpine** image name (or use the **k8s.gcr.io/nginx:alpine** URL if it works instead).

# Deploying an Application Using the Dashboard III

Once we created the **webserver** Deployment, we can use the resource navigation panel from the left side of the Dashboard to display details of Deployments, ReplicaSets, and Pods in the **default** Namespace. The resources displayed by the Dashboard match one-to-one resources displayed from the CLI via **kubectl**.

## List the Deployments

We can list all the Deployments in the `default` Namespace using the `kubectl get deployments` command:

```bash 
$ kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
webserver   3/3     3            3           9m
```

## List the ReplicaSets

We can list all the ReplicaSets in the `default` Namespace using the `kubectl get replicasets` command:

```bash 
$ kubectl get replicasets
NAME                   DESIRED   CURRENT   READY   AGE
webserver-74d8bd488f   3         3         3       9m
```

## List the Pods

We can list all the Pods in the `default` namespace using the `kubectl get pods` command:

```bash 
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
webserver-74d8bd488f-dwbzz    1/1     Running   0          9m
webserver-74d8bd488f-npkzv    1/1     Running   0          9m
webserver-74d8bd488f-wvmpq    1/1     Running   0          9m 
```

# Exploring Labels and Selectors I

Earlier, we have seen that labels and selectors play an important role in grouping a subset of objects on which we can perform operations. Next, we will take a closer look at them.

## Look at a Pod's Details

We can look at an object's details using `kubectl describe` command. In the following example, you can see a Pod's description:

```bash 
$ kubectl describe pod webserver-74d8bd488f-dwbzz
Name:           webserver-74d8bd488f-dwbzz
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 15 May 2019 13:17:33 -0500
Labels:         k8s-app=webserver
                pod-template-hash=74d8bd488f
Annotations:    <none>
Status:         Running
IP:             172.17.0.5
Controlled By:  ReplicaSet/webserver-74d8bd488f
Containers:
  webserver:
    Container ID:   docker://96302d70903fe3b45d5ff3745a706d67d77411c5378f1f293a4bd721896d6420
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:8d5341da24ccbdd195a82f2b57968ef5f95bc27b3c3691ace0c7d0acf5612edd
    Port:           <none>
    State:          Running
      Started:      Wed, 15 May 2019 13:17:33 -0500
    Ready:          True
    Restart Count:  0
...

```

The `kubectl describe` command displays many more details of a Pod. For now, however, we will focus on the `Labels` field, where we have a Label set to `k8s-app=webserver`.


## List the Pods, along with their attached Labels

With the `-L` option to the `kubectl get pods` command, we add extra columns in the output to list Pods with their attached Label keys and their values. In the following example, we are listing Pods with the Label keys `k8s-app` and `label2`:

```bash 
$ kubectl get pods -L k8s-app,label2
NAME                         READY   STATUS    RESTARTS   AGE   K8S-APP     LABEL2
webserver-74d8bd488f-dwbzz   1/1     Running   0          14m   webserver   
webserver-74d8bd488f-npkzv   1/1     Running   0          14m   webserver   
webserver-74d8bd488f-wvmpq   1/1     Running   0          14m   webserver 
```

All of the Pods are listed, as each Pod has the Label key `k8s-app` with value set to `webserver`. We can see that in the `K8S-APP` column. As none of the Pods have the `label2` Label key, no values are listed under the `LABEL2` column.

## Select the Pods with a given Label

To use a selector with the `kubectl get pods` command, we can use the `-l `option. In the following example, we are selecting all the Pods that have the `k8s-app` Label key set to value `webserver`:

```bash 
$ kubectl get pods -l k8s-app=webserver
NAME                         READY     STATUS    RESTARTS   AGE
webserver-74d8bd488f-dwbzz   1/1       Running   0          17m
webserver-74d8bd488f-npkzv   1/1       Running   0          17m
webserver-74d8bd488f-wvmpq   1/1       Running   0          17m
```

In the example above, we listed all the Pods we created, as all of them have the `k8s-app` Label key set to value `webserver`.

## Try using k8s-app=webserver1 as the Selector

```bash 
$ kubectl get pods -l k8s-app=webserver1
No resources found.
```
As expected, no Pods are listed.

## Delete the Deployment we created earlier

We can delete any object using the `kubectl delete` command. Next, we are deleting the `webserver` Deployment we created earlier with the Dashboard:

```bash 
$ kubectl delete deployments webserver
deployment.extensions "webserver" deleted
```
Deleting a Deployment also deletes the ReplicaSet and the Pods it created:

```bash 
$ kubectl get replicasets
No resources found.

$ kubectl get pods
No resources found.
```

# Deploying an Application Using the CLI II

## Create a YAML configuration file with Deployment details

Let's create the webserver.yaml file with the following content:

```bash 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
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
        image: nginx:alpine
        ports:
        - containerPort: 80
```

Using `kubectl`, we will create the Deployment from the YAML configuration file. Using the `-f`option with the `kubectl create` command, we can pass a YAML file as an object's specification, or a URL to a configuration file from the web. In the following example, we are creating a webserver Deployment:

```bash 
$ kubectl create -f webserver.yaml
deployment.apps/webserver created
```

This will also create a ReplicaSet and Pods, as defined in the YAML configuration file.

```bash 
$  kubectl get replicasets
NAME                  DESIRED   CURRENT   READY     AGE
webserver-b477df957   3         3         3         45s

$ kubectl get pods
NAME                        READY     STATUS    RESTARTS   AGE
webserver-b477df957-7lnw6   1/1       Running   0          2m
webserver-b477df957-j69q2   1/1       Running   0          2m
webserver-b477df957-xvdkf   1/1       Running   0          2m
```

# Exposing an Application I

We explored different ServiceTypes. With ServiceTypes we can define the access method for a Service. For a `NodePort` ServiceType, Kubernetes opens up a static port on all the worker nodes. If we connect to that port from any node, we are proxied to the ClusterIP of the Service. Next, let's use the `NodePort` _ServiceType_ while creating a Service.

## Create a webserver-svc.yaml file with the following content:


```bash 
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx 
```

## Using kubectl, create the Service:

```bash 
$ kubectl create -f webserver-svc.yaml
service/web-service created
```
A more direct method of creating a Service is by exposing the previously created Deployment (this method requires an existing Deployment).

## Expose a Deployment with the kubectl expose command:

```bash 
$ kubectl expose deployment webserver --name=web-service --type=NodePort
service/web-service exposed
```


## List the Services:

```bash 
$ kubectl get services
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP        1d
web-service   NodePort    10.110.47.84   <none>        80:31074/TCP   22s
```

Our `web-service` is now created and its ClusterIP is `10.110.47.84`. In the PORT(S)section, we see a mapping of `80:31074`, which means that we have reserved a static port 31074 on the node. If we connect to the node on that port, our requests will be proxied to the ClusterIP on port `80`.

It is not necessary to create the Deployment first, and the Service after. They can be created in any order. A Service will find and connect Pods based on the Selector.

To get more details about the Service, we can use the `kubectl describe `command, as in the following example:

```bash 
$ kubectl describe service web-service
Name:                     web-service
Namespace:                default
Labels:                   run=web-service
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP:                       10.110.47.84
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31074/TCP
Endpoints:                172.17.0.4:80,172.17.0.5:80,172.17.0.6:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

`web-service` uses `app=nginx` as a Selector to logically group our three Pods, which are listed as endpoints. When a request reaches our Service, it will be served by one of the Pods listed in the `Endpoints` section.

# Accessing an Application

Our application is running on the Minikube VM node. To access the application from our workstation, let's first get the IP address of the Minikube VM:

```bash 
$ minikube ip
192.168.99.100
```
Now, open the browser and access the application on 192.168.99.100 at port 31074.


![Accessing the Application In the Browser](../../../assets/images/intro-kubernetes/10_-_4_-_Accessing_the_Application_In_the_Browser.png)

 We could also run the following `minikube` command which displays the application in our browser:

```bash 
$ minikube service web-service
Opening kubernetes service default/web-service in default browser...
```

We can see the _Nginx_ welcome page, displayed by the `webserver` application running inside the Pods created. Our requests could be served by either one of the three endpoints logically grouped by the Service since the Service acts as a Load Balancer in front of its endpoints.

# Liveness and Readiness Probes

While containerized applications are scheduled to run in pods on nodes across our cluster, at times the applications may become unresponsive or may be delayed during startup. 
 Implementing **Liveness** and **Readiness Probes** allows the **kubelet** to control the health of the application running inside a Pod's container and force a container restart of an unresponsive application.
 When defining both **Readiness** and **Liveness Probes**, it is recommended to allow enough time for the Readiness Probe to possibly fail a few times before a pass, and only then check the Liveness Probe. If **Readiness** and **Liveness Probes** overlap there may be a risk that the container never reaches ready state. 

 ## Liveness

 If a container in the Pod is running, but the application running inside this container is not responding to our requests, then that container is of no use to us. This kind of situation can occur, for example, due to application deadlock or memory pressure. In such a case, it is recommended to restart the container to make the application available.

 Rather than restarting it manually, we can use a **Liveness Probe**. Liveness probe checks on an application's health, and if the health check fails, **kubelet** restarts the affected container automatically.

 Liveness Probes can be set by defining:

- Liveness command
- Liveness HTTP request
- TCP Liveness Probe.

We will discuss these three approaches in the next few sections.

In the following example, we are checking the existence of a file `/tmp/healthy`:

```bash 
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

The existence of the `/tmp/healthy` file is configured to be checked every 5 seconds using the `periodSeconds` parameter. The `initialDelaySeconds` parameter requests the kubelet to wait for 5 seconds before the first probe. When running the command line argument to the container, we will first create the `/tmp/healthy` file, and then we will remove it after 30 seconds. The deletion of the file would trigger a health failure, and our Pod would get restarted.


## Liveness Command (Demo)

<iframe width="736" height="450" src="https://www.youtube.com/embed/l_gx8fMhpkU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



## Liveness HTTP Request

In the following example, the kubelet sends the `HTTP GET` request to the `/healthz` endpoint of the application, on port 8080. If that returns a failure, then the kubelet will restart the affected container; otherwise, it would consider the application to be alive.

```bash 
livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

## TCP Liveness Probe

With TCP Liveness Probe, the kubelet attempts to open the TCP Socket to the container which is running the application. If it succeeds, the application is considered healthy, otherwise the kubelet would mark it as unhealthy and restart the affected container.

```bash 
livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

## Readiness Probes



Sometimes, applications have to meet certain conditions before they can serve traffic. These conditions include ensuring that the depending service is ready, or acknowledging that a large dataset needs to be loaded, etc. In such cases, we use **Readiness Probes** and wait for a certain condition to occur. Only then, the application can serve traffic.

A Pod with containers that do not report ready status will not receive traffic from Kubernetes Services.

```bash 
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

Readiness Probes are configured similarly to Liveness Probes. Their configuration also remains the same.

Please review the [Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) for more details.

## Deploying a Containerized Application

<iframe width="736" height="450"  src="https://www.youtube.com/embed/QVi5jOqcfcE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

