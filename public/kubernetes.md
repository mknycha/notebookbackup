 - Pod - basic unit for all of the workloads you run on Kubernetes. Group of containers sharing some stuff (storage, IP, etc.). Deployment creates Pods.
 - Node - Pod always run on a Node. Pod is tied to the Node where it is scheduled, and remains there until termination (according to restart policy) or deletion. Can be VM or physical machine.
 - Kubectl - command line interface for kubernetes
 - Kubernetes always runs some **Master Node**, this node has some **children nodes** (they work like separate VMs), on each child node there is a **pod** (think of it as a container for containers).
 - Service - can act as a middle man, allowing the consumers to address pod even when it goes down and up and it's IP has changed.
 
## Master:
- Master node consists of:
  - apiserver - exposes a frontend UI and some rest API. This rest API consumes JSON (that's how our manifest file is used). The only component we deal with directly.
  - cluster store - node memory, uses **etcd** (no relational, distributed database) under the hood. Source of truth for the cluster. Make sure there is a backup for it!
  - controller - controler manager for other controllers :) like nodes controller, namespace controller etc. They watch for changes, and make sure that current state of the cluster is the desired one.
  - scheduler - checks if there is a new pod and assigns work to nodes.
- Master node should not do any work for the user, it only looks after the cluser!

## Node:
- Kublet, container runtime, kube-proxy
- Kublet - main kubernetes agent, watches apiserver on a master for work assignment, creates pods and reports back to the master. (master "makes decisions" regarding creating pods). Exposes endpoint on port 10255 for inspeciton
- Contianer runtime - container manager. Pulls images, starts and stops containers. Docker or rkt.
- Kube-proxy - makes sure that every pod has a separate IP assigned (but containers in a pod share IP). Does some light load balancing between the pods.

## Declarative model and desired state concept
-  We declare some model representing how the desired state of our application. If the desired state is not the actual one, kubernetes takes care to fix it.

## Replication controller:
- Takes care of desired state in terms of replication. Deploys the pod and replicates it when needed.

## Pod:
- Atomic scheduling unit for kubernetes. Containers run inside it. All containers in a pod share the environment - shared memory, IP . etc. Although, you scale with creating PODS, not CONTAINERS. Pod cannot span multiple nodes.

## Service:
- Abstraction layer for the pods doing the load balancing between them. Thanks to a service we can always refer to the changing pods with a stable API. How service is associated with pods? WITH LABELS!
- Each type is built on top of other type (e.g. NodePort creates ClusterIP under the hood)
- Types:
 - ClusterIP - stable internal IP (default). Not available outside the cluster
 - NodePort - exposes the app outside of the cluster (on top of the above)
 - LoadBalancer - takes care of traffic distribution accros the nodes. When it receives a traffic, it selects one node at random. Then, it uses this node IP table to send the traffic to a random Pod, it does not need to be a pod within that node - this may cause latency (Double-hop). We can make it always select a Pod within chosen Node, by setting `externalTrafficPolicy: local`. This way preserve the source client IP address, but may cause the imbalance in the traffic distribution.
 
## Ingress:
- Kind of service for services
- It's a collection of rules that direct external inbound connections to a set of services within the cluster. Each http rule will contain a host and optionally a path. Ingress routes incoming requests based on these rules. If no host is matched, the default service is called.

## GCP Cloud-native load balancing
- Load balancing not on Node level, but on Pods level directly.

## Network policy:
- Allows to disable access from some Pods to other Pods within a cluster.
- Policy types (could be specified in one field):
 - Ingress - incoming traffic to the affected pods (**has nothing to do with Ingress resource**)
 - Egress - outgoing traffic from the pods
 - Empty policy type means that ingress is applied by default

## Deployment:
- Describe desired state of pods
- Feature-rich abstraction for managing replica sets (replication controllers?). Allows for simple updates and rollbacks.
- Some features: easily move from one version of the code to the next one, use health checks to verify if the deployed application works as it should, wait for configurable amount of time between upgrading each pod.
- Stores the revision history (its size is configurable) so you can rollback (cmd: `rollback undo`)
- There are two deployment strategies:
  - Recreate - terminates all the pods at once and recreates them (all olds pods are deleted before new pods can be created). Fast, simple and does not require pods compatibility with the previous version but (usually) causes downtime.
  - RollingUpdate - updates pods incrementally, a few at the time. Takes more time but it's more robust and does not cause downtime. Since this means that for a short while we will have some pods running old version and some running new version, **make sure that the pods you are updating are compatible with other parts of the app** (e.g. if we are updating backend API, will it be a problem when our frontend calls one time API v1 and the other time API v2?)
