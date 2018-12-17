# <span style="color:blue"> Concepts of Kubernetes</span>

## What is Kubernetes ? What does K8s stand for ?

The name ***Kubernetes*** originates from Greek, meaning helmsman or pilot and is the root of governor and cybernetic.<br>
***K8s*** is an abbreviation derived by replacing the 8 letters "ubernete" with "8"... 
<br><br>


-------------------------
## Kubernetes Components

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
    
    <br><br><br>

















    
----------------------------------------
## The Kubernetes API

1. **Api Changes** :
    
2. **OpenAPI and Swagger definitions** :

3. ....
4. ....
5. ....
6. ....


<br><br><br>
    
    
    
    
---------------------------------
## Working with Kubernetes Objects

#### Understanding Kubernetes Objects
    
- _Kubernetes Objects_ describe :
    - what containerized apps are running (and on which nodes)
    - the resources available to those apps
    - the policies around how those apps behave i.e.: restart, upgrades and fault-tolerance
    

- _Object Spec and Status_ : 
    Kubernetes object includes two object field: Object _Spec_ and Object _Status_.
    - Object Spec : the desired state for the object - the characteristics that you want the object to have.
    - Object Status : the actual state of the object. <br>
    Kubernetes Control Plane actively manages an object's actual state to match the desired state.
- _Describing a Kubernetes Object_ : 
    - Spec must be provided at creation. desired state includes basic info like name, number of replicas.
     
    Save this file as `nginx-here.yaml`.
    ```yaml
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2 # tells deployment to run 2 pods matching the template
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80
    ```    
    `$ kubectl create -f nginx-here.yaml  --record`
    
    
------------------------------------

#### Names

All objects in Kubernetes REST API are uniquely identified by a Name and UID.

- **Name** :
    - A client-provided string to refer an object in a resource URL i.e.: `/api/v1/pods/some-name`.
    - Unique in a given kind
    - max length 253 chars, lower alphanumeric
    
- **UIDs** :
    - Kubernetes system generated string to uniquely identify objects
    - unique over the whole lifetime of kubernetes
    

    
--------------------------------------

#### Namespaces

Virtual clusters backed by same physical cluster are known as _namespaces_.

- **Working with Namespaces** :
    - Viewing namespaces : <br>
        `$ kubectl get namespaces` <br>
        Initial Namespaces : 
        1. default
        2. kube-system
        3. kube-public 
    - Setting the namespace for a request : 
        ```
        $ kubectl --namespace=<insert-namespace-name-here> run <container-name> --image=<username>/<image-name-with-version>
        $ kubectl --namespace=<insert-namespace-name-here> get pods
        ```
        
        
        
        
-------------------------------------

#### Labels and Selectors

- Labels are key/value pair attached to objects (e.g: pods) as identifying attributes.

    ```
    "metadata" : {
      "labels" : {
        "key1" : "value1",
        "key2" : "value2" 
      }
    }
    ```
Some valid label examples :
 
- `"release" : "stable", "release" : "canary"`
- `"environment" : "dev", "environment" : "qa", "environment" : "production"`
- `"tier" : "frontend", "tier" : "backend", "tier" : "cache"`,
- `"partition" : "customerA", "partition" : "customerB"`
- `"track" : "daily", "track" : "weekly"` 

Syntax and character sets : 

Valid label keys have two segments: an optional prefix and name, separated by a slash (/)

max length of key or value is 63 and supported character set [a-zA-Z0-9_.-]

###### Label Selectors:

- Equality-based requirement : 
    - supported operators are : `=`, `==`, `!=`. Operators `=` and `==` are similar - equility.
    - examples :
    ```
    environment = production
    tier != frontend
    ```
    ```
    environment=production,tier!=frontend
    ```
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: cuda-test
    spec:
      containers:
        - name: cuda-test
          image: "k8s.gcr.io/cuda-vector-add:v0.1"
          resources:
            limits:
              nvidia.com/gpu: 1
      nodeSelector:
        accelerator: nvidia-tesla-p100
    ```
    
- Set-based requirement : 
    - supported operators : `in`, `notin`, `exists`.
    - examples : 
    ```
    environment in (production, qa)
    tier notin (frontend, backend)
    partition
    !partition
    ``` 
    ```
    partition,environment notin (qa)
    ```
- Mixed : 
    ```
    partition in (customerA, customerB),environment!=qa
    ```
 
 ###### API
 
 LIST and WATCH filtering
 
 - equality-based   : `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
 - set-based        : `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`
 using `kubectl` :
    ```
    $ kubectl get pods -l environment=production,tier=frontend
    $ kubectl get pods -l 'environment in (production, qa),tier in (frontend)'
    $ kubectl get pods -l 'environment,environment notin (frontend)'
    ```
    
    
    
