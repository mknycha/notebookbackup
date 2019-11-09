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

## Pod:
- Atomic unit for kubernetes. Containers run inside it. All containers in a pod share the environment - shared memory, IP . etc. Although, you scale with creating PODS, not CONTAINERS.

## Service:
- Abstraction layer for the pods doing the load balancing between them. Thanks to a service we can always refer to the changing pods with a stable API. How service is associated with pods? WITH LABELS!

## Deployment:
- Feature-rich abstraction for managing pods. Allows for simple updates and rollbacks.
