# Build

- [Build](#build)
  - [Container Options](#container-options)
    - [Docker](#docker)
    - [Container Runtime Interface (CRI)](#container-runtime-interface-cri)
    - [Rkt](#rkt)
    - [CRI-O](#cri-o)
    - [Containerd](#containerd)
  - [Containerizing an Application](#containerizing-an-application)
  - [Creating the Dockerfile](#creating-the-dockerfile)
  - [Hosting a Local Repository](#hosting-a-local-repository)
  - [Creating a Deployment](#creating-a-deployment)
  - [Running Commands in a Container](#running-commands-in-a-container)
  - [Multi-Container Pod](#multi-container-pod)
  - [readinessProbe](#readinessprobe)
  - [livenessProbe](#livenessprobe)
  - [Labs](#labs)
    - [Deploy a New App](#deploy-a-new-app)
    - [Configure a Local Docker Repository](#configure-a-local-docker-repository)
    - [Configure Probes](#configure-probes)
      - [Exemple of YAML section](#exemple-of-yaml-section)

## Container Options 

The container runtime is the component that runs the containerized application (e.g. Docker). DE is the default for K8s, however cri-o and rkt are gaining community support.

Why not Docker? People are moving to a environment that are not bound to higher-level tools and more portable across OS and env. The Open Container Initiative (OCI) was formed for that. Docker used to be the only choice, but not anymore.

### Docker

The default one. Most ready for production right now, might not be the case in the foresable future. They develop docker-swarm for orchestration but that increase vendor-locking.

### Container Runtime Interface (CRI)

The goal of the CRI is to allow easy integration of different container runtime with kubelet. 

Still in alpha stage, with a lot of development in action. Docker-CRI is done; cri-o,rktlet and frakti are WIP.

### Rkt

Pronounced rocket. Part of the CNCF. Focused on being mroe secure,open and interoperable. Not a drop-in replacement for Docker. rkt use appc specs, and run Docker, appc and OCI images. 

It was expected to be the replacment for Docker until CRI-o became part of the official K8s Incubator.

### CRI-O

Uses the K8s CRI with OCI-compatible runtimes, (CRI-O...). Currently, there is support for RunC and Clear Containers, but the goal of the projet is to work with any OCI-compliant runtime. Newer than Docker/rkt. 

### Containerd

Not intented to build a user-facing tool; more about exposing a highly-decoupled low-level primitives. Defaults to RunC to run containers, intended to be embedded into larger systems with a minimal CLI, focused on debugging/dev. Geared towards specialized products.

## Containerizing an Application

The more stateless/transient the application is, the better it will perform in a container. 

Should not have any environmental configuration, those will be provided using other tools, liek ConfigMaps. Should be deployable on any machine without the need of being changed. Many legacy projects becomes a series of objects/artifacts.

## Creating the Dockerfile

Create a directory that holds all the application files, and create a *Dockerfile*. The name is important. It is expected by `docker build`. As this is not a notebook about Docker, more details about Dockerfile over [here](https://docs.docker.com/engine/reference/builder/).

## Hosting a Local Repository

It's easy to publish on the Docker Hub, however that means your images are then public and accessible to the world. We can create a local repo and push images there. Consider setting up an insecure repository until you know it works, then configuring TLS for security.

## Creating a Deployment

Once we can push/pull the images using docker command, we should run a new deployment inside K8s using the same image.

```bash
kubectl run [Deploy-name] --image=[repo]/[app-name]:[version]
```

## Running Commands in a Container

To run commands inside a container that runs in K8s, we can use the `-it` options for an interactive shell. 

```bash
kubectl exec -it [podName] --/bin/bash
```

## Multi-Container Pod

Instead of making an entire image to add functionality like a shell/logging agent. Instead you could add another container to the pod. 

## readinessProbe

Most of the time, an app needs to be init/config before accepting traffic. Instead of trying to reach the node before it's ready, we can set up a *readinessProbe*. There is three example of such probes

* With the *exec* command, the container is not considered ready until a command returns a zero exit code.
* With HTTP GET requests and the container is not considered healthy before it returns a code between 200-399. 
* With TCP Socket probe, it will attempt to open a socket on a port, once the port can be opened, the container is considered healthy.

## livenessProbe

If container is not checking if it's healthy, we can implement a *livenessProbe*. Similar to the readinessprobe, but if it's fail the test, the container get terminated. 

## Labs

### Deploy a New App

This lab mostly create a simple python scripts, make a Dockerfile to run it and then deploy it with Docker, nothing related to K8s

### Configure a Local Docker Repository

I won't go over all the steps but only write down usefull commands

```bash
kompose convert -f docker-compose.yaml -o localregistry.yaml # Convert docker file into YAML file for K8s. Does not always work 100%
kubectl get deployment [name] -o yaml --export # returns the deployment in a YAML file
```

### Configure Probes

If large datasets needs to be loaded prior to client access, one can set up a readinessProbes. The pod isn't available until a test is met.

readinessProbes and livenessProbes uses the same syntax. The difference between them is that readinessProbes check only at the begining but livenessProbes is checked intervally. 

Three types of livenessProbes:

* A commands returns 0 exit value
* HTTP returns code between 200-399
* TCP Socket

In this lab, we will make a command one, explicity a cat on a file, that means the file is healthy.

#### Exemple of YAML section

```yaml
## This section add a readinessProbe that check if the pod is ready by "cat"ing the /tmp/healthy file. If it's not there, the pod won't be ready
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
    periodSeconds: 5
#This section adds a "sidecar" to the pod, that will response to the port 8080
- name: goproxy
  image: k8s.gcr.io/goproxy:0.1
  ports:
  - containerPort: 8080
  readinessProbe:
    tcpSocket:
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
