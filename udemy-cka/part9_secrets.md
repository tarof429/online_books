# Part 18: Secrets

To create a secret, you can use an imperative command.

```
kubectl create secret generic dbpass --from-literal=PASSWORD=veryseret
secret/dbpass created
```

If you need multiple values, specify multiple `--from-literal` arguments. If you need to pass in a large number of values, it may be easier to use a file. 

Secrets can also be created using a delcarative approach. When creating secrets this way, the values can either be encrypted or plain-text. 

```
$ kubectl create secret generic dbpass --from-literal=PASSWORD=verysecret --dry-run=client -o yaml > dbpass-secret.yaml
$ cat dbpass-secret.yaml
apiVersion: v1
data:
  PASSWORD: dmVyeXNlY3JldA==
kind: Secret
metadata:
  creationTimestamp: null
  name: dbpass
```

To create encrypted values, on Linux you can echo the value to `base64` and use those values instead.

```
$ echo -n verysecret | base64
dmVyeXNlY3JldA==
```

Once the secret has been applied, you can see the secret values only if you use the `get` command; `describe` will not show it.

```
$ kubectl get secret/dbpass -o yaml
apiVersion: v1
data:
  PASSWORD: dmVyeXNlY3JldA==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"PASSWORD":"dmVyeXNlY3JldA=="},"kind":"Secret","metadata":{"annotations":{},"creationTimestamp":null,"name":"dbpass","namespace":"default"}}
  creationTimestamp: "2021-01-03T23:30:11Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:PASSWORD: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:type: {}
    manager: kubectl-client-side-apply
    operation: Update
    time: "2021-01-03T23:30:11Z"
  name: dbpass
  namespace: default
  resourceVersion: "123191"
  uid: 2026e426-38d0-4366-9683-7c832829c363
type: Opaque
```

But how do you decode the encrypted password? To do this, use `base64` with the `--decode` option.

```
echo -n  dmVyeXNlY3JldA==| base64 --decode
verysecret
```

To inject secrets into a pod, use `envFrom`. 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    envFrom:
      - secretRef:
          name: dbpass
```