- You cannot have less pods running than the current number? Set `maxUnavailable` to 0%, and `maxSurge` to more than 0%. This way deployment will first create a new pod before removing the old one.
- Strategies that can be applied manually:
  - Blue/Green deployment - create all new pods, then patch the service so that it redirects all the traffic to the new pods (if everything is fine). It causes twice as many pods in total to be created for the time of deployment.
  - Canary deployment - same as above, but redirect the traffic gradually. 

## Job:
- Creates one or more pods in order to run a specific task reliably
- Unlike other Kubernetes controllers, jobs manages a task up to its completion rather than to an open-ended desired state.

## DaemonSet:
- Daemon set ensures a copy of a Pod is running across a set of nodes in a Kubernetes cluster
- When we have a stateless app decoupled from a node that we can replicate -> use ReplicaSet
- When we have an app of which single copy should run on each node or subset of nodes -> use DaemonSet
- By default, DaemonSet creates pod on every node in the cluster. We can limit nodes using node selector.
- Update -> after version 1.6 use rolling update
- _If you want to use deployments to reliably rollout your software you have to specify readiness health checks for the containers in your Pod_

## Volume:
- A method to attach storage to a Pod
- Can outlive the Pod (NFS, PersistentVolume), or die when they are no longer attached (ConfigMap and Secrets).
- Volume is stored within a Pod. It can connect to a persistent data storage, but we would need to specify it in the Pod manifest, which is not flexible.
- PersistentVolume is stored outside of the Pod, but we need a PersistentVolumeClaim within a Pod to access it. PersistentVolume allows to decouple storage administration from application configuration.
- StorageClass - set of characteristics with a name assigned
- Need high availability of persistent disk? Try setting in the StorageClass
```
replication-type: regional-pd
zone: some-regional-zone-1, some-regional-zone-2
```

## Storage integration:
- There are several ways to go here:
  - Import the external service. You can use the current database running in your infrastructure or in a cloud provider. You deifine a service referring to an external database - either by specifing type `ExternalName` in the service spec, or defining `Endpoints` for it. There is no health check option in that case.
  - Reliable singleton. Runs your database in one pod. In that case you would need a persistent volume, pod running the database (managed by ReplicaSet, so that it's restarted in case it goes down) and a service to expose it.
  - Using StatefulSets. 
  
## StatefulSet:
-  manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods
- each Pod in a StatefulSet has ordinal index (unique sequence number assigned to this pod), stable hostname and stably indentified storage.
- Uses unique volume claim for each storage. This way each Pod can have individual state.

## ConfigMap:
- decouples configuration from the pod
- can be defined with a manifest file, passing a file, specify a directory or a name for file path
- ConfigMap can be used in a Pod as a container environment variable, in a pod command, or by creating a volume
- Can be updated within a Pod when created through a volume

## Example files:
- Create a pod:
```
apiVersion: v1
kind: Pod   # This way kubernetes knows what kind of resource to deploy
metadata:
  name: hello-pod  # simply pod name
spec:
  containers:
  - name: hello-ctr
    image: nigelpoulton/pluralsight-docker-ci
    ports:
    - containerPort: 8080
```
- Create a replication controller:
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: hello-rc
spec:
  replicas: 10
  selector:
    app: hello-world  # selects all pods with hello-world selector
  template:   # use this pod template
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-pod
        image: nigelpoulton/pluralsight-docker-ci
        ports:
        - containerPort: 8080
```
Create a service:
```
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
  labels:
    app: hello-world
spec:
  type: NodePort # Service type - exposes the app outside by adding a cluster wide port
  ports:
  - port: 8080
    nodePort: 30001
    protocol: TCP
  selector:
    app: hello-world # has to match the pods in out RC
```
Deployment:
```
apiVersion: apps/v1 # we need to use new version to use deployment
kind: Deployment
metadata:
  name: hello-deploy
spec:
  replicas: 10
  minReadySeconds: 10 # wait 10 seconds before you mark the deployment as ready
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1 # one pod at a time
      maxSurge: 1 # no more than one extra pod (in this case 1 above 10)
  selector:
    matchLabels:
      app: hello-world  # selects all pods with hello-world selector
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-pod
        image: nigelpoulton/pluralsight-docker-ci
        ports:
        - containerPort: 8080
```

## Commands:
- `kubectl expose rc hello-rc --name=hello-svc --target-port=8080 --type=NodePort` - wrap replication controller in a service and expose it on port 8080 (create a service manually)
- `kubectl logs --since=3h [POD_NAME]` -> print the logs from most recent 3 hours


Resources:
- "Kubernetes Up and Running" book
- "Getting started with Kubernetes" course by Nigel Poulton on Pluralsight

## Why is helm important to Kubernetes?
https://boxboat.com/2018/09/19/helm-and-kubernetes-deployments/
