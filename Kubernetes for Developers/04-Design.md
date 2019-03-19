# Design

- [Design](#design)
  - [Introduction](#introduction)
  - [Traditional Application - Considerations](#traditional-application---considerations)
  - [Decoupled Resources](#decoupled-resources)
  - [Transience](#transience)
  - [Managing Resource Usage](#managing-resource-usage)
  - [CPU](#cpu)
  - [Memory](#memory)
  - [Ephemeral Storage](#ephemeral-storage)
  - [Multi-Container Pods](#multi-container-pods)
    - [Sidecar Container](#sidecar-container)
    - [Adapter Container](#adapter-container)
  - [Ambassador](#ambassador)
  - [Jobs](#jobs)
  - [Labs](#labs)

## Introduction

This chapter will touch how to define an application's resource requirements and understanding some design patterns (Ambassador, Adapter, Sidecar)

## Traditional Application - Considerations

Optimal change of an app for K8s is not only to containerize them. Most of older applications are runned without any reset in mind. Older application has a lot of tweaks in their Apache web server. 

## Decoupled Resources

Decoupled resources is integral to K8s. Instead of using an dedicated port/socket, the goal is to be decoupled from other ressources. Instead of hard coding, an intermediary like a Service, enable conncetion/reconnection. Enabled the devs to optimize small part of app without worrying 

## Transience

Every component should be developped with keeping in mind that they will die and rebuilt everytime. 

## Managing Resource Usage

The kube-scheduler uses the PodSpec to determine the best ndoe for deployment. By default Pods use as much CPU/memory as the workload requries, acting like any Linux processes. Throught the use of resource request, the scheduler will only schedule a Pod to a node if resource exist to meet all requests on that node. 

Monitoring the resource usage cluster-wide is not an included feature of K8s.

## CPU

One CPU in K8s is equivalent to 1 AWS vCPU, 1 GCP Core, 1 Hyperthread on a bare-metal Intel processor w/ Hyperthreading

To allow more resource (CPU) than allowed, one can set

`spec.containers[].resources.limits.cpu` and `specs.containers[].resources.requests.cpu`

## Memory

In Docker engine, the `limits.memory` value is converted to an integer value and becomes the value to the `docker run --memory [value] [image]` . 

To allow more resource (MEMORY) than allowed, one can set

`spec.containers[].resources.limits.memory` and `specs.containers[].resources.requests.memory`

## Ephemeral Storage

Files,logs and emptydir, as well as a kubernetes cluster data

To allow a container to use more than the limit than allowed, one can set

`spec.containers[].resources.limits.ephemeral-storage` and `specs.containers[].resources.requests.ephemeral-storage`

## Multi-Container Pods

The idea of multi-container pods is against the architectural idea of decoupling as much as possible. One could run an entire OS inside an container, but it'd lose the scalability. However, there's some reason to do it.

### Sidecar Container

Add some functionality not present in the main container. Instead of bloating code, just adding a container to handel a function (logging). e.g, prometheus monitoring and Fluentd leverage sidecar containers to collect data.

### Adapter Container

An adapter container will modify data, either on ingress/egress, to match some other need. An adapter would be an efficient way to standardize the output of the main cnotainer to be ingested by the monit tool, w/o having to modify the monitor/app. An adapter container can transforms multiple apps to a singular viwe.

## Ambassador

An ambassador allows for access to the outside world w/o having to implement a service or another entry in an ingress controller:

* Proxy local connection
* Reverse proxy
* Limits HTTP requests
* Re-route from the main container to the outside world

It's an Open-Source, k8s-native API gateway for microservices built on Enjoy.

## Jobs

Jobs are part of the *batch* API group. They are used to run a set # of pods. If a pods fails, it'll be restarted. Can be seen as a way to do batch-processing on k8s, but that's not their true reason to live. A job spec have a parallelism/completion key. If ommited, they will be set to one, if present, the parallelism number will set the # of pods that can run concurrently, and the completion number will set how many pods need to run succesfully for the job itself to be done. Several Job patterns can be implemented.

## Labs

