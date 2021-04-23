# Part 16: Network Troubleshooting

## Network plugin in Kubernetes

The exam won't ask you to install a network plugin. It's good to know that the documentation only refers to instaling Weave network (see https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/) by running the following command:

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Other solutions include: Flannel and Calico. 

Note: You must have a network plugin installed in your cluster for components to be able to talk to each other. Check the pods in the kube-system namespace and verifyt that the weave-net daemonset ins running. If not, install weavenet. 

## DNS in Kubernetes

- Check if CoreDNS pod is installed. You can check if the core-dns pod is running in the kube-system namespace. 
- If the core-dns pod is running and is in a CrashLoopBackOff or Error state, there are several things you can try:
    - Upgrade to a newer version of docker (maybe yum update?)
    - Disable SELinux. to view the SELinux status on Ubuntu, run `sestatus`. To check the current running mode, run `getenforce`. Run `setenforce 0` to make SELinux permissive. You should also set SeLINUX=permissive in /etc/selinux/config and reboot. 
    - Modify the coredns deployment to set `allowPrivilegeEscalation` to true.
For all of these, see https://github.com/coredns/deployment/blob/master/kubernetes/FAQs.md. 

## Kube-proxy

Kube-proxy is responsible for networking within the cluster. 

- Check the statue of the kube-proxy pod by running `kubectl describe kube-proxy -n kube-system`. 
- Look at logs: `kubectl logs kube-proxy -n kube-system`. 
- Check configmap is correctly defined
- Check kube-proxy running inside the container: `netstat -plan | grep kube-proxy`.
- Check to see if kube-proxy pod is configured to load its configuration file at the location specified in its config map.

See:

- https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/
- https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/