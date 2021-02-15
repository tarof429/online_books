# Services

Services are objects which connect components in an application together. Pods use them to talk to other pods, and pods use them to connect to external data sources. They also allow us to connect to pods from outside the node. 

There are three types of services:

1) Node port: The node port is a port on the node which can be used to access a pod in the node. If you dont' specify a port, a port will be assigned automatically. 
2) Cluster IP: An IP address that can be used to reach a pod within the node. 
3) Load Balancer: Allows us to reach a node from outside the cluster among a pool of nodes. 

## How to create a NodePort service

```
 kubectl create svc nodeport my-ns --tcp=4678:8080
 ```

 What this does is create a service with a target port of 8080 and a port of 4678. In this case, 8080 is the port which the pod expects incoming connections, but it is mapped to port 4678 and is used by other pods to connect to the pod.

 ## Relationship between services and nodes

 In order to associate a service with a pod, you need to make use of selectors. A selector associates a service with labels that are set for a pod. For example, if a pod had a label of `app: my-ns`, then the service should have a selector with an entry for `app: my-ns`. 

 ## How to create a ClusterIP service

The cluster IP server connects a port to a target port and offers load balancing between the nodes. You can refer to the cluster IP using either the cluster IP or the name of the service.

```
kubectl create svc clusterip my-cs --tcp=5678:8080
```

### Load Balancer service

The load balancer service is used to connect external load balancers such as that provided by AWS with pods.