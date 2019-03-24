# Troubleshooting

- [Troubleshooting](#troubleshooting)
  - [Ongoing Constant Change](#ongoing-constant-change)
  - [Basic Troubleshooting Flow](#basic-troubleshooting-flow)
    - [Pods](#pods)
    - [Node and Security](#node-and-security)
  - [Monitoring](#monitoring)
    - [Prometheus](#prometheus)
    - [Fluentd](#fluentd)
    - [OpenTracing](#opentracing)
    - [Jaeger](#jaeger)
  - [Logging Tool](#logging-tool)


## Ongoing Constant Change

Keep in mind that K8s is constantly changing and isn't owned by a corporation.

## Basic Troubleshooting Flow

### Pods

Running a shell inside the container can give us great details

```bash
kubectl exec -ti [name] -- /bin/sh
# or
kubectl logs [name]
# to view the pods std-outs
```

Verify if the pod is running and not pending. Pending means it's waiting for something. `kubectl describe pod` 

And if nothing if found, one can check the logs of the container itself with `kubectl logs [pod-name] [container-name]

### Node and Security

Disabling security for testing is a good idea to determine what's wrong

Internode networking can be an issue too. 

## Monitoring

* Heapster: part of the K8s project, can be deployed in a cluster

* Prometheus: part of the CNCF. It's a k8s plugin

### Prometheus

* Focus on metrics and alerts.
* Provides a time-series database, queries and alerts to gather infos. 

### Fluentd

* Unified logging layer, which can collect logs from over 500+ plugins. 

### OpenTracing

* Single standard mechanism for everything to propagate  tracing among all services

### Jaeger

* A tracing system developed by Uber.
* Focus on distributed context propagation, transaction monitoring and a root cause analysis


## Logging Tool

* Elasticsearch, logstash, Kibana Stack: quite common
* Cluster-wide: **Fluentd**: part of the CNCF, and combine well with Prometheus.

Setting a Fluentd is a nice exercise to understand the DaemonSets. Fluentd aggregate the logs, feed them into an elasticseach and visualize it through Kibana

