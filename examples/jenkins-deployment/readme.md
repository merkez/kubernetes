
## Installation of Jenkins 

Jenkins installation could be made in different forms and variations depends on your cluster setup. In this guide, I am trying to install jenkins deployment in a cluster where cluster has HAProxy in front and NGINX ingress controller within cluster. 

Default values and specifications are given in helm charts repo for Jenkins installation, therefore, in this guide easy approach of installing Jenkins on top of cluster will be described. 

### Download and Change some values

```bash 
$ wget https://raw.githubusercontent.com/helm/charts/master/stable/jenkins/values.yaml
```

#### Change default values 

You need to change following values from downloaded yaml file from helm charts. 

```yaml
runAsUser: 0 # required to bypass permission denied error when persistent volume is created. 

jenkinsOpts: "--prefix=/jenkins"
jenkinsUrl: "jenkins.example.com"


storageClass: jenkins-pv # name for persistent volume which will be created manually.

```

## Create YAML files for Ingress, PV, Namespace

* __Namespace__

```yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
```
* __PV__

```yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  namespace: jenkins
spec:
  storageClassName: jenkins-pv
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 20Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/jenkins-volume/
```

* __Ingress__

Note that ingress did not work work very well, it will be updated when it is fixed. Therefore, you may not need this ingress controller for accessing dashboard, since it throws `503 Service Unavailable Error`

```yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins-ingress
  namespace: jenkins
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
    nginx.ingress.kubernetes.io/cors-allow-headers: Authorization, origin, accept
    nginx.ingress.kubernetes.io/cors-allow-methods: GET, OPTIONS
    nginx.ingress.kubernetes.io/enable-cors: "true"

spec:
  rules:
  - host: jenkins.example.com
    http:
      paths:
      - path: /jenkins
        backend:
          serviceName: jenkins
          servicePort: 8080
```

## Create all through yaml files

```bash 
$ kubectl create -f namespace.yaml
$ kubectl create -f pv.yaml
```

### Install Jenkins with help of HELM 


```bash 
$ helm install stable/jenkins --name-template=jenkins --namespace=jenkins -f values.yaml
```
In given command, namespace and values file defined in order run Jenkins with customized values. Once it is deployed to cluster, following information will be printed to console. You can access and manage your Jenkins through given printed information .


```raw
NAME: jenkins
LAST DEPLOYED: Fri Jul 24 14:16:11 2020
NAMESPACE: jenkins
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace jenkins -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=jenkins" -o jsonpath="{.items[0].metadata.name}")
  echo http://127.0.0.1:8080
  kubectl --namespace jenkins port-forward $POD_NAME 8080:8080

3. Login with the password from step 1 and the username: admin

4. Use Jenkins Configuration as Code by specifying configScripts in your values.yaml file, see documentation: http:///configuration-as-code and examples: https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine
For more information about Jenkins Configuration as Code, visit:
https://jenkins.io/projects/jcasc/
```