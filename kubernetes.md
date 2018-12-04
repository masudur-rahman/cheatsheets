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

`kubectl run <container-name> --image=<image-name>` - to deploy an image

`kubectl run <container-name> --image=<user-name>/<docker-image-name>` - run from online hub

`kubectl get deployments` - to view the deployments

`kubectl get pods` - to view the generated pods

`kubectl get pods -o wide` - to view the generated pods with extra information


`export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')`<br>
`echo Name of the Pod: $POD_NAME` - to know the pod-name in which the image has been deployed

`kubectl expose --port=<port-number> <container-name> --type=<newworking-options>` - <port-number> : `that-has-been-exposed-in-Dockerfile-of-the provided-image`
```
Networking Options :
    - NodePort : Bistarito pore janano hobe
    - LoadBalancer : Bistarito pore janano hobe
```
`export PORT=$(kubectl get svc <container-name> -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')`<br>
`echo "Accessing host01:$PORT` - to know the host port that has been exposed outside the cluster.

`export NODE_PORT=$(kubectl get services/<container-name> -o go-template='{{(index .spec.ports 0).nodePort}}')`<br>
`echo NODE_PORT=$NODE_PORT` - to know the node port of the service

`curl $(minikube ip):$NODE_PORT` 

`kubectl scale deployments/<container-name> --replicas=<expected-number-of-replicas>` - to scale the deployment to expected number of replicas

`minikube service <container-name>` - to ready the exposed deployment for service

`kubectl describe deployments/<container-name>` -to get detailed description about the deployment

  
#### Update the version of the App

`kubectl set image deployments/<existing-container-name> <existing-container-name>=<username>/<docker-image-name>` - along with tag

`kubectl describe services/<container-name>` - to see the updated image version

`kubectl rollout status deployments/<container-name>` - to confirm the rollout status

`kubectl describe pods` - to view the current image version

###### If the image is not available in the repository, we won't get the desired number of pods.
###### We can undo the unexpected update

`kubectl rollout undo deployments/<container-name>` - to rollback to the previous version

`kubectl describe pods` - now we see that the deployment is still using the stable previous version
 
 
 #### Executing Command on the container 

`kubectl exec <pod-name> env` - to list all the environment variable

`kubectl exec -it <pod-name> sh` - to enter the shell of the mentioned pod



