---
layout: default
title: C9. Authentication, Authorization
published: true
nav_order: 8
parent: Introduction To Kubernetes- edX
---

⚠️ __Notice: With respect to licence ([https://creativecommons.org/licenses/by-nc-nd/4.0/](https://creativecommons.org/licenses/by-nc-nd/4.0/)) on this course, I am distributing and sharing important points and some information which  are taken from [https://www.edx.org/course/introduction-to-kubernetes](https://www.edx.org/course/introduction-to-kubernetes) course__

- [Introduction](#introduction)
- [Authentication, Authorization, and Admission Control - Overview](#authentication-authorization-and-admission-control---overview)
- [Authentication I](#authentication-i)
- [Authentication II](#authentication-ii)
  - [Authorization I](#authorization-i)
  - [Authorization II](#authorization-ii)
  - [Authorization III](#authorization-iii)
- [Admission Control](#admission-control)
- [Authentication and Authorization Exercise Guide](#authentication-and-authorization-exercise-guide)
- [Authentication and Authorization (Demo)](#authentication-and-authorization-demo)

# Introduction

Every API request reaching the API server has to go through three different stages before being accepted by the server and acted upon. In this chapter, we will be looking into the Authentication, Authorization and Admission Control stages of Kubernetes API requests. 


# Authentication, Authorization, and Admission Control - Overview

To access and manage any Kubernetes resource or object in the cluster, we need to access a specific API endpoint on the API server. Each access request goes through the following three stages:

- Authentication
  Logs in a user.
- Authorization
  Authorizes the API requests added by the logged-in user.
- Admission Control
  Software modules that can modify or reject the requests based on some additional checks, like a pre-set Quota.

The following image depicts the above stages:

![Access To k8s](../../../assets/images/intro-kubernetes/access-k8s.png)


**Accessing the API**

# Authentication I

Kubernetes does not have an object called user, nor does it store usernames or other related details in its object store. However, even without that, Kubernetes can use usernames for access control and request logging, which we will explore in this chapter.

Kubernetes has two kinds of users:

- **Normal Users**

  They are managed outside of the Kubernetes cluster via independent services like User/Client Certificates, a file listing usernames/passwords, Google accounts, etc.
- **Service Accounts**

  With Service Account users, in-cluster processes communicate with the API server to perform different operations. Most of the Service Account users are created automatically via the API server, but they can also be created manually. The Service Account users are tied to a given Namespace and mount the respective credentials to communicate with the API server as Secrets.
  
If properly configured, Kubernetes can also support **anonymous requests**, along with requests from Normal Users and Service Accounts. **User impersonation** is also supported for a user to be able to act as another user, a useful feature for administrators when troubleshooting authorization policies.

# Authentication II

For authentication, Kubernetes uses different [authentication modules](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authentication-strategies):

- **Client Certificates**

    To enable client certificate authentication, we need to reference a file containing one or more certificate authorities by passing the `--client-ca-file=SOMEFILE` option to the API server. The certificate authorities mentioned in the file would validate the client certificates presented to the API server. A demonstration video covering this topic is also available at the end of this chapter.
- **Static Token File**

    We can pass a file containing pre-defined bearer tokens with the `--token-auth-file=SOMEFILE` option to the API server. Currently, these tokens would last indefinitely, and they cannot be changed without restarting the API server.
- **Bootstrap Tokens**

    This feature is currently in beta status and is mostly used for bootstrapping a new Kubernetes cluster.

- **Static Password File**

    It is similar to Static Token File. We can pass a file containing basic authentication details with the `--basic-auth-file=SOMEFILE` option. These credentials would last indefinitely, and passwords cannot be changed without restarting the API server.
- **Service Account Tokens**

    This is an automatically enabled authenticator that uses signed bearer tokens to verify the requests. These tokens get attached to Pods using the ServiceAccount Admission Controller, which allows in-cluster processes to talk to the API server.
- **OpenID Connect Tokens**

    OpenID Connect helps us connect with OAuth2 providers, such as Azure Active Directory, Salesforce, Google, etc., to offload the authentication to external services.
- **Webhook Token Authentication**

    With Webhook-based authentication, verification of bearer tokens can be offloaded to a remote service.

- **Authenticating Proxy**

    If we want to program additional authentication logic, we can use an authenticating proxy. 

## Authorization I

After a successful authentication, users can send the API requests to perform different operations. Then, those API requests get authorized by Kubernetes using various authorization modules.

Some of the API request attributes that are reviewed by Kubernetes include user, group, extra, Resource or Namespace, to name a few. Next, these attributes are evaluated against policies. If the evaluation is successful, then the request will be allowed, otherwise it will get denied. Similar to the Authentication step, Authorization has multiple modules/authorizers. More than one module can be configured for one Kubernetes cluster, and each module is checked in sequence. If any authorizer approves or denies a request, then that decision is returned immediately.

## Authorization II

- **Node Authorizer**

    Node authorization is a special-purpose authorization mode which specifically authorizes API requests made by kubelets. It authorizes the kubelet's read operations for services, endpoints, nodes, etc., and writes operations for nodes, pods, events, etc. For more details, please review the [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/node/).

- **Attribute-Based Access Control (ABAC) Authorizer**

    With the ABAC authorizer, Kubernetes grants access to API requests, which combine policies with attributes. In the following example, user student can only read Pods in the Namespace `lfs158`.

    ```bash 
    {
    "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
    "kind": "Policy",
    "spec": {
        "user": "student",
        "namespace": "lfs158",
        "resource": "pods",
        "readonly": true
    }
    }
    ```

    To enable the ABAC authorizer, we would need to start the API server with the `--authorization-mode=ABAC` option. We would also need to specify the authorization policy with `--authorization-policy-file=PolicyFile.json`. For more details, please review the Kubernetes documentation.

- **Webhook Authorizer**

    With the Webhook authorizer, Kubernetes can offer authorization decisions to some third-party services, which would return true for successful authorization, and false for failure. In order to enable the Webhook authorizer, we need to start the API server with the `--authorization-webhook-config-file=SOME_FILENAME`option, where `SOME_FILENAME `is the configuration of the remote authorization service. For more details, please see the [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/webhook/).

## Authorization III

- **Role-Based Access Control (RBAC) Authorizer**

In general, with RBAC we can regulate the access to resources based on the roles of individual users. In Kubernetes, we can have different roles that can be attached to subjects like users, service accounts, etc. While creating the roles, we restrict resource access by specific operations, such as `create, get, update, patch`, etc. These operations are referred to as verbs.

In RBAC, we can create two kinds of roles:

**Role**

With Role, we can grant access to resources within a specific Namespace.

**ClusterRole**

The ClusterRole can be used to grant the same permissions as Role does, but its scope is cluster-wide.

In this course, we will focus on the first kind, Role. Below you will find an example:

```bash 
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: lfs158
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

```
As you can see, it creates a `pod-reader` role, which has access only to read the Pods of `lfs158` Namespace. Once the role is created, we can bind users with RoleBinding.

There are two kinds of __RoleBindings__:

*RoleBinding*

It allows us to bind users to the same namespace as a Role. We could also refer a ClusterRole in RoleBinding, which would grant permissions to Namespace resources defined in the ClusterRole within the RoleBinding’s Namespace.

*ClusterRoleBinding*

It allows us to grant access to resources at a cluster-level and to all Namespaces.

In this course, we will focus on the first kind, RoleBinding. Below, you will find an example:

```bash 
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-read-access
  namespace: lfs158
subjects:
- kind: User
  name: student
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

As you can see, it gives access to the student user to read the Pods of lfs158 Namespace.

To enable the RBAC authorizer, we would need to start the API server with the `--authorization-mode=RBAC` option. With the RBAC authorizer, we dynamically configure policies. For more details, please review the [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).


# Admission Control

Admission control is used to specify granular access control policies, which include allowing privileged containers, checking on resource quota, etc. We force these policies using different admission controllers, like ResourceQuota, DefaultStorageClass, AlwaysPullImages, etc. They come into effect only after API requests are authenticated and authorized.

To use admission controls, we must start the Kubernetes API server with the `--enable-admission-plugins`, which takes a comma-delimited, ordered list of controller names:

```bash
--enable-admission-plugins=NamespaceLifecycle,ResourceQuota,PodSecurityPolicy,DefaultStorageClass
```

Kubernetes has some admission controllers enabled by default. For more details, please review the [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).

# Authentication and Authorization Exercise Guide

This exercise guide assumes the following environment, which by default uses the certificate and key from `/var/lib/minikube/certs/`, and `RBAC` mode for authorization:

- Minikube v1.0.1
- Kubernetes v1.14.1
- Docker 18.06.3-ce

This exercise guide may be used together with the video demonstration following on the next page and it has been updated for the environment mentioned above, while the video presents an older Minikube distribution with Kubernetes v1.9. 

Start Minikube:

```bash 
$ minikube start
```

View the content of the `kubectl` client's configuration file, observing the only context `minikube` and the only user `minikube`, created by default:

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

Create `lfs158` namespace:

```bash 
$ kubectl create namespace lfs158

namespace/lfs158 created
```

Create `rbac` directory and `cd` into it:

```bash 
$ mkdir rbac

$ cd rbac/
```
Create a `private key` for the `student` user with `openssl` tool, then create a `certificate signing request` for the `student` user with `openssl` tool:

```bash 
~/rbac$ openssl genrsa -out student.key 2048

Generating RSA private key, 2048 bit long modulus (2 primes)
.................................................+++++
.........................+++++
e is 65537 (0x010001)

~/rbac$ openssl req -new -key student.key -out student.csr -subj "/CN=student/O=learner"
```

Create a YAML configuration file for a `certificate signing request object`, and save it with a blank value for the request field: 

```bash 
~/rbac$ vim signing-request.yaml

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: student-csr
spec:
  groups:
  - system:authenticated
  request: <assign encoded value from next cat command>
  usages:
  - digital signature
  - key encipherment
  - client auth
```

View the `certificate`, encode it in `base64`, and assign it to the `request` field in the `signing-request.yaml` file:

```bash 

~/rbac$ cat student.csr | base64 | tr -d '\n'

LS0tLS1CRUd...1QtLS0tLQo=

~/rbac$ vim signing-request.yaml

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: student-csr
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUd...1QtLS0tLQo=
  usages:
  - digital signature
  - key encipherment
  - client auth
```


Create the `certificate signing request object`, then list the certificate signing request objects. It shows a pending state:
```bash 
~/rbac$ kubectl create -f signing-request.yaml

certificatesigningrequest.certificates.k8s.io/student-csr created

~/rbac$ kubectl get csr

NAME          AGE   REQUESTOR       CONDITION
student-csr   27s   minikube-user   Pending

Approve the certificate signing request object, then list the certificate signing request objects again. It shows both approved and issued states:

~/rbac$ kubectl certificate approve student-csr

certificatesigningrequest.certificates.k8s.io/student-csr approved

~/rbac$ kubectl get csr

NAME          AGE   REQUESTOR       CONDITION
student-csr   77s   minikube-user   Approved,Issued
```

Extract the approved certificate from the certificate signing request, decode it with base64 and save it as a certificate file. Then view the certificate in the newly created certificate file:

```bash 
~/rbac$ kubectl get csr student-csr -o jsonpath='{.status.certificate}' | base64 --decode > student.crt

~/rbac$ cat student.crt

-----BEGIN CERTIFICATE-----
MIIDGzCCA...
...
...NOZRRZBVunTjK7A==
-----END CERTIFICATE-----
```

Configure the student user's credentials by assigning the key and certificate: 

```bash 
~/rbac$ kubectl config set-credentials student --client-certificate=student.crt --client-key=student.key
```

User "student" set.

Create a new context entry in the kubectl client's configuration file for the student user, associated with the lfs158 namespace in the minikube cluster:
```bash 
~/rbac$ kubectl config set-context student-context --cluster=minikube --namespace=lfs158 --user=student
```
Context "student-context" created.

View the contents of the kubectl client's configuration file again, observing the new context entry student-context, and the new user entry student:
```bash 
~/rbac$ kubectl config view

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
- context:
    cluster: minikube
    namespace: lfs158
    user: student
  name: student-context
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/student/.minikube/client.crt
    client-key: /home/student/.minikube/client.key
- name: student
  user:
    client-certificate: /home/student/rbac/student.crt
    client-key: /home/student/rbac/student.key
```

While in the default minikube context, create a new deployment in the lfs158 namespace:
```bash 
~/rbac$ kubectl -n lfs158 create deployment nginx --image=nginx:alpine

deployment.apps/nginx created
```
From the new context student-context try to list pods. The attempt fails because the student user has no permissions configured for the student-context:
```bash 
~/rbac$ kubectl --context=student-context get pods
```
Error from server (Forbidden): pods is forbidden: User "student" cannot list resource "pods" in API group "" in the namespace "lfs158"

The following steps will assign a limited set of permissions to the student user in the student-context. 

Create a YAML configuration file for a pod-reader role object, which allows only get, watch, list actions in the lfs158 namespace against pod objects. Then create the role object and list it from the default minikube context, but from the lfs158 namespace:
```bash 
~/rbac$ vim role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: lfs158
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

~/rbac$ kubectl create -f role.yaml

role.rbac.authorization.k8s.io/pod-reader created

~/rbac$ kubectl -n lfs158 get roles

NAME         AGE
pod-reader   57s
```

Create a YAML configuration file for a rolebinding object, which assigns the permissions of the pod-reader role to the student user. Then create the rolebinding object and list it from the default minikube context, but from the lfs158 namespace:
```bash 
~/rbac$ vim rolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-read-access
  namespace: lfs158
subjects:
- kind: User
  name: student
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

~/rbac$ kubectl create -f rolebinding.yaml 

rolebinding.rbac.authorization.k8s.io/pod-read-access created

~/rbac$ kubectl -n lfs158 get rolebindings

NAME              AGE
pod-read-access   23s

Now that we have assigned permissions to the student user, we can successfully list pods from the new context student-context.

~/rbac$ kubectl --context=student-context get pods

NAME                    READY   STATUS    RESTARTS   AGE
nginx-77595c695-f2xmd   1/1     Running   0          7m41s
```


# Authentication and Authorization (Demo)

<iframe  width="736" height="450" src="https://www.youtube.com/embed/pALxfDJiaCo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>