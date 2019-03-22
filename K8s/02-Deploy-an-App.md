# Deploy an App

- [Deploy an App](#deploy-an-app)
  - [K8s Deployments](#k8s-deployments)
  - [Kubectl](#kubectl)
  - [Cheatsheet](#cheatsheet)

## K8s Deployments 

To deploy containerized applications, we need a K8s **Deployment** configuration. This instructs K8s how to create/update instances of the application.

Once the app instances are created, the K8s Deployment Controller continuously monitors those instances. If a Node goes down/gets deleted, the controller replaces the instance with an instance on another Node in the cluster. This provides a self-healing mechanism to adress machine failure or maintenance.

## Kubectl

Kubectl uses the Kubernetes API to interact with the lucster. When creating a Deployment, one should specify the container image for the application and the # of replicas that it must run on. 

## Cheatsheet

```bash
kubectl run NAME --image=image [--port=8080]    # Creates a new deployment.
kubectl get deployments                         # List the deployments.
kubectl proxy                                   # Create a proxy that forward comminucations into the cluster-wide, private network.
```