###### Service and ReplicationController

- Service and replicationController support only equality-based requirement selectors.
    
    ```json
    {
      "selector": {
          "component" : "redis"
      }
    }
    
    ```
    ```yaml
    selector:
        component: redis
    
    ```

###### Resources that support set-based requirements

- Job, Deployment, Replica Set and Daemon Set
    
    ```
    selector:
      matchLabels:
        component: redis
      matchExpressions:
        - {key: tier, operator: In, values: [cache]}
        - {key: environment, operator: NotIn, values: [dev]}
    ```
All of the requirements, from both matchLabels and matchExpressions are ANDed together – they must all be satisfied in order to match.




#### Annotations
- Annotations are somewhat similar to labels but accepts anything as key or value.

- Examples
    ```json
    {
        "metadata": {
          "annotations": {
            "key1" : "value1",
            "key2" : "value2"
          }
        }
    }
    ``` 



<br><br><br>

------------------------------



#### Field Selector

Field selectors allows to select Kubernetes resources based on the value of one or more resource fields.

- Examples : 
  
    ```
    metadata.name=my-service
    metadata.namespace!=default
    status.phase=Pending
    ```
`kubectl` command : `$ kubectl get pods --field-selector status.phase==Running`

- The following commands are same :
 
    ```
    $ kubectl get pods
    $ kubectl get pods --field-selector ""
    ```

###### Supported operators / fields: 
- `=`, `==`, `!=` are the supported operators.
- supported field selectors vary by kubernetes resource types. 

 **Multiple resource types and chained selectors can be used**  
 - Example :
  
    `$ kubectl get statefulsets,services --field-selector metadata.namespace!=default,spec.restartPolicy=Always`






--------------------------
<h2 id = "object-management"> Kubernetes Object Mangement </h2>

- **Management Techniques** : 
    - _Imperative Commands_ : Simplest way to get started to get started or to run a one-off task in a cluster.
        - Examples : 
            ```
            $ kubectl run nginx --image nginx
            ```
            Same thing  using a different syntax : 
            ```
            $ kubectl create deployment nginx --image nginx
            ```
        - Trade-offs :  
            
            Advantages compared to object configuration:
            
            - Commands are simple, easy to learn and easy to remember.
            - Commands require only a single step to make changes to the cluster. 
            
            Disadvantages compared to object configuration:
            
            - Commands do not integrate with change review processes.
            - Commands do not provide an audit trail associated with changes.
            - Commands do not provide a source of records except for what is live.
            - Commands do not provide a template for creating new objects.

    - _Imperative Object Configuration_ : In imperative object configuration, the `kubectl` command specifies the operation (create, replace, etc.), and at least one YAML or JSON file name.
        
        - Examples :  
            Create the objects defined in a configuration file :
            ```
            $ kubectl create -f nginx.yaml
            ```
            Delete the objecft defined in two configuration file :
            ```
            $ kubectl delete -f nginx.yaml -f redis.yaml
            ```
            Update the objects defined in a configuration file by overwriting the live configuration : 
            ```
            $ kubectl replace -f nginx.yaml
            ```
        - Trade-offs : 
         
            Advantages compared to imperative commands :
           
            - Object configuration can be stored in a source control system such as Git.
            - Object configuration can integrate with processes such as reviewing changes before push and audit trails.
            - Object configuration provides a template for creating new objects.
            
            Disadvantages compared to imperative commands:
            
            - Object configuration requires basic understanding of the object schema.
            - Object configuration requires the additional step of writing a YAML file.
            
            Advantages compared to declarative object configuration:            
            
            - Imperative object configuration behavior is simpler and easier to understand.
            - As of Kubernetes version 1.5, imperative object configuration is more mature.
            
            Disadvantages compared to declarative object configuration:
            
            - Imperative object configuration works best on files, not directories.
            - Updates to live objects must be reflected in configuration files, or they will be lost during the next replacement.   
    - _Declarative Object Configuration_ : 
        Create, eUpdate and Delete operations are automatically detected per-object by `kubectl`. It can work on directories. possible to use `patch` instead of `replace` operation. 
        
        - Examples : 
        
            Process all object configuration files in the configs directory and `create` or `patch` the live objects. `diff` to see the changes and then `apply` :  
            ```
            $ kubectl diff -f configs/
            $ kubectl apply -f configs/
            ```
            Recursively process directories:
            ```
            kubectl diff -R -f configs/
            kubectl apply -R -f configs/
            ```
        
        - Trade-offs : 
        
            Advantages compared to imperative object configuration:
            
            - Changes made directly to live objects are retained, even if they are not merged back into the configuration files.
            - Declarative object configuration has better support for operating on directories and automatically detecting operation types (create, patch, delete) per-object.
            
            Disadvantages compared to imperative object configuration:
            
            - Declarative object configuration is harder to debug and understand results when they are unexpected.
            - Partial updates using diffs create complex merge and patch operations.
         
        
        
