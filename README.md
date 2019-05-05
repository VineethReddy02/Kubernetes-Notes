# Kubernetes-Notes

#### Kubernetes ####

Kubernetes only understands pods not containers.

Pod Creation Requests 
1. Bare pod
2. RelplicaSet
3. Deployment

### Multi Container Pods ####

Share access to memory space

Connect to each other using localhost

Share access to the same volumes (storage abstraction)

TIghtly coupled.

One crashes, all crashes.

Same parameters such as config maps.

### Pod ###

Atomic Unit in Kubernetes is Pod

In a pod either all conatiners or none of them run.
It's always a pod runs on single node.
Which pod runs on which node decided be scheduler.
If a pod is failed kubelet notifies to k8s control plane.

### Higher level Kubernetes Objects ###

Replicaset, ReplicationController: Scaling and healing
Deployment: Versioning and rollback
Service: Static (non-epemeral) IP and networking
Volume: Non-ephemeral storage

### K8s nodes contains ###

1. Kubelet
2. Kube proxy
3. Conatiner Runtime.

Pod cannot be auto scaled or self healed for tis cases we need higher level objects such as 
ReplicaSet, ReplicationController, Deployment.

### Micro-Services ###

Bare Metal: Apps ere very tightly coupled.
Virtual Machines: Less tighly coupled, but not yet micro-services.
Containers for Micro-Services: Simple, Independent components.

###  Resource in Conatiners ###

1. Usually the conatianers are aloocated with deafult resources in kubernetes.
2. By providng Resource Request we are seeking resources as requested to be default resources.
3. By providing limit in resources. You are restricting conatiner to occupy till the limit mentioned 
	starting from default/Requested resources.
4. You can even mention both request and limit or either of them or neither of them.

### Communicaion between Master and Cluster ###

Kubernetes Master Conatins  following components

1. Kube-scheduler
2. Controller-Manager this is based n where k8s cluster is running. If it is not on cloud provider
	then Kube-controller manager, If it is cloud then cloud contoller manager.
	1. Cloud controller Manager
	2. Kube-controller Manager
3. etcd
4. Kube-apiserver

*** Cluster to Master ***

Only APISERVER will be exposed outside(i.e to cluster) none of the other components are 
exposed outside.

All cluster to master communication happen with only API-SERVER.

Relatively Secure

*** Master to Cluster ***

1. APISERVER to Kubelet

These are not safe in public or untusted networks

1. Certificate not verified by default.
2. Vulnerable to man in the middle attacks.
3. Dont run on public network
4. To hardern.
	1. set- -kubelet-certificate-authority
	2. Use SSH tunneling.
	
	
	
2. APISERVER to nodes/pods/Services

	- Not safe
	- Plain HTTP
	- Neither authenticated or encrypted.
	- On public clouds, SSH tunneling provided by cloud provider e.g. GCP.
	
	
	
*** Where can we run kubernetes ***

1. Public Clouds
	1.AWS
	2.Azure
	3.GCP


2. Bootstrap for running k8s cluster on private cloud or on prem. Using kubeadm we configure the 
	kubernetes.


3. Playgrounds.
	1.PWK
	2.MINIKUBE
	
*** Hybrid, Multi-Cloud ***

Hybrid = On-prem + Public Cloud

Multi-Cloud = More than 1 public cloud.

*** Federated CLusters ***

1. Nodes in multiple clusters.
2. Administer with kubefed.


**** Individual CLuster ****

1. All nodes on sae infra.
2. Administer with kubectl. 


### Kubernetes Provides ####

1. Fault-tolerance: Pod/Node faiures
2. Rollback: Advanced Deployment options
3. Auto-healing: Crashed conatiner restart
4. Auto-scaling: More clients? More demand
5. Load-balancing: Distribute client requets
6. Isolation: Sanboxes so that containers don't interfere.

#### How to interact with kubernetes #####

kubectl: Most common command line utility. Makes POST requests to apiserver of control plane.
kubeadm: Bootstrap cluster when not on cloud kubernetes service. To create cluster out of individual
				infra nodes.
kubefed: Administer federated cluters.
			  Federated cluster -> group of multiple clusters (multi-cloud,hybrid)
			  
kubelet, kube-proxy,....

All these are different cmd line utilities to interact with different components of k8s cluster.


**** Kubernetes API ****

1. APISERVER within conrol plane exposes API endpoints
2. CLients hit these endpoints with RESTful API calls.
3. These clients could be command line tools such as kubectl, kubeadm.....
4. Could also be programmatic calls using client libraries.

*** Objects ***

1. Kubernetes Objects are persistent entities.
2. Everything is an object....
3. Pod, RelplicaSet, Deployment, Node .... all are objects
4. Send object specification (usually in .yaml or json)

%%%%% IMPORTANT POINTS %%%%%%

