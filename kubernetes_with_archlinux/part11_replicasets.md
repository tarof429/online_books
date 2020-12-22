# Part 11: Replicasets

## Example

Below is an example of a replicaset:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

To apply it:

```
kubectl apply -f frontend-replicaset.yml
```

To check the state:

```
kubectl  describe replicaset frontend
```

or

```
kubectl  describe rs/frontend
```

To scale it:

```
kubectl scale --replicas=2 rs/frontend
```

Tip:

In vi, you should set the following when editing yml files.

```
:set expandtab
```

To edit it:

```
kubect edit rs/frontend
```