<br><br>        
        

-----------------------------
<h2 id = "manage-object"> Managing Kubernetes Objects Using Imperative Commands </h2>

#### How to create objects : 

- `run`: Create a new Deployment object to run Containers in one or more Pods.
- `expose`: Create a new Service object to load balance traffic across Pods.
- `autoscale`: Create a new Autoscaler object to automatically horizontally scale a controller, such as a Deployment.

- `create <object-type> [<sub-type>] <instance-name>` - almost similar to `run` and `expose` command.
- i.e.: `$ kubectl create service nodeport <myservicename>`

#### How to update objects : 

- `scale`: Horizontally scale a controller to add or remove Pods by updating the replica count of the controller.
- `annotate`: Add or remove an annotation from an object.
- `label`: Add or remove a label from an object.
<br><br>
- `set` : Set an aspect of an object.
<br><br>
- `edit`: Directly edit the raw configuration of a live object by opening its configuration in an editor.
- `patch`: Directly modify specific fields of a live object by using a patch string. For more details on patch strings

#### How to delete objects : 

- `delete <type>/<name>`
- `kubectl delete deployment/nginx`


#### How to view an object

- `get`: Prints basic information about matching objects. Use `get -h` to see a list of options.
- `describe`: Prints aggregated detailed information about matching objects.
- `logs`: Prints the stdout and stderr for a container running in a Pod.


#### Using set commands to modify objects before creation

```
kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run | kubectl set selector --local -f - 'environment=qa' -o yaml | kubectl create -f -
```

- The `kubectl create service -o yaml --dry-run` command creates the configuration for the Service, but prints it to stdout as YAML instead of sending it to the Kubernetes API server.
- The `kubectl set selector --local -f - -o yaml` command reads the configuration from stdin, and writes the updated configuration to stdout as YAML.
- The `kubectl create -f -` command creates the object using the configuration provided via stdin.
 
 
#### Using --edit to modify objects before creation

 
- `kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run > /tmp/srv.yaml` 
- `kubectl create --edit -f /tmp/srv.yaml`
- The `kubectl create service` command creates the configuration for the Service and saves it to /tmp/srv.yaml.
- The `kubectl create --edit` command opens the configuration file for editing before it creates the object.








---------------------------------
<h2 id = "object-using-config-file"> Imperative Management of Kubernetes Objects Using Configuration Files </h2>


#### How to create objects : 

- `kubectl create -f <filename|url>`

#### How to update objects : 

- `kubectl replace -f <filename|url>`

#### How to view an  object : 

- `kubectl get -f <filename|url> -o yaml`

#### Limitations : 

- You create an object from a configuration file.
- Another source updates the object by changing some field.
- You replace the object from the configuration file. Changes made by the other source in step 2 are lost.

If you need to support multiple writers to the same object, you can use kubectl apply to manage the object.

#### Creating and editing an object from a URL without saving the configuration

`kubectl create -f <url> --edit`


#### Migrating from imperative commands to imperative object configuration


Migrating from imperative commands to imperative object configuration involves several manual steps.

- Export the live object to a local object configuration file:

    `kubectl get <kind>/<name> -o yaml --export > <kind>_<name>.yaml`
- Manually remove the status field from the object configuration file.

