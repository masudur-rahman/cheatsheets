# <span style="color:blue"> Concepts of Kubernetes</span>

### What is Kubernetes ? What does K8s stand for ?

The name ***Kubernetes*** originates from Greek, meaning helmsman or pilot and is the root of governor and cybernetic.<br>
***K8s*** is an abbreviation derived by replacing the 8 letters "ubernete" with "8"... 


### Kubernetes Components


1. **Master Components** :
    It provides the clusters control plane. It makes global decisions about the cluster. Can run on any machine in the cluster but simply uses one.
    
    - _kube-apiserver_ : 
        - Front-End for the Kubernetes control plane
        - exposes the Kubernetes API
        - Scales by deploying more instances
    
    - _etcd_ :
         - a distributed key-value store. primary datastore of Kubernetes
         - always have a backup plan for etcd's data
     
    - _kube-scheduler_ :
        - component on the master
        - watches newly created pods to select a node for them to run on
    
    - _kube-controller-manager_ :
        - Node Controller : notices and responds when nodes go down
        - Replication Controller : maintains correct number
        - Endpoints Controller : populates the Endpoints objects
        - Service Account & Token Controller : creates default accounts and API access tokens for new namespaces
        
    - _cloud-controller-manager_ : It runs controllers that interact with the underlying cloud providers.
        - Node Controller : checks the cloud to determine if a node has been detected after stopping responding
        - Route Controller : sets up routes in the cloud 
        - Service Controller : creates, updates and deletes cloud provider load balanceers
        - Volume Controller : creates, attaches and mounts volumes to cloud provider
        
        
2. **Node Components** :
    Node components run on every node and maintain running pods in Kubernetes
    
    - _kubelet_ : 
        - an agent on each node in the cluster
        - makes sure that containers are running in a pod created by Kubernetes
    - _kube-proxy_ : 
        - maintains a network
    - _Container Runtime_ : 
        - is a software
        - responsible for running containers
        - supports several runtimes: 
            - docker
            - rkt
            - runc
            - any OCI runtime-spec implementation
3. **Addons** : 
    addons are pods and services that implement cluster features. These pods may be managed by Deloyments, ReplicationControllers and so on.<br>
    Namespaced addon objects are created in the kube-system namespace.<br>
    Some add-ons :
    
    - _DNS_ : 
        - Kubernetes clusters should have cluster DNS (DNS server).
        - containers started by kubernetes automatically include this DNS server into their DNS searches.
        
    - _Web UI (Dashboard)_ :
        - allows users to manage and troubleshoot applications running in the cluster and the cluster
        
    - _Container Resource Monitoring_ : 
        - records generic time-series metrics about containers in the central database
        - provides a UI for browsing that data
    - _Cluster-level Logging_ : 
        - saves container logs
    
    

