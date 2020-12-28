# Part 14: Scheduling

## Manually adding a nodeName

Normally Kubernetes will have a scheduler to schedule pods on nodes. Without a scheduler, you need to manually schedule a pod to run on a node. This can be done by adding a `nodeName` attribute to a pod.

```
$ cat nginx-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: minikube
```
## Labels and Selectors

Add labels to your template and add seletors to select them in your object. The deployment below defines the `app: nginx` label.

```
$ kubectl create deployment --image=nginx --replicas=3 nginx -o yaml --dry-run=client > nginx-deployment.yml

$ cat nginx-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-bad
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

But be aware that this deployment has an error. We need to match the lable defined in the seelector with that in the template or the deployment will not start.


```
$ kubectl apply -f nginx-deployment.yml
The Deployment "nginx" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"nginx"}: `selector` does not match template `labels`
```

Once we fix this error, we are able to create the deployment. Below, we have added another label `tier=frontend`.

```
$ kubectl describe deploy nginx
Name:                   nginx
Namespace:              default
CreationTimestamp:      Sat, 26 Dec 2020 17:28:21 -0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx,tier=frontend
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
           tier=frontend
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-6fcb56fb4b (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  21s   deployment-controller  Scaled up replica set nginx-6fcb56fb4b to 3
```

We can query the system for nodes with a label.

```
$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
nginx-6fcb56fb4b-b792j   1/1     Running   0          6m58s   app=nginx,pod-template-hash=6fcb56fb4b,tier=frontend
nginx-6fcb56fb4b-ksd8x   1/1     Running   0          6m58s   app=nginx,pod-template-hash=6fcb56fb4b,tier=frontend
nginx-6fcb56fb4b-nm65g   1/1     Running   0          6m58s   app=nginx,pod-template-hash=6fcb56fb4b,tier=frontend
```

The `kubectl` command also has a label selector so we don't need to use `grep`.

```
$ kubectl get pods -l app=nginx,tier=frontend
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6fcb56fb4b-b792j   1/1     Running   0          8m11s
nginx-6fcb56fb4b-ksd8x   1/1     Running   0          8m11s
nginx-6fcb56fb4b-nm65g   1/1     Running   0          8m11s
```

## Taints and Tolerations

Certain nodes can have `taints` to contstrain what pods can be scheduled on them. A pod that is `tolerant` can be scheduled on such a node, while other pods which are not tolerant will be scheduled on other nodes if available. For example, we can taint the `minikube` node with the following:

```
$ kubectl taint node minikube role=maintenance:NoSchedule
node/minikube tainted
```

The next question is, how do we remove the taint? Simply add `-` at the end.

```
$ kubectl taint node minikube role=maintenance:NoSchedule-
node/minikube untainted
```

A node that is tainted can be queried thus:

```
$ kubectl  describe node minikube|grep Taint
Taints:             role=maintenance:NoSchedule
```

We will not be able to schedule deployments:

```
$ kubectl apply -f nginx-deployment.yml
deployment.apps/nginx created
$ kubectl  get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/3     3            0           3s
```

We can edit the running depoyment and manually add tolerations to allow it to be scheduled.

```
 tolerations:
 - key: "role"
   operator: "Equal"
   value: "maintenance"
   effect: "NoSchedule"
 ```

 Afterwards the deployment will be scehduled.

Taint and tolerations do not change the way pods are scheduled. Rather, they only restrict what pods the nodes can accept.

## Node Selectors

In a multi-node cluster, you may have nodes of varying capabilities. You may want certain objects to be placed on certain nodes depending on their requirements. One way to do this is to make use of node selectors. A node selector is a label specified in an object. If it matches a node then the object will be scheduled on that node. We can make use of the command option `label` to add a label to a node.

To illustrate, we first start minikube with two nodes.

```
minikube start --memory 8192 --cpus=4 --nodes=2
```

We then label one of the nodes with a label in a key-value format.

```
$ kubectl label nodes minikube-m02 size=large
$ kubectl get nodes -l size=large
NAME           STATUS   ROLES    AGE    VERSION
minikube-m02   Ready    <none>   133m   v1.20.0
```

We can then add a node selector to a pod so that it will be scheduled on the minikube node.

```
$  cat nginx-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    size: large
```

This pod will be scheduled on the second node.

```
$ kubectl get pod nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          14s   10.244.1.2   minikube-m02   <none>           <none>
```

## Node Affinity

Node selectors are limited to basic AND expressions that seek to limit how they are expressions. You cannot say, for example, to schedule a pod on node A or node B. For more advanced expressions, you can use node affinity. With node affinity, you can schedule a node to run on a node that has a medium or a arge size label.

You can use the `explain` command to get information about the syntax:

```
kubectl explain pods.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms.matchExpressions
```

A pod that can run on a pod with a label of either `large` OR `medium` looks like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: "size"
              operator: "In"
              values:
              - "large"
              - "medium"
```

Be very careful when specifying nodeAffinity. It is very easy to mistype something or forget that an object is an array. If you're creating more than one pod or deployment and one of them has a noideAffinity, it may be helpful to just copy the first file to avoid typing. If you have a deployment and you want to add nodeAffinity to, you could:

```
 echo kubectl explain pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms >> mynginx-deployment.yaml
 ```

 You can use a combination of taints and nodeAffinity to prevent pods from being scheduled on nodes and to constrain certain pods to be scheduled on a node.

First we taint the node:

```
kubectl taint node minikube-m02 Color=Red:NoSchedule
$ kubectl describe node minikube-m02|grep Taint
Taints:             Color=Red:NoSchedule
```

We also add a label on our node:

```
 kubectl label nodes minikube-m02 Color=Red
```

Then we create a pod with a toleration so that it can accept a node with the label `Color=Red` and also nodeAffinity for the same node.

 ```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
 tolerations:
 - key: "Color"
   operator: "Equal"
   value: "Red"
   effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: "Color"
              operator: "Equals"
              values:
              - "Red"
```

## Resources

By default, a pod will consume 0.5 CPU and 356Mi memory. If you want to request more resources, then you need to add a resources section in the `containers` section.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "0.5"
      limits:
        memory: "2Gi"
        cpu: "1"
```


If the pod doesn't have enough memory, it can fail to start. For example, let's create a deployment for mysql.

create deployment --image=mysql mysql -o yaml --dry-run=client > mysql-deployment.yml

```yml
$ cat mysql-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        resources:
          requests:
            memory: "6Mi"
            cpu: "0.5"
          limits:
            memory: "6Mi"
            cpu: "0.5"
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "secret"
```

Because we only request 6Mi memory, the pod will fail to start due to out-of-memory.

```
$ kubectl get pod
NAME                     READY   STATUS      RESTARTS   AGE
mysql-6df7867f89-w2kqc   0/1     OOMKilled   5          3m16s
```

To fix thsi, increase the memory size. Below is a snippet.

```yml
        resources:
          requests:
            memory: "64Mi"
            cpu: "0.5"
          limits:
            memory: "128Mi"
            cpu: "0.5"
```