- For subsequent object management, use replace exclusively.

    `kubectl replace -f <kind>_<name>.yaml`


#### Defining controller selectors and PodTemplate labels

The recommended approach is to define a single, immutable PodTemplate label used only by the controller selector with no other semantic meaning.

Example label:

```yaml
selector:
  matchLabels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
template:
  metadata:
    labels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
      
```
<br><br><br><br>

--------------------------------------------------------

## Kubernetes Architecture

### Nodes

 A node is a worker machine in Kubernetes (also minion). A node may be a VM or physical machine.  
 
#### Node Status 
 
A node's status contains the following information :

##### Addresses
Usage of these fields varies depending on the cloud provider or bare metal configuration.
 - HostName : hostName is reported by node's kernel. can be overridden via the kubelet `--hostname-override` paramenter
 
 - ExternalIP : the IP of the node that is externally routable.
 - InternalIP : the IP of the node that is internally routable.
 
#### Conditions

The `conditions` field describes the status of all `running` nodes.

| Node Condition | Description|
|----------------|------------|
| `OutOfDisk` | `True` if insufficient free space for new pods, otherwise `false`|
| `Ready` | `True` if the node is healthy and ready to accept pods, `False` if not healthy and not accepting pods and `Unknown` if the node controller didn't hear from the node in the last `node-monitor-grace-period`|
| `MemoryPressure` | `True` if pressure exists on node memory|
| `PIDPressure` | `True` if pressure on processes|
| `DiskPressure` | `True` if p    ressure on the disk size|
| `NetworkUnavailable` | `True` if the network for the node is not correctly configured, otherwise `False`|

Node condition is represented as a JSON Object. e.g.: 

```json
{
    "conditions":[
        {
          "type": "Ready",
          "status": "True"
        }
    ]
}
```

If the stutus of the `Ready` condition remains `Unknown` or `False` for longer than th `pod-eviction-timeout`, an argument is passed to the kube-controller-manager and all the Pods on the nodes are scheduled for deletion by the Node Controller. The defalut eviction timeout duration is **five minutes**.


#### Capacity

Describes the resources available on the node: CPU, memory and the maximum number of pods that can be scheduled onto the code.

#### Info

General information about the node, such as kernel version, Kubernetes version (kubelet and kube-proxy version), Docker version (if used), OS name. The information is gathered by Kubelet from the node.



### Management 


Unlike pods and services, a node is not inherently created by Kubernetes: it is created externally by cloud providers like Google Compute Engine, or it exists in the pool of physical or virtual machines.

```json
{
  "kind" : "Node",
  "apiVersion" : "v1",
  "metadata" : {
    "name" : "10.240.79.157",
    "labels" : {
      "name" : "my-first-k8s-node"
    }
  }
}
```

using the following content a node can be created. Kubernetes validates the node by health checking based on the `metadata.name` field. If the pod is not valid it is ignored for any cluster activity until it becomes valid.


#### Node Controller

The node controller is a Kubernetes master component which manages various aspects of nodes.

Node controller has multiple roles in a node's life.

- The **first one** is assigning a CIDR block to the node when it is registered.
- The **second one** is keeping the node controller's internal list of nodes up to date with the cloud provider's list of available machines. If the VM for a unhealthy node is not available, the node controller deletes the node from its list of nodes.
- The **third one** is monitoring the nodes' health. - The node controller is responsible for updating the NodeReady condition of NodeStatus to ConditionUnknown when a node becomes unreachable.



#### Self-Registration of Nodes

for self-registration, the kubelet is started with the following options :

 * `--kubeconfig` - Path to credentials to authenticate itself to the apiserver.
 * `--cloud-provider` - how to talk to a cloud provider to read metadata about itself
 * `--register-node` - automatically register with the api server (default true)\
 * `register-with-taints` - register the node with the given list of taints (`<key>=<value>:<effect>`). No-op if `register-node` is false
 * `--node-ip` - IP address of the node
 * `--node-labels` - labels to add when registering the node in the cluster.
 * `--node-status-update-frequency` - specifies how often kubelet posts node status to master.
 
 When the Node authorization and NodeRestriction admission plugin are enabled, kubelets are only authorized to create/modify their own Node resource.
 
 
 
<br><br><br>

-------------------------------------
 


### Master-Node Communication


#### Cluster to Master : 

