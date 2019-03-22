# Exposing Application

- [Exposing Application](#exposing-application)
  - [Service Type](#service-type)
  - [Services Diagram](#services-diagram)
  - [Service Update Pattern](#service-update-pattern)
  - [Accessing an Application with a Service](#accessing-an-application-with-a-service)
  - [ClusterIP](#clusterip)
  - [NodePort](#nodeport)
  - [LoadBalancer](#loadbalancer)
  - [External Name](#external-name)
  - [Ingress Resource](#ingress-resource)

## Service Type

There are 4 types of services:

* ClusterIP: Default one, only provides access internally
* NodePort: Great for debugging, or when a static IP is necessary
* LoadBalancer: Created for passing requests to a cloud provider like GKE or AWS. Even without a cloud provider, the address is made available to public traffic and packets are spread among Pods.
* ExternalName: Newer service. It has no selectors, no ports or endpoint. It allows the return of an alias to an external service. Redirection happens at the DNS level, not via proxy or forward.

`kubectl proxy` creates a local service to access a ClusterIP. Useful for troubleshooting or dev work.

## Services Diagram

`kube-proxy` runs on cluster nodes and watches the API server service resources. Since 1.1, the iptables proxy was added and became the default mode in 1.2.

The iptables proxy mode makes that the `kube-proxy` continues to monitor the API server for changes in Service/Endpoint and updates rules for each objet when created or removed. One limitation to the new mode is an inability to connect to a Pod should the original request fail, so it uses a readinessProbe to ensure all is functional before connection.

Allows for ~ 5000 nodes. 

Starting from 1.9, the starting mode is ipvs (beta). It works in the kernel space for greated speed, and allows for a configurable load-balancing algorithm. Can be useful for +5000 nodes clusters. 

The `kube-proxy` mode is configured via flag during init, such as `mode=[iptables | IPVS | userspace]`


## Service Update Pattern

Labels are used to determine which Pods should receives traffic from a service. It can be dynamically updated.

The default pattern is for a rolling deployment, is that the new Pods receive traffic along with previous versions of the application.

If there's a difference in the app, a more specific label is better (version number). When the deployment creates a new replication controller for the update, the label would not match.

Once the new Pods have been created, we would edit the labels for the Service connects and then traffic would shift to the new and ready version.

## Accessing an Application with a Service

```bash
kubectl expose deployment/nginx --port=80 --type=NodePort # creates a service for the nginx deployment. 
kubectl get svc nginx -o yaml # this will give us the new port that we can then access.
```

The expose command create a service that used port 80 and then generated a random port for every node. 

## ClusterIP

For inter-cluster communication, frontends talking to backends can use clusterIP.

```yaml
spec:
  clusterIP: [ip]
  ports: 
  - name: "443"
    port: 443
    protocol: TCP
    targetPort: 443
```

## NodePort

NodePort is a simple connection from a high-port routed to a ClusterIP using iptables/ipvs. Creating a NodePort will create a ClusterIP by default. Traffic is routed from the NodePort to the ClusterIP

Then it will be accesible by [ip]:[node]

```yaml
spec:
  clusterIP: [ip]
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 31070
    port: 80
    protocol: TCP
    targetPort: 800a0
  selector:
    io.kompose.service: nginx
  sessionAffinity: None
  type: NodePort
```

## LoadBalancer

Creating a LaodBalancer will generate a Nodeport, which will then create a ClusterIP.

It will send an async call to a load balancer (usually a cloud provider). The external-ip will remain <Pending> until the laod balance returns something.

```yaml
Type: LoadBalancer
loadBalancerIP: 12.34.561.123
clusterIP: [ip]
ports:
- protocol: TCP
  Port: 80
```

## External Name

The use of an ExternalName service is to point to an external DNS server. Handy when using a resource external to the cluster. Maybe before full integration

```yaml
spec: 
  Type: ExternalName
  externalName: ext.db.example.com
```

## Ingress Resource

API object containing a list of rules matches against all incoming requests. The HTTP request must match both the host and the path declared in the ingress.

Handlinga few services can be easily done. But may creates a lot of inefficiencies. An Ingress Controller manages ingress rules to route traffic to existing services. 

