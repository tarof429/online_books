# Part 20: Upgrading

## Upgrading the Underlying OS

When upgrading a node, it's a best practice to first `drain` it. Doing so means the scheduler makes sure new pods arent' scheduled on it, and pods that are currently running on the ndoe will be scheduled on another node. 

Once done, you should `uncordon` the node to make it available for scheduling again. This does not mean the pods that were previously running on it will be rescheduled on this node.

An alternative to `drain` and `uncordon` is the `cordon` command. This command marks a node as unschdulable. New pods will not be scheduled on a node that has beeen cordoned. 

{% hint style="info" %}
To find out what node a pod is running on, run:
{% endhint %}

```
kubectl get pods -o wide
```

## Upgrading Kubernetes

To upgrade a cluster, we must first upgrade kubeadm. 

```
apt-get upgrade -y kubeadm=1.12.0-00
```

Now we can upgrade the control plane.

```
kubeadm upgrade apply v1.12.0
```

However, at this point the kubelet on the masdter node is still at older versions. To upgrade the kubelet:

```
apt-get upgrade -y kubelet=1.12.0-00
```

Now we just have to restart the kubelet service.

```
systemctl restart kubelet
```

 For the worker nodes, we must first drain the node. Since the worker nodes are on remote machines, we need to SSH to them first.

 ```
 kubectl drain node-1
 ```

 Then install the latest kubelet and restart it.

 ```
 apt-get upgrade -y kubelet=1.12.0-00
 systemctl restart kubelet
 ```

 We need to do this for all the worker nodes.

{% hint style="info" %}
 When upgrading Kubernetes, it's a best practice to only upgrade minor versions at a time.
{% endhint %}

## Backup and Restore

### Backing up etcd the easy way
It is important to backup all definition files. That way, if the cluster goes down or you need to recreate the cluster from scratch, you can use the back up copies of the definition files. 

At times however, some definitions are created using the imperative approach. In this case, you do not have the definitions stored in source control. To make a backup of these definitions, you should use `kubectl` to save the definition to a file.

For example, lets say we have a secret that was created imperatively. 

```
$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
dbpass                Opaque                                1      6d4h
```

To save this to a file, simply use `kubectl get`.

```
kubectl get secret dbpass -o yaml > dbpass-secret-backup.yaml
```

Then to restore it, simply apply the secret before creating the pod that refers to it.

There are other strategies as well for backing up defintions, such as backing up all definitions to a file.

```
kubectl get all --all -namespaces -o yaml > all-definitions.yaml
```

### Backing up etcd directly

To get working with etcd backups, Make sure that `etcdctl` is installed. In minikube, you can ssh to the master node and install `etcd-client`.

```
minikube ssh
sudo apt-get install -y etcd-client
```

Once etcdctl is installed, you want to make sure you can list the members of the cluster.

```
$ sudo ETCDCTL_API=3 etcdctl member list --cacert=/var/lib/minikube/certs
/etcd/ca.crt --cert=/var/lib/minikube/certs/etcd/server.crt --key=/var/lib/minikube/certs/etcd/server.key --endpoints=https://127.0.0.1:2379
aec36adc501070cc, started, minikube, https://192.168.49.2:2380, https://192.168.49.2:2379
```

It's important that you specify ETCDCTL_API; otherwise, the command-line arguments are completely different and it will not work.

The major problem with using etcdctl is knowing the location of the certs. This can be different for every distribution of etcd; what is set for minikube may be completely different for another vendor. How do you know where they are? Well, one place to look is the definition for the static pod for etcd. If the kubelet is running on the master, look under /etc/kubernetes/manifests/etcd.yaml. 

To save the backup:

```
ETCDCTL_API=3 etcdctl snapshot save ---cacert=/var/lib/minikube/certs
/etcd/ca.crt --cert=/var/lib/minikube/certs/etcd/server.crt --key=/var/lib/minikube/certs/etcd/server.key --endpoints=https://127.0.0.1:2379
aec36adc501070cc, started, minikube, https://192.168.49.2:2380, https://192.168.49.2:2379 /etcd-snapshot.db
```

{% hint style="info" %}
Do not attempt to memorize the location of the certs. Put a space between the argument and the value and let the shell autocomplete the path.
{% endhint %}

To restore etc from a snapshot, first stop the API server and use the `restore` command. 

```
systemctl stop kube-apiserver
```

Then restore etcd from the snapshot. You can use the previous command as a "template". Note that `--data-dir` needs to be a different directory than the location of the backup. The argument `--initial-cluster` refers to the name and IPs of the master nodes. The argument `--initial-cluster-token` can be specified at this time to prevent corruption of etcd. 

```
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --initial-cluster="master=https://192.168.5.11:2380" --name="master" --initial-cluster-token etcd-cluster-1 --initial-advertise-peer-urls https://127.0.0.1:2380  --data-dir=/var/lib/etcd-from-backup /opt/snapshot-pre-boot.db
```

{% hint sytel="info"}
According to the etcd github page at https://github.com/etcd-io/etcd, 2379 is the default port for client requests, 2380 is the default port for peer communication.
{% endhint %}

Afterwards, you need to modify the static pod definition of etcd under /etc/kubernetes/manifests. The etcd command line arguments need to be modified to point to the new data directory and mount path to this directory.

Also you need to point to the cluster token that was just created. 

{% hint style="info" %}
If the master node is running a kubelet, you can view /etc/kubernetes/manifests/etcd.pod which has a list of arguments that's very similar to what is needed to perform the backup!
{% endhint %}


As soon as you save the file, etcd should be redeployed.

Finally, run `systemctl daemon-reload` to update the service before starting etcd again. 

{% hint style="info" %}
It can be convenient to export ETCDTL_API since it is required by etcdtl.
{% endhint %}

In the exam, you won't know whether what you did was correct. You need to run `kubectl describe pods` to verify that the pod was created with the correct name and image.