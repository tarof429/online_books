# Part 6: Kubeiostat: A simple Python REST service on Kubernetes

In this section, we will build a REST service using Python and deploy it to Kubernetes. The application that we will build uses the iostat Python library to gather system metrics and make them avaialable to clients using REST. It is avaiable from https://github.com/tarof429/kube-iostat. 

To run as a docker container:

```bash
$ sh ./build.sh
[taro@zaxman kube-iostat (master)]$ sh ./build.sh
Sending build context to Docker daemon  15.68MB
Step 1/6 : FROM python:3
 ---> 28a4c88cdbbf
Step 2/6 : WORKDIR /src
 ---> Using cache
 ---> 1e8546c06f2f
Step 3/6 : COPY main.py requirements.txt ./
 ---> Using cache
 ---> 7fd6067a6080
Step 4/6 : RUN pip install virtualenv &&     virtualenv env &&     . ./env/bin/activate &&     ./env/bin/pip install -r requirements.txt
 ---> Using cache
 ---> 8d74f83b2ecf
Step 5/6 : CMD ["./env/bin/python", "main.py"]
 ---> Using cache
 ---> d393720fbcb5
Step 6/6 : EXPOSE 5000
 ---> Using cache
 ---> ef45faf0280f
Successfully built ef45faf0280f
Successfully tagged kube-iostat:latest
 sh ./run.sh
ac1fb1779cd76a176bee79e1f12ba05d790f5280082e9b1cc03bb0de528a1361
$ curl http://localhost:5000/cputimes
{"user": 4980.22, "nice": 0.43, "system": 1725.59, "idle": 170968.54, "iowait": 117.3, "irq": 160.81, "softirq": 116.9, "steal": 0.0, "guest": 0.0, "guest_nice": 0.0}
$ curl http://localhost:5000/cpupercent
{"value": [3.0, 2.6, 5.5, 2.0, 2.9, 2.4, 3.8, 4.2, 4.0, 2.6, 3.7, 2.9]}
$ curl http://localhost:5000/cpucount
{"value": 12}
```

## Running in minikube

Clone the repository https://github.com/tarof429/kube-iostat.

```bash
$ git clone https://github.com/tarof429/kube-iostat
```

Then start minikube

```bash
$ minikube start
```

Then set the enviornment.

```bash
$ eval $(minikube -p minikube docker-env)
```

Build using the docker daemon in minikube:

```bash
$ sh ./buid.sh
...
```

Then deploy.

```bash
$ kubectl apply -f deployment.yml
service/kube-iostat-service created
deployment.apps/kube-iostat created
```

Now you SHOULD be able to access the service. However, I learned the hard way that using a LoadBalancer service with Minikube requires some extra setup. This is documented at https://minikube.sigs.k8s.io/docs/handbook/accessing/.

Normally you should just be able to start the tunnel. 

```bash
$ minikube tunnel
```

But in order to do this, you need to use visudo to allow your non-root user to run `ip` and `route`. 

Next, start the tunnel. You'll see output scrolling periodically. Mine looks like this:

```bash
Status:	
	machine: minikube
	pid: 214869
	route: 10.96.0.0/12 -> 172.17.0.3
	minikube: Running
	services: [kube-iostat-service]
    errors:
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors
```

Next run kubectl get svc. My output look like this.

```bash
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
kube-iostat-service   LoadBalancer   10.111.78.220   10.111.78.220   6000:30525/TCP   13m
kubernetes            ClusterIP      10.96.0.1       <none>          443/TCP          63m
```

Then you can access the service using the external IP.

```bash
$ curl http://10.111.78.220:6000/hello
Hello!
$ curl http://10.111.78.220:6000/cputimes
{"user": 7656.76, "nice": 0.48, "system": 2633.51, "idle": 240894.99, "iowait": 214.04, "irq": 259.1, "softirq": 200.67, "steal": 0.0, "guest": 0.0, "guest_nice": 0.0}
$ curl http://10.111.78.220:6000/cpupercent
{"value": [2.9, 0.7, 2.2, 1.4, 3.6, 9.3, 2.2, 0.7, 2.9, 0.7, 5.7, 8.7]}
$ curl http://10.111.78.220:6000/cpucount
{"value": 12}
```

As explained on this website, each service will get its own external ip.

It took a while for me to figure it out, but to see deployments and services with k9, just type `:` to enter a command prompt, then type `:deployments` or `:services` (`:svc` for short).

Try changing the number of replicas from 1 to 4 and applying the change. To do this, edit deployment.yml and run `kubectl apply -f .`. You'll see the new replicas show up automatically in k9s.

## References

https://github.com/giampaolo/psutil
https://kubernetes.io/blog/2019/07/23/get-started-with-kubernetes-using-python/


