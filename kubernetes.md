# Kubernetes Cheatsheets

### to do :

minikube, kubectl installation


## Starting Minikube

`minikube version` - to check the minikube version

`minikube start`

`minikube delete; minikube start` - if minikube instance is already running and needs to be stopped 

`minikube delete; minikube start --kubernetes-version=v1.11.3` - to start a particular kubernetes version


`kubectl version` - to check kubectl version

`kubectl cluster-info` - details of the cluster and its health status

`kubectl get nodes` - to view the available nodes in the cluster

#### Deploying Containers

`kubectl run <container-name> --image=<image-name>`

`kubectl run <container-name> --image=<user-name>/<docker-image-name>` - run from online hub

`kubectl get pods` - to view the generated pods






