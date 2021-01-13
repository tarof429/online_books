# Part 19: Init Containers

A pod can have multiple containers. However, there is also a type of container called an init container which runs before app containers. Init containers can be used to run utilities or scripts before the app container runs.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  initContainers:
  - image: busybox
    name: prepare-nginx
    command: ['sh', '-c', '/setup.sh']
```

