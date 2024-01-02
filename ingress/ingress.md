# Ingress

## Overview
Uptil now we have been providing external connections through  external service (Load Balancer).
This is only good for testing, we want a secure connection.

So we use Ingress and an internal service. Ip address and port is not opened.

Request -> Ingress -> Internal service -> Pod

## Components 
Ingress components.
- kind: Ingress
- host: my-app (the ip or the domain of the entry point node/server)
    - Should be a valid domain address.
    - Node's ip address which is the entry point of the cluster.
    - host name of the outside servcer.
- serviceName: myapp-internal-service (The internal service)
- servicePort: (The port on the internal service)

## Ingress controller
An implementation of Ingress. 
- It is also a Pod.
- Does evaluation and procesing of ingress rules.
- Managaes redirection.
- The entry point of the cluster.

## Proxy server
The cloud providers will give us load balancers that will then forward the request to Ingress controller pod.

For our own implementation we need a proxy server that will forward the requests to ingress controller.

## Intsall Ingress controller
Multiple implemenations. We will use minikube here.
```
$ minikube addons enable ingress
```
Automatically starts the K8s nginx kubernetes implementation.

```
$ kubectl get pod -n kube-system
```

```
$ kubectl get ns
```