All communication paths from the cluster to the master terminate at the apiserver.

In a typical deployment, the apiserver is configured to listen for remote connections on a secure HTTPs port (443) with one or more clients.

The master components also communicate with the cluster apiserver over the secure port.


#### Master to Cluster

Two primary communication paths from the master (apiserver) to the cluster are available.

* The first one is from the apiserver to the kubelet process which runs on each node in the cluster.
* The second one is from the apiserver to any node, pod or service through the apiserver's proxy functionality.

##### apiserver to kubelet

The connections from the apiserver to the kubelet are used for:

* Fetching logs for pods.
* Attaching (through kubectl) to running pods.
* Providing the kubelet’s port-forwarding functionality.

Kubelet authentication and/or authorization should be enabled to secure the kubelet API


##### apiserver to nodes, pods and services

default plain HTTP connections. no guarantee of integrity. these connections are **not currently safe** to run over untrusted and/or public networks.  


<br><br><br>


--------------------------------------


### Concepts underlying the Cloud Controller Manager
..........
<br><br><br>


--------------------------------------



## Containers

#### Images


// Faltu jinis


<br><br><br><Br>


---------------------------


#### Container Environment Variables

##### Container environment: 

The kubernetes container environment5 provides several important resources to Containers :

* A filesystem, which is a combination of an image and one or more volumes
* Information about the container itself 
* Information about other objects in the cluster



<br><br><br><Br>


---------------------------


#### Runtime Class


<br><br><br><Br>


---------------------------



#### Container Lifecycle Hooks

##### Container Hooks

There are two hooks that are exposed to Containers :

* PostStart : This hook executes immediately after a container is created.
* PreStop : This hook is called immediately before before a container is terminated.

Users should make their hook handlers as lightweight as possible.

If the `PostStart` hook takes too long to run or hangs, the Container cannot reach a `running` state.

Similarly, if the `PreStop` hook hangs during execution, the Pod phase stays in a `Terminating` state and is killed after `terminationGracePeriodSeconds` of pod ends.

If a `PostStart` or `PreStop` hook fails, it kills the Container.



<br><br><br><Br>


---------------------------



## Workloads
 
 
### Pods

#### Pod Overview 

##### Understanding Pods

A **Pod** is the basic building block of Kubernetes - the smallest and the simplest unit in the Kubernetes object model that anyone create or deploy. A Pod represents a running process on the cluster.

A Pod encapsulates an application container (or, in some cases multiple containers), storage resources, a unique network IP and options that govern how the container(s) should run. 

Docker is the most common container runtime used in a Kubernetes Pod.

Pods in a Kubernetes cluster can be used in two main ways :

* Pods that run a single container : The "one-container-per-Pod" model is the most common Kubernetes use case. It's like a wrapper around a single container.

* Pods that run multiple containers that need to work together.  

###### How Pods manage multiple Containers

Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service. The containers are automatically co-located and scheduled in the same node. They can share resources and dependencies, communicate with one another.


![Pod Diagram](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)


###### Pod Diagram

Pods provide two kinds of shared resources for their constituent containers : networking and storage.
 
**Networking** :

Each pod is assigned a unique IP address and network port. outside the pod, the containers can communicate using ports and inside the cluster `localhost` is used.

**Storage** :

A pod can specify a set of shared storage volumes. All containers in the Pod can access the shared volumes, allowing those containers to share data. 




##### Working with Pods

Pods are rarely created by users - even singleton Pods. Pods do not, by themselves, self-heal. If a Pod is scheduled to a Node that fails, or if the scheduling operation itself fails, the Pod is deleted. 

It's far more common in Kubernetes to manage Pods using a Controller.

##### Pods and Controllers

A controller can create and manage multiple Pods, handling replication and rollout and providing self-healing capabilities. If a Node fails, the Controller might automatically replace the Pod by scheduling an identical replacement on a different Node.


##### Pod Templates

Pod templates are pod specifications which are included  in other objects, such as Replication Controllers, Jobs and DaemonSets. 

Controller use Pod Templates to make actual pods.

