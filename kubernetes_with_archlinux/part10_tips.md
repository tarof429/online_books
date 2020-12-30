# Tips

## How to find the image used by a pod

Suppose you have a pod called docker-trac-deployment-7d799b7b95-8dn85.

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

$ kubectl apply -f nginx-pod.yml
pod/nginx unchanged

$ kubectl describe pod nginx |grep -i image
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:6b1daa9462046581ac15be20277a7c75476283f969cb3a61c8725ec38d3b01c3
  Normal  Pulling    85s   kubelet            Pulling image "nginx"
  Normal  Pulled     74s   kubelet            Successfully pulled image "nginx" in 11.360929107s
  ```

## Best .vimrc configuration

```
cat ~/.vimrc
set expandtab
set tabstop=4
set showmatch
set nu
set showmatch
set autoindent
```

## Fastest way to find which nodes the pods are running on

```
kubectl get pods -o wide
 $ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          7m29s   172.18.0.4   minikube   <none>           <none>
```

## Fastest way to find the number of containers running in a pod

```
kubectl get pods -o wide
```

## Fastest way to create a pod

```
kubectl run redis --image=redis
```

## Fastest way to generate a yaml file for a pod

```
kubectl run redis --image=redis --dry-run=client -o yaml > redis-pod.yml
$ cat redis-pod.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### Generators

If you're creating an object, any object, besides a pod, then you can look at `generators`. These are objects created by the `create` command.

To begin with, type:

```
kubectl create
```

You will get a long list of options (including examples). For example:


```
 kubectl create deploy --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yml
 ```

 After this file is generated, you can use vi to modify it as needed (for example, set the number of replicas). It is best to become very familiar with this commmand as a quick way to generate yaml files.

For example, we can quickly create a yaml file for a namespace.

```
kubectl create namespace mynamespace --dry-run=client -o yaml > mynamespace.yaml
```

## Fastest way to modify a running pod/replicaset

```
kubectl edit pod redis
kubectl edit rs redis
```

For a replicaset, this is the fastest way to change the number of replicas. Alterntively:

```
kubectl scale --replicas=2 rs/new-replica-set
```

### How to see all objects

```
kubectl get all
```

### How to find an object if you don't know its namespace

```
kubectl get pods --all-namespaces | grep <pod name>
```

### How to lookup a section of the yaml

```
kubectl explain pods.spec.tolerations
```

### How to make changs to a pod

If a pod is already deployed and you need to make a change, it may be best to export the configuration to a file. Delete the running pod, modify the file, and apply it again with any changes.

```
kubectl get pod nginx -o yaml > another-pod.yml
```

### How to find the location where static pods are deployed

In some implementations, this can be found as an argument passed to the kubelet executable. 
```
ps -ef | grep kubelet
```

Other times, you need to look at the `config.yaml` passed to the kubelet.