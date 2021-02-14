# Part 22: Storage

Persistent volume (PV) refers to a volume from which persistent volume claims (PVC) can be created and used by pods. A persistent volume may be created per pod, or used by many pods. 

For example, the persistent volume below creates a 25 MB volume under my home directory.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 50Mi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /data/storage-example-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
  selector:
    matchLabels:
      name: "example-pv"
```

Now in theory, this tells Minikube to carve 10 MB from the `example-pv` persistent volume. However when you apply this configuration, Minikube will create a PVC automatically and ignores the PVC. In other Kubernetes implementations, a 10MB PVC will be created from the PV.

One you create a PVC, you can reference it from a pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: mypod
spec:
    containers:
    - name: myfrontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
        name: mypd
    volumes:
    - name: mypd
        persistentVolumeClaim:
        claimName: myclaim
```

Instead of creating a PV, you can create a StorageClass. If you create a StorageClass, then the PV will automatically be created.