A sample Pod Template :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```


#### Pods

A Pod is a group of one or more containers with shared storage/network.

In terms of Docker constructs, a pod is modelled as a group of Docker containers with shared namespaces and shared volumes

Pods serve as unit of deployment, horizontal scaling, and replication.

##### Motivation for Pods


###### Resource sharing and communication

Pods enable data sharing and communication among their constituents. The applications in a pod all user same namespace (same IP and port space) and communicate using localhost. Because of this, applications in a pod must coordinate their usage of ports.

Each pod has an IP address in a flat shared networking space that has full communication with other physical computers and pods across the network.

The hostname is set to the pod’s Name for the application containers within the pod.


###### Uses of Pods

// ..............



###### Alternatives considered

*Why not just run multiple programs in a single (Docker) container ?*

* Transparency : Making everything visible to users.
* Decoupling software dependencies : to facilitate individual versioning of each container.
* Ease of use : Users don’t need to run their own process managers, worry about signal and exit-code propagation, etc. (**Bujhi nai**)
* Efficiency : containers can be more lighter


   

###### Durability of Pods (or lack thereof)

In general, users shouldn;t need to create pods directly. They should almost always use controllers for singletons, for example `deployments`.

Pod is exposed as a primitive in order to facilitate : 

* scheduler and controller pluggability
* support for pod-level operations without the need ot "proxy" them via controller APIs
* decoupling of pod lifetime from controller lifetime.
* decoupling of controllers and services - the endpoints just watch pods


###### Termination of Pods

 * User sends command to delete Pod, with default grace period of 30s
 * The Pod in the API sever is considered dead along with the grace period
 * Pod shows up Terminating when listed in the client commands
 * (simultaneous with 3) When kubelet sees the pod as terminating, it begins the pod shutdown process
    * If the pod has defined a **preStop** hook and the **preStop** hook is still running after the grace period expires, grace period is 2s extended
    * The processes in the Pod are sent the TERM signal
 * (simultaneous with 3) Pod is removed from endpoints list for service, and are no longer considered part of the set of running pods for replication controllers.
 * When the grace period expires, any processes still running in the Pod are killed with SIGKILL
 * The Kubelet will finish deleting the Pod on the API server by setting grace period 0 (immediate deletion). The Pod disappears from the API and is no longer visible from the client.

 
 -- default graceful period is 30s. The `kubectl delete` command supports `--grace-period=<seconds>` for providing custom values.
 
 additional flag `--force` is needed along with `grace-period=0` in order to perform force deletions.
 
  

###### Privileged mode for Pod Containers

From Kubernetes v1.1, any container in a pod can enable privileged mode, using th `privileged` flag on the `SecurityContext` of the container spec. This is useful for containers that want to use linux capabilities like manipulating the network stack and accessing devices.


### Pod Lifecycle

#### Pod Phase

A Pod's `status` field is a PodStatus object, which has a `phase` field.

Some possible values for phase :

Value | Description
------|------------
`Pending` | The Pod has been accepted but some Container images hasn't been created yet
`Running` | The pod has been bound to a node and at least one container is still running
`Succeeded` | All containers in the Pod have terminated in success and will not be restarted
`Failed` | All containers terminated but at least one of them terminated in failure
`Unknown` | The state of the Pod could not be obtained typically due to error in communication



#### Pod Conditions

A Pod has a PodStatus, which has an array of Pod Conditions through which the Pod has or has not passed. Each element of the PodCondition array has six possible fields :

* The `lastProbeTime` field provides a timestamp for when the Pod condition was last probed.

* The `lastTransitionTime` field provides a timestamp for when the Pod last transitioned from one status to another.

* The `message` field is a human-readable message indicating details about the transition.

* The `reason` field is a unique, one-word, CamelCase reason for the condition’s last transition.

* The `status` field is a string, with possible values “True”, “False”, and “Unknown”.

* The `type` field is a string with the following possible values:

    * `PodScheduled` : the Pod has been scheduled to a node;
    * `Ready` : the Pod is able to serve requests and should be added to the load balancing pools of all matching Services;
    * `Initialized` : all init containers have started successfully;
    * `Unschedulable` : the scheduler cannot schedule the Pod right now, for example due to lacking of resources or other constraints;
    * `ContainersReady` : all containers in the Pod are ready.
  
 
##### Container Probes

A **Probe** is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a **Handler** implemented by the Container. There are three types of handlers :

* ExecAction : Executes a specified command inside the Container. The diagnostic is considered successful if the command exits with a status code of 0.
* TCPSocketAction : Performs a TCP check against the Container’s IP address on a specified port. The diagnostic is considered successful if the port is open.
* HTTPGetAction : Performs an HTTP Get request against the Container’s IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.

<br><br>

Each probe has one of three results:

* Success: The Container passed the diagnostic.
* Failure: The Container failed the diagnostic.
* Unknown: The diagnostic failed, so no action should be taken.


The kubelet can optionally perform and react to two kinds of probes on running Containers : 

* `livenessProbe` : Indicates whether the container is running. If the liveness probe fails, the kubelet kills the Container and the Container is subjected to its restart policy. Default state is `success`.
* `readinessProbe` : Indicates whether the Container is ready to service requests. If the readiness probe fails, the endpoints controller removes the Pod’s IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is `Failure`. If a Container does not provide a readiness probe, the default state is `Success`.
                                                                                   

#### Pod and Container status

##### Pod readiness gate

// **Bujhi nai thik**

##### Restart policy

A PodSpec has a `restartPolicy` field with possible values `Always`, `OnFailure`, and `Never`. The default value is `Always`. 

`restartPolicy` applies to all Containers in the Pod,


##### Pod lifetime

Pods do not disappear until a human or a controller destroys them.

Three types of controllers are available :

* Use a Job for Pods that are expected to terminate, for example, batch computations. Jobs are appropriate only for Pods with restartPolicy equal to OnFailure or Never.
* Use a ReplicationController, ReplicaSet, or Deployment for Pods that are not expected to terminate, for example, web servers. ReplicationControllers are appropriate only for Pods with a restartPolicy of Always.
* Use a DaemonSet for Pods that need to run one per machine, because they provide a machine-specific system service. 

**Example states** :

* Pod is running and has one Container. Container exits with success.

	* Log completion event.
	* If restartPolicy is:
		* Always: Restart Container; Pod phase stays Running.
		* OnFailure: Pod phase becomes Succeeded.
		* Never: Pod phase becomes Succeeded.
* Pod is running and has one Container. Container exits with failure.

	* Log failure event.
	* If restartPolicy is:
		* Always: Restart Container; Pod phase stays Running.
		* OnFailure: Restart Container; Pod phase stays Running.
		* Never: Pod phase becomes Failed.
* Pod is running and has two Containers. Container 1 exits with failure.

	* Log failure event.
	* If restartPolicy is:
		* Always: Restart Container; Pod phase stays Running.
		* OnFailure: Restart Container; Pod phase stays Running.
		* Never: Do not restart Container; Pod phase stays Running.
	* If Container 1 is not running, and Container 2 exits:
		* Log failure event.
		* If restartPolicy is:
			* Always: Restart Container; Pod phase stays Running.
			* OnFailure: Restart Container; Pod phase stays Running.
			* Never: Pod phase becomes Failed.
* Pod is running and has one Container. Container runs out of memory.

	* Container terminates in failure.
	* Log OOM event.
	* If restartPolicy is:
		* Always: Restart Container; Pod phase stays Running.
		* OnFailure: Restart Container; Pod phase stays Running.
		* Never: Log failure event; Pod phase becomes Failed.
* Pod is running, and a disk dies.

	* Kill all Containers.
	* Log appropriate event.
	* Pod phase becomes Failed.
	* If running under a controller, Pod is recreated elsewhere.
* Pod is running, and its node is segmented out.

	* Node controller waits for timeout.
	* Node controller sets Pod phase to Failed.
	* If running under a controller, Pod is recreated elsewhere.




<br><br><br>

----------------------------

#### Init Containers

##### Understanding Init Containers

A Pod can have multiple Containers running apps within it, but it can also have one or more Init Containers, which are run before the app Containers are started.

Init Containers are exactly like regular Containers, except:

* They always run to completion.
* Each one must complete successfully before the next one is started.

If an Init Container fails for a Pod, Kubernetes restarts the Pod repeatedly until the Init Container succeeds. However, if the Pod has a `restartPolicy` of Never, it is not restarted.


##### What can Init Containers be used for?

Because Init Containers have separate images from app Containers, they have some advantages for start-up related code:

* They can contain and run utilities that are not desirable to include in the app Container image for security reasons.
* They can contain utilities or custom code for setup that is not present in an app image. For example, there is no need to make an image FROM another image just to use a tool like sed, awk, python, or dig during setup.
* The application image builder and deployer roles can work independently without the need to jointly build a single app image.
* They use Linux namespaces so that they have different filesystem views from app Containers. Consequently, they can be given access to Secrets that app Containers are not able to access.
* They run to completion before any app Containers start, whereas app Containers run in parallel, so Init Containers provide an easy way to block or delay the startup of app Containers until some set of preconditions are met.

Examples :

`myapp.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    labels:
        app: myapp
