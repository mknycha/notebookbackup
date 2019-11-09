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
- Types:
 - ClusterIP - stable internal IP (default)
 - NodePort - exposes the app outside of the cluster (on top of the above)

## Deployment:
- Feature-rich abstraction for managing replica sets (replication controllers?). Allows for simple updates and rollbacks.

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
