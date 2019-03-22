# Create a Cluster

- [Create a Cluster](#create-a-cluster)
  - [Personal notes](#personal-notes)
  - [Kubernetes Clusters](#kubernetes-clusters)
    - [Master](#master)
    - [Node](#node)
  - [Development](#development)
  - [Intro's Cheat sheet](#intros-cheat-sheet)

## Personal notes

Those notes are a brief (and not consice) summary of the [official K8s' tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Kubernetes Clusters

K8s coordinates a highly available cluster of computers that are connected to work as a single unit.

It will automate the distribution and scheduling of application containers across a cluster in a more efficient way.

The cluster will have two type of resources

* A Master
* A node

### Master

The master is responsible for managing the cluster. It will coordinates all activities. 

### Node

The node is a VM/physical compouter that serves as a worker machine in a k8s cluster.

Every node has a **Kubelet**, an agent to manage the node and communicating with the master.

A cluster thant handles prod traffic should have a minimum of 3 nodes

## Development

A k8s cluster can be deployed on either physical or vms, but to get started with K8s, we can use Minikube.

## Intro's Cheat sheet

```bash
minikube version      # returns the version of minikube
minikube start        # Start a K8s cluster locally
kubectl version       # Verify the kubectl's version
kubectl cluser-info   # View the cluster details
kubectl get nodes     # View the nodes that can host our app in the cluster
```