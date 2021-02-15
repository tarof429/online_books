# Namespaces

Unless specified, Kubernetes objects are created in the `default` namespace. Namespaces are a way to isolate objects from either. For example, you could have a namespace called `dev` and another namespace callled `prod`. Any objects in `dev` are isolated from objects from objects in `prod`. Objects in one namespace can, however, interact with each other if needed. For example, you may have a database in one database that is shared among the `dev` and `prod` database.

To list namespaces:

```
kubectl get ns
NAME              STATUS   AGE
default           Active   79d
kube-node-lease   Active   79d
kube-public       Active   79d
kube-system       Active   79d
```

Here, kub-system is a namespace that is internal to Kubernetes, kube-public is a namespae that is used by other namespaces, and default is the namespace where objects are created by default.

To create a namespace:

```
kubectl  create namespace mynamespace --dry-run=client -o yaml > mynamespace.yml
```

or:

```
kubectl  create namespace prod
```

To apply the namespace:

```
kubectl  apply -f mynamespace.yml
namespace/mynamespace created
```

To create a pod in a namspace (declaratively):

```
kubectl run hello-world --image=hello-world --namespace mynamespace
pod/hello-world created
```

To view the pod in the new namespace:

```
kubectl  get pods --namespace mynamespace
NAME          READY   STATUS             RESTARTS   AGE
hello-world   0/1     CrashLoopBackOff   2          39s
```

To change your namespace permanently:

```
kubectl config current-context
minikube
kubectl config set-context minikube --namespace=mynamespace
kubectl get pods
```

Resource quotas can be created for namespaces.

```
kubectl create quota myquota --dry-run=client -o yaml > myquota.yml
cat myquota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myquota
spec: {}
status: {}
```

Then add the namespace in the metadata section.

```
cat myquota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myquota
  namespace: mynamespace
spec: {}
status: {}
```

Within the same namsapce, objects can refer to other objects (such as a service) directly using their object name. For example, within the namespace `marketing`, the `blue` pod can access the `db-service` running in the same namespace using the url `db-service:3306`. 

If `blue` pod is in the `marketing` namespace and `db-service` is running in the `dev` namespace, then, use the format ` <service-name>.<namespace-name>.svc.cluster.local`, or `db-service.dev.svc.cluster.local`. 

