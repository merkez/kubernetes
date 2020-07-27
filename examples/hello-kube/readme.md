# Install services 

Deploying given services is quite easy task to do, you only need to execute create command on provided yaml files. 

```bash 
$ kubectl create -f hello-service_1.yaml
$ kubectl create -f hello_service_2.yaml
```

Once those services are deployed and you have already configured your HAProxy for forwarding traffic from 80 and 443, you can access those services through provided ingress rule. 

```bash 
$ kubectl create -f ingress-rule.yaml
```

It is advised to create different namespaces for services  which are running on kubernetes cluster, since it provides more robust authentication, accessibility and flexibility in terms of managing kubernetes cluster. 

If no namespace defined in any yaml file, then all deployment will run on default namespace which is not recommended. No seperate namespace defined in this example, since it is just a simple example of how ingress rules work. 


__Do NOT hesitate to change or propose something new or create new issue__
