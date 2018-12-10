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