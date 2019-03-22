# Explore your App

- [Explore your App](#explore-your-app)
  - [K8s Pods](#k8s-pods)

## K8s Pods

A Pod is K8s abstraction that represents a group of one or more app container (such as Docker/rkt) and some resource such as:

* Shared storage, as Volume
* Networking, as unique cluster IP address
* Information about how to run each container

A pod models an app-specific logical host