-> Pods doen't support auto-healing or auto scaling.
-> Kube-apiserver - Accepts incming HTTP post requests from users.
->  Etcd - Stores metadata that forms the state of the cluster.
-> Kube-scheduler - Makes Decision about where and when the pods should run.
-> CLoud-controller manager - Keeps the actual and desired state of the cluster in synch.

*** THree object Management Methods ***
1. Imperative COmmands
	No .yaml or config files
	eg: kubectl run ..., kubectl expose ..., kubectl autscale ..
	For this happen the objects should be live in cluster and this is the least robust way of managing objects.
	
2. Imperative Object Configuration
	kubectl + yaml or config files used.
	eg: 
	kubectl create -f config.yaml
	kubectl replace  -f config.yaml
	kubectl delete -f config.yaml
	
3. Declarative Object Configuration
	Only .yaml or configfiles used
	eg:
	kubectl apply -f config.yaml 
	This is the most preferred way of handling objects.
	
Note: Don't mix and match different methods in handling k8s objects.

### Imperative COmmands ####

kubectl run nginx --image nginx
kubectl create deployment nginx --image nginx

No config file.
Imperative: intent is in command.

Pro:
Simple

COns:
No audit trail or review mechanism
cant reuse or use in template.


#### Imperative Object COnfiguration #####

kubectl create -f nginx.yaml
kubectl delete -f nginx.yaml
kubectl replace -f nginx.yaml

config file required
dtill imperative: intent is in cmd.

Pros:
-still simple
-Robust - files checked into repo
-One file for multiple operations

#### Declarative Object COnfiguration #### Used in production

kubectl apply -f configs/

config files are all that is required.
Declarative not imperative.

Pros;
-Most robust - review,repos,audit trails.
-k8s will automatically figure out intents
-Can specify multiple files/directories recursively.

Declarative COnfiguration has three phases.

1. Live object configuration
2. Current object configuration file.
3. Last-applied object configuration file.

Merging changes.

1. Primitive fields
		1. String, int, boolean,images or replicas
		2. Replace old state with current object configuration file.
		
2. Map fields 
		1. Merge old state with current state with current object configuration file.

3. List fields
		1.Complex- varies by field.
		

### VOLUMES AND PersistentVolumes ####

1. Volumes(in general): lifespan of abstraction = lifetime of pod.
		1. Note that this is longer than lifetime of any container inside pod.
		2. Persistent Volumes.
		
2. Persistent Volumes: lifetime of abstraction independent of pod lifetime.


Using Volumes

```apiVersion: v1
kind: Pod
metadata: 
		name: configmap-pod
spec:
   containers:
	- name: test
	  image: bisybox
    volumeMounts:         // Each container will mount independently
	- name : config-vol
	  mountPath: /etc/config					// different paths in each container.
    volumes:                                                   //Define volume in pod spec
	-name: config-vol
	configMap:
	   name: log-config
	   items:
	      - key: log_level
	        path: log_level
```
						  
						  
						  
						  
Volumes binded to pod are persistent across the lifecyclces of containers. But when pod restarts the vlumes are last.
emptyDir comes with empty volume initially and also when pod restarts it loses all the data in it.

### Important types of volumes are ###

1. configMap
2. emptyDir
3. gitRepo
4. secret
5. hostPath


emptyDir

This is not persistent. his exists as long as the pod exists. Created as empty volume.
Share space/state across conatiners in same pod.
When the pod is removed the emptyDir volume is lost.

When pod removed/crashes. data lost
When conatiner crashes data remains
Usecases: Scartch space, checkpointing

hostPath

Mount file/directory from node filesystem into pod
Uncommon - pods should be independent of nodes
Makes pod-node coupling tight
Usecases: Access docker internals, running cAdvisor
BLock devices or sockets on host

gitRepo

This volume will create an empty directory and go ahead and clone git repo to our volume so that our conatiners can use it.

configMap

Used to inject paraeters and configuration data into pods.
configMap volume mount data from configmap object
configMap objects define key-value pairs
configMap objects inject parameters into pods

Two main usecases:
1.Providing config information for apps running inside pods
2. Specifying config infrmation for control plane(controllers )

kubectl create configmap fmap --from-file=file1.txt --from-file=file2.txt
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.how: very
  
 Inside pod yaml file
 
 env:
   - name: SPECIAL_LEVEL_KEY
     valueFrom:
       configMapKeyRef:
         name: special-config
	 key: special.how
```	 

secret

Pass sensitive information to pods.
You can store secrets using kubernetes api and mount those secrets as files these files will be available to use by pods.
using the secret volume
You should know secrets are backed by RAM based file system which ensures contents of this files are never written to non volatile storage
```
	apiVersion: v1
	kind: Secret
	metadata:
	     name: test-secret
	data:
	     username: VINEETH
	     password: ###@!#
