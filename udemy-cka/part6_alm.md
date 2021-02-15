# Part 15: Application Lifecycle Management

## Rolling updates and rollbacks

When you do a deployment, you actually trigger a rollout. The next time you update the image, you trigger a new rollout (for example, revision 2). 

Note: rollouts apply to deployments, daemonsets, and statefulsets. They do not apply to pods.

```
$ kubectl apply -f nginx-deployment.yml
deployment.apps/nginx created
$ kubectl rollout status deployment/nginx
deployment "nginx" successfully rolled out
```

To see all the rollouts for a given resource, use the keyword `history`.

```
$ kubectl rollout history deployment/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
```

By default, kubernetes will do a `rolling` update (as opposed to an `recreate` update) for deployments. This ensures high availabilty of the application. Simply use the kubectl `apply` command to do rolling updates.

Behind the scenes, Kubernetes will create a new replicaset when doing rolling updates. After all the pods are created in the new replicaset, each pod in the running replicaset will be replaced one-by-one by a new revision of the pod. 

It is possible to rollback the previous rollout. To do this, use the `rollout undo` command. 

Note: if you edit a deployment and change the rollout strategy from RollingUpdate to Recreate, you also need to remove the rollingUpdate section.

To update the image of a deployment, use the `set image` command.

```
kubectl set image  deployment/nginx nginx=nginx:1.9.1
```