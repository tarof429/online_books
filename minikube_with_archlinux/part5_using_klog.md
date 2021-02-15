# Part 5: Using klog

While on my journey to learn Kubernetes, I came across an article regarding klog in the Kubernetes blog. This library is used to Kubernetes log events in a structured format, and as I'm mildly familiar with logging, decided to see if I could make use of it.

## hellokube with klog

I've refactored hellokube to use klog. To deploy hellokube v1.1, first remove v1.0 by running `kubectl delete -f .`. Then checkout  https://github.com/tarof429/hellokube/tree/v1.1 and run `kubectl apply -f .`. The application is still accesible from http://172.17.0.2. To see the new log messages, find the name of the deployment and run `kubectl logs <id>` against it. Below is my output.

```bash
$ kubectl logs hellokube-deployment-64f6db6dcd-vl5fb
I0924 17:22:59.348455       1 main.go:17] "Starting pod...
```

## References

https://kubernetes.io/blog/2020/09/04/kubernetes-1-19-introducing-structured-logs/

https://github.com/kubernetes/klog