spec:
    containers:
      - name: myapp-container
        image: busybox
        command:
            - sh
            - -c
            - while true; do echo app is running; sleep 1; done
        imagePullPolicy: IfNotPresent
    initContainers:
      - name: init-myservice
        image: ubuntu
        command:
            - sh
            - -c
            - "apt-get update; apt-get install dnsutils -y; until nslookup myservice; do echo waiting for myservice; sleep 2; done"
      - name: init-mydb
        image: ubuntu
        command:
            - sh
            - -c
            - "apt-get update; apt-get install dnsutils -y; until nslookup myservice; do echo waiting for myservice; sleep 2; done"
    restartPolicy: Always
```



`services.yaml` :
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376

---

apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
    ports:
      - port: 80
        protocol: TCP
        targetPort: 9377
``` 

**Kubectl Commands** :

`kubectl create -f myapp.yaml`

`kubectl get -f myapp.yaml`

Following will be shown:  
```
➤ kubectl get -f go/src/yaml_files/myapp.yaml -w
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          4s
myapp-pod   0/1   Init:0/2   0     7s
```


To inspect the first init container: 
`kubectl logs myapp-pod -c init-myservice`

To inspect the second init container :

`kubectl logs myapp-pod -c init-mydb`

 


`kubectl create -f services.yaml`