```
	     
Once the secrets are created we can access from volumes inside the pod yaml file.
```
spec:
   conatiners:
   - name: test-container
     image: nginx
     volumeMounts:
     // name must matc the voume name below
     - name: secret-volume
       mountPath: /etc/secret-volume
   // The secret data is exposed to containers in the Pod through a volume.
   volumes:
   	- name: secret-volume
	  secret:
	     secretName: test-secret
```
We can access this secret by getting into the container shell and by going to etc/secret-volume.

We can create secrets directly from files.

kubectl create secret generic sensitive --from-file=./username.txt --from-file=./password.txt
Inside pod yaml file
```
env:
   - name: SECRET_USERNAME
     valueFrom:
        secretKeyRef:
	   name: sensitive
	   key: username.txt
  - name: SECRET_PASSWORD
    valueFrom:
         secretKeyRef:
	    name: sensitive
	    key: password.txt
```


### Using PersistentVolumes ###

we mount the persistnt volumes with containers 
```
volumeMounts:
- mountPath: /test-pd
  name: test-volume
```
  
  

### Conatiners in Pod ###

1. Configure Nodes to Authenticate to Private Repos. All pods can pull any image.
2. Pre-pull images. Pods can only use cached images.
3. ImagePullSecrets on each pod. Only pods with secret can pull secrets.
 
 What Environment Do Containers See ?
 
 1. Filesystem
 	Image(at root)
	Associated Volumes
	  - ordinary
	  - persistent
	  
 2. Container
 	Hostname
Hostname refers to the pod name in which conatiner is running.
We can get by cmd hostname or gethostname function call from libc.

 3. Pod
 	Pod Name
	User-defined
	environment variables using Downward API
4. Services
	List of all services

	
	  
 
  
### Services for stable IP Addresses ###

Service object - load balancer
Service = Logical set of backend pods + stable front-end
Front-end: Static clusterIP address + Port + DNS Name
Back-end: Logical set of backend pods(label selector)


#### Setting up environment varibales #####
```
spec:
  conatiners:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO
      value: "HELLO"
    - name: DEMO1
      valueL "HEY"
 ```     
kubectl exec -it demo-pod -- /bin/bash
This will take into the bash shell within our conatiner.
`printenv` \\ will print all env variables.


### Downward API ###

Passing information from pod to conatiner such as metadata, annotations.

pods/inject/dapi-volume.yaml 
```
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
```


In the above example we are making pod metadata such as labels, annotations available for conatiners.
etc/podinfo/annotations annotations are available in this file.
etc/podinfo/labels labels are available in this file.


### Conatiner Lifecycle Hooks ###

1. Post Start

Called immediately after conatiner created
No parameters

2. Pre Stop

Immediately before conatiner terminates.

Blocking - must complete before conatiner can be deleted. This is synchronous.

 1. Hook handkers
    - Exec //This executes shell scripts by getting inside conatiner
    - HTTP // We can make calls to specific endpoint on the conatiner
```
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  conatiners:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
	  command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
        exec:
	  command: ["/usr/sbin/nginx","-s","quit"]
```	  
	  
	  
### Pod Node Matching ###

How can pods be assigned to specific nodes?

  Handled by kube-scheduler
    -Quite smart (it makes sure the nodes which has resources gets the pod assigned.)
  Granular usecases:
    -specific hardware: SSD required by pod
    -Colocate pods on same node: they communicate a lot.
    -High-availability:force pods to be an different nodes.


nodeSelector (nodes have predefined labels hostname, zone, OS, instance type...)
-Simple
-Tag nodes with labels
-Add nodeSelector to pod template
-Pods will only reside on nodes that are selected ny nodeSelector
-Simple but crude - hard constriant

Affinity and Anti-Affinity

Node Affinity (nodes have predefined labels hostname, zone, OS, instance type...)
-steer pod to node
- can be 'soft'
-Only affinity (for anti-affinity use taints)

Pod Affinity

-Steer pods towards or away from pods.
-Affinity: pods close to each other
-Anti-Affinity: pods away from each other.

```
apiVersion: v1
kind: Pod
metadata: Pod
  name: nginx
  labels:
    env: test
spec:
  conatiners:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```    
    
### Taints and Tolerations ###

Using nodeslector you can make sure this pod should run on specific node but using taints and tolerations you can make sure certain pods can only run on certain nodes.

Dedicated nodes for certain users
  - Taint subset of nodes
  - Only add tolerations to pods of those users.
Nodes with special hardware
  - Taint nodes with GPUs
  - Add toleration only pods running ML jobs

Taints based on Node Condition

  - New feature - in Alpha in v1.8
  - Taints added by node controller

Taints added by node controller
  - node.kubernetes.io/memory-pressure
  - node.kubernetes.io/disk-pressure
  - node.kubernetes.io/out-of-disk
  
 Pods with tolerations are scheduled on this nodes.
 This will happen if flag set on nodes
  - TaintNodesByCondition=true
	  



 

