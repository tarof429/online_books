# Part 14: Networking

## Networking commands

Below are some commands that you should know about networking in Linux.

1. ip link - Lists interfaces on the host

2. ip addr - lists the IP addresses assigned to those interfaces

3. ip addr add <ip> dev eth0 - sets an IP address to an interface. For example:

    ```
    ip addr add 192.168.1.10/24 dev eth0
    ```

4. ip route is used to view the routing table

5. ip route add is used to add entries into the routing table. for example:

    ```
    ip route add 192.168.1.0/24 via 192.168.2.1
    ```

6. cat /proc/sys/net/ipv4/ip_forward shows whether the host is configured as a router

Below are some commands specific to DNS

1. nslookup <hostname> - Queries the DNS server for a hostname

2. dig <hostname> - Another DNS server query program. It can provide the type of records

## Network namespaces

See https://matrix.ai/blog/linux-network-namespaces/

## Problems

1. Q: There are several nodes in a cluster. How do you find the network interface for the master node?

A: If you run `kubectl get nodes -o wide`, it will print information about the nodes including the internal IP addrress of the controlplane (master node). This IP is assigned to an interface, which we can find out if we run `ip addr`. 

```
kubectl get nodes -o wide
NAME           STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
controlplane   Ready    master   23m   v1.18.0   172.17.0.13   <none>        Ubuntu 18.04.5 LTS   4.15.0-122-generic   docker://19.3.13
node01         Ready    <none>   23m   v1.18.0   172.17.0.46   <none>        Ubuntu 18.04.5 LTS   4.15.0-122-generic   docker://19.3.13
```

Then we can run `ip addr` on the master node and grep for the IP address. It should be bound to an interface.

```
controlplane $ ip addr | grep -B 3 172.17.0.13
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:42:ac:11:00:0d brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.13/16 brd 172.17.255.255 scope global ens3
```

2. Q: How do you find the IP address of the default gateway?

A: Run:

```
ip route show default
default via 172.17.0.1 dev ens3 
```

or

```
ip route |grep default
```

3. Q: How do you find the port the kube-scheduler is listening to on the master node?

A: Run the following on the master node:

```
netstat -ntlp
```

As described at https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/, port 10251 is used for HTTP (which is deprecated), 10259 is used for HTTPS.

Is there another way?

4. Q: What port does etcd listen to in a k8s cluster?

A: 2379

5. Q: If there is a pod on node1 and another pod on node2, how can you configure the network so that the pods can communicate with each other? The pods will be on different subnets.

A: To do this, you create a route. On node1 to node 2. You tell node1 that, in order to reach a pod on node2, use the public IP assigned to node2.

```
ip route add 10.244.2.2 via 192.168.1.12
```

Where 10.244.2.2 is the IP address of the pod, and 192.168.1.12 is the IP address of node2.

However, this solution is not scalable and a better way is to configure the router and define which network is reachable by which gateway (node).

```
Network         Gateway
10.244.1.0/24   192.168.1.11
10.244.2.0/24   192.168.1.12
10.244.3.0/24   192.168.1.13
```

6. Q: Do pods get the same range of IP addresses as the nodes?

A: No, pods get IP address from the network plugin. To find the range of IP addresses, run: kubectl logs <weave-pod-name> weave -n kube-system and look for ipalloc-range

7. Q: How do you find the IP range configured for the services within the cluster?

A: Inspect the setting on kube-api server by running on command cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range. Does the kube-api server create services?

8. How do you find the proxy used by kube-proxy?

A: Examine the logs of the kube-proxy by running *kubectl logs <kube-pod-name> -n kube-system and search for something like *iptables*. 

9. How do you find out how the cluster ensures that a kube-proxy pod runs on all nodes in the cluster?

A: Inspect the kube-proxy logs and identify how they are deployed (for example, daemonset). 