`kubectl get -f myapp.yaml`

Now the pod will be ready: 
```
myapp-pod   0/1   Init:0/2   0     25s
myapp-pod   0/1   Init:1/2   0     82s
myapp-pod   0/1   Init:1/2   0     88s
myapp-pod   0/1   PodInitializing   0     114s
myapp-pod   1/1   Running   0     115s
```


##### Pod restart reasons :

A Pod can restart, causing re-execution of Init Containers, for the following reasons:

* A user updates the PodSpec causing the Init Container image to change. App Container image changes only restart the app Container.
* The Pod infrastructure container is restarted. This is uncommon and would have to be done by someone with root access to nodes.
* All containers in a Pod are terminated while `restartPolicy` is set to Always, forcing a restart, and the Init Container completion record has been lost due to garbage collection.



#### Pod Preset

// couldn't run : `no matches for kind "PodPreset" in version "settings.k8s.io/v1alpha1"`

<br><br><br>

----------------------
----------------------

### Controllers

#### ReplicaSet

##### How to use a ReplicaSet

Most `kubectl` commands that support Replication Controllers also support ReplicaSets. 

For `rolling-update` functionality using Deployments is **highly recommended**.
Also, the `rolling-update` command is imperative whereas Deployments are declarative, `rollout` command is recommended.


##### When to use a ReplicaSet

From Deployment, creating Replicas is recommended.

An example using ReplicaSet is provided below : 
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
```


Another example using `Deployment` : 

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: rss-site
spec:
  replicas: 2
  template:
      metadata:
          labels:
              app: web
      spec:
          containers:
              - name: front-end
                image: nginx
                ports:
                    - containerPort: 80
              - name: rss-reader
                image: nickchase/rss-php-nginx:v1
                ports:
                    - containerPort: 88
```


<br><br><br>

-----------------

#### ReplicationController

A ReplicationController ensures that a specified number of pod replicas are running at any time. 
In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.


##### Running an example ReplicationController

```yaml
apiVersion: v1
kind: ReplicationController

metadata:
    name: nginx
spec:
    replicas: 3
    selector:
        app: nginx
    template:
        metadata:
            name: nginx
            labels:
                app: nginx
        spec:
            containers:
                - name: nginx
                  image: nginx
                  ports:
                    - containerPort: 80
```


##### Multiple release tracks

to create 10 replicated pods among which 9 will be stable and 1 will be canary,
needed to set up a ReplicationController with `replicas=9` and labels `tier=frontend, environment=prod, track=stable` and another ReplicationController with `replicas=1` and labels `tier=frontend, environment=prod, track=canary`.




