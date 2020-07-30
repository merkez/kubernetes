# Elasticsearch Deployment to Kubernetes Cluster 

There is a same issue with Jenkins, the issue is that although services and pods are running without any error, ingress is complaining about `504 Gateway error`. 
Currently, there is no proper solution for it, however in worst case, ingress controller might be re-installed or changed with something new like Traefik, Istio or any other 
ingress controller which is available in following list: [https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)


For detailed information about rules, examples and more about ELK stack deployment to Kubernetes cluster, this repo [https://github.com/pires/kubernetes-elasticsearch-cluster](https://github.com/pires/kubernetes-elasticsearch-cluster) might help in someway, although it is archieved. 