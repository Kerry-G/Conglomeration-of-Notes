# Kubernetes Architecture

- [Kubernetes Architecture](#kubernetes-architecture)
  - [What is Kubernetes?](#what-is-kubernetes)
  - [Components of Kubernetes](#components-of-kubernetes)
  - [Challenges](#challenges)
  - [The Borg Heritage](#the-borg-heritage)
  - [Kubernetes Architecture](#kubernetes-architecture-1)
    - [Kubelet](#kubelet)
    - [Kube-proxy](#kube-proxy)
  - [Terminology](#terminology)
  - [Master Node](#master-node)
    - [kube-apiserver](#kube-apiserver)
    - [kube-scheduler](#kube-scheduler)
    - [etc database](#etc-database)
    - [Other agents](#other-agents)
  - [Minions nodes](#minions-nodes)
    - [kubelet](#kubelet)
  - [Pods](#pods)
  - [Services](#services)
  - [Controllers](#controllers)
  - [Networking Setup](#networking-setup)
  - [CNI Network Config File](#cni-network-config-file)
  - [Pod-to-Pod communication](#pod-to-pod-communication)
  - [Labs](#labs)
    - [Deploy a master node](#deploy-a-master-node)
    - [Deploy a working node](#deploy-a-working-node)
    - [Add autocompletion](#add-autocompletion)
    - [Pod](#pod)
    - [YAML files dummy](#yaml-files-dummy)
    - [Create a Simple Deployment](#create-a-simple-deployment)

## What is Kubernetes?

According to kubernetes.io website, Kubernetes is: "an open-source system for automating deployment, scaling, and management of containerized applications."

It comes from 15 years of experience at Google in a project called borg.

## Components of Kubernetes

Instead of using large server, K8s approaches the issue by deploying a lot of small servers aka "microservices".

Clients expect server processes to die and replaced quickly.

Bigger application are breaked down into smaller application, or *services* and they communicate throught API calls.

Communication is API call-driven internally. The configuration information is stored in a JSON format, but often written in YAML. 

Kubernetes is written in Go

## Challenges

Containers have re-become popular in the past years. it helps to the dev experience to build/share quickly.

The challenge with containers is that it needs a solid CI pipeline to build images, test them and verify them -- then it needs a cluster of machine to act as the base infrastructure to which it runs the containers. It also needs a system to launch all the containers, watch them fail and heal. Also it should provide a way to rollback and rolling updates. 

## The Borg Heritage

Borg is the internal system used by Google to manage its applications. It is the software that inspired current data center systems. [The ideas behind Kubernetes are written down in this paper](https://ai.google/research/pubs/pub43438).

## Kubernetes Architecture

There's a master node and some worker node. The manager runs:

* An API server
* A scheduler
* Various controllers
* A storage system to keep the state of the cluster
* Container settings
* Networking configuration

K8s exposes the API via the API server. You can communicate with it through `kubectl`, or we could write our own client. The *kube-scheduler* sees the request and find a suitable node to run that container in. Each node run two process:

* kubelet
* kube-proxy

### Kubelet

* Receives request to run the containers
* Manages any necessary resources and watch them

### Kube-proxy

Create and manages networking rules to expose the container on the network

## Terminology

* Pod: One or more containers which share an IP/storage/namespace.
* Controllers: Series of watch-loops that interrogates the kube-apiserver for a particular object state, modifying the object until the declared state matches the current states.
* Deployment: Ensure that the resources are free and then deploy a ReplicaSet
* ReplicaSet: A controller that deploys/restarts containers, until the request # of containers is running.
* Jobs/CronJobs: handle single/recurring tasks
* Labels: Arbitrary strings which become part of the object metadata.
* Taints: Node attribute to discourage Pod assignements, unless the pod has a **toleration** in its metadata.
* Annotation: Remain with the object but can't be used by K8s commands. Usefull for 3rd party agents.

## Master Node

It runs various server and maanger processes for the cluster, such as: kube-apiserver, kube-scheduler, the etcd database, the cloud-controller-manager (it handles task once handled by the kube-controller-manager to interact with other tools for cluster management/reporting)

### kube-apiserver

It's central to the K8s cluster. All calls (internal/external) are handled via thsi agent. All actions are accepted and validated by this agent. Somewhat the front-end of the cluster.

### kube-scheduler

It uses an algorithm to determine which node will host a Pod of containers. We can affect the algorithm or even provide our own custom scheduler. Also we can bind a Pod to a particular node.

### etc database

The state of the cluster/networking is kept in an etcd DB, or more precisly an b+tree k-v store. 

Instead of retrieving a value and chaging its entry, values are always appended to the end. Previous copies of the data are then marked for future removal by a compaction process. 

Request to update a value travel via the kube-apiserver, which then passes along the request to etc in a series.

### Other agents

The kube-controller-manager is a core control loop daemon which interacts with the kube-apiserver.

If at some point, the state does not match, the manager will make contact the right manager to make it match.

Since 1.8, the cloud-controller-manager interacts with agents outside of the cloud. It handles task previously handled by kube-controller-manager.

It allows faster changes without altering the core K8s control process. Each kubelet must use the `--cloud-provider-external`

## Minions nodes

All workers nodes run the kubelet and the kube-proxy, as well as the container's engine (Docker or cri-o)

The kubelet intereacts with the underlying Docker Engine and make sure everything runs. The kube-proxy is in charge of managing the network connectivity to the containers. It does so with the help of iptables. ipvs can be enabled to replace it.

Supervisord is a lightweight process used in traditional Linux env. to monitor and notify about other processes. In the clsuter, this daemon monitors both the kubelet/docker processes. As in restarting them if they fail and log events. K8s does not provide cluster-wide logging yet. 

### kubelet

The kubelet agent is the heavy lifter for changes and configuration on worker nodes. Accepts API calls for Pod specifications. If a Pod requires storages (Secrets/ConfigMaps), the kubelet ensure access/creation. It also sends back status to the kube-apiserver for eventual persistence

## Pods 

Smallest unit that we can work on with K8s are Pods. Containers in a Pod are started in parallel. No way to know which one starts first. Only one IP per Pod, if more than one container, they must share the same IP. To communicate with eachother, they can either use IPC or a shared fs.

A sidecar can be used for logging or any helper tasks. This is a common reason to have multiples containers in a Pod

## Services

Each services is a microservice handling a particular bit of traffic, such as a single NodePort/LoadBalancer to distribute inbound requests among many Pods. It also handles access policies for inbound requests, useful for resource control/security. 

## Controllers

Many controllers ship with K8s, and we can create our own. A simplified view of an controller is an agent, or Informer and a downstream store. Using a DeltaFIFO queue, the source and the downstream are compared. 

A loop process receives an object, and as long as the delta is not of type Deleted, the controller is used to create/modify some objects until the specification are matched. 

The Informer requests the state of an object via the API server, The data is then cached. A similar agent is the SharedInformer; objects are often used by multiple other objects, so it creates a shared cache of the state.

The Workqueue uses a key to hand out tasks to various workers. The standard Go work queues of rate limiting, delayed, and time queue are usually used.

## Networking Setup

The three main challenges to solve in a container orchestration system are:

* Coupled container-to-container communications (solved by the Pod Concept)
* Pod-to-pod communications
* External-to-pod communications (solved by the services concept)

K8s excepts the network configuration to enable pod-to-pod communication to be available; it will not do it for you.

[An usefull guide made by one of the lead K8s dev](https://speakerdeck.com/thockin/illustrated-guide-to-kubernetes-networking?slide=6).

## CNI Network Config File

K8s is standardizing on the CNI (Container Network Interface) specification. It is language agnostic.

To have more information about this subject, [refer to here](https://github.com/containernetworking/cni).

## Pod-to-Pod communication

A CNI plugin can be used to configure the network of a pod and provide a single IP per pod, CNI does not help you with p2p communication.

The requirement from K8s is the following:

* All pods can comunicate with each other across nodes
* All nodes can communicate with all pods
* No NAT

[K8s reference](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

## Labs

### Deploy a master node 

```bash
# Disable swap until next reboot
sudo swapoff -a

# Update the local node
sudo apt-get update && sudo apt-get upgrade -y 

# Install docker
sudo apt-get install -y docker.io

# Install kubeadm and kubectl
sudo sh -c "echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' >> /etc/apt/sources.list.d/kubernetes.list"
sudo sh -c"curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -"
sudo apt-get update
sudo apt-get install -y kubeadm=1.13.1-00 kubelet=1.13.1-00 kubectl=1.13.1-00

# Starting the cluster
sudo kubeadm init --kubernetes-version 1.13.1 --pod-network-cidr 192.168.0.0/16 

# Provided by init
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Download Calico plugin and RBAC YAML files and apply
wget https://tinyurl.com/yb4xturm -O rbac-kdd.yaml
wget https://tinyurl.com/y8lvqc9g -O calico.yaml
kubectl apply -f rbac-kdd.yaml
kubectl apply -f calico.yaml

# Verify that everything is OK
kubectl get node
```

### Deploy a working node

```bash
# Disable swap until next reboot
sudo swapoff -a

# Update the local node
sudo apt-get update && sudo apt-get upgrade -y 

# Install docker
sudo apt-get install -y docker.io

# Install kubeadm and kubectl
sudo sh -c "echo 'deb http://apt.kubernetes.io/ kubernetes-xenial main' >> /etc/apt/sources.list.d/kubernetes.list"
sudo sh -c"curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -"
sudo apt-get update
sudo apt-get install -y kubeadm=1.13.1-00 kubelet=1.13.1-00 kubectl=1.13.1-00

# Join the cluster -- This command is provided by the init command on the master node
sudo kubeadm join [token] [discovery-token-ca-cert-hash]
```

### Add autocompletion

```bash
source <(kubectl completion bash) # Add autocompletion on this shell
echo "source <(kubectl completion bash)" >> ~/.bashrc # Add autocompletion on the future shell
```

### Pod

```bash
kubectl create -f [name].yaml           # Create a Pod with a YAML file
kubectl get pod                         # Details of all the pods
kubectl describe pod [nameGivenInYAML]  # More details about a specific pod
kubectl delete pod [nameGivenInYAML]    # Delete the pod
kubectl get pod -o wide                 # See the internal IP assigned and the NOMINATED MODE
kubectl get scv                         # Get the current serivces
```

### YAML files dummy

```yaml
# dummy.yaml
apiVersion: v1
kind: Pod
metadata:
  name: dummypod            # name of the pod
  labels:
    type: webserver
spec:
  containers:
  - name: webcont
    image: nginx
    ports:                  # expose port 80
    - containerPort: 80 
  - name: fdlogger          # multi-container configuration aka sidecar
    image: fluent/fluentd   # image where it's pulling the image from
# dummyservice.yaml 
apiVersion: v1
kind: Service
metadata:
  name: dummyservice        # name of the service
spec:
  selector:
    type: webserver
  type: NodePort            # expose a port to the outside world
  ports:
  - protocol: TCP
    port: 80
```

### Create a Simple Deployment

```bash
kubectl create deployment firstpod --image=nginx    # create a simple, single replica deployment running nginx (similar to just a pod, but with extra controllers)
kubectl get deployment,pod                          # see the deployment and the pod that it created
kubectl get namespaces                              # see all the available namespaces
kubectl get pod -n [namespace]                      # see all the pod in a namespaces
kubectl get pod --all-namespaces                    # get all pod in all namespaces
kubectl get deploy,rs,po,svc,ep                     # see all deployments, ReplicaSet, Pod, Service and Endpoints

# In a deployment, we can see that if we delete the ReplicaSet, it will manage to put it back up automatically in a matter of seconds. However if you delete
# the deployment, it will take around 30s and everything related to that deployment will get deleted.
```