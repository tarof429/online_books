# Part 3: Helloweb: A simple Go application

## Introduction

In this section, I will create a simple web server using Go and deploy it to Minikube. Writing web servers with Go is very simple and is easy to run whether on bare-metal or inside a container. We'll do both and hopefully have fewer stumbling blocks.

## Step 1: Create the web server

Below is the code for the web server which listens on port 9090. There are several places to find basic examples of web servers in Go. For example, the net/http package documentation at https://golang.org/pkg/net/http/. For this example, we don't need to do something as complicated as https://github.com/gorilla/mux.


{% code title="main.go" %}
```go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func myhandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from Go!\n")
}

func main() {
	http.HandleFunc("/", myhandler)

	err := http.ListenAndServe(":8090", nil)

	if err != nil {
		fmt.Printf("Unable to start serevr")
		os.Exit(1)
	}
}
```
{% endcode %}

To run, simply run:

```bash
$ go run main.go
```

## Step 2: Create the docker image

When creating docker images for Go, sometimes you think it's as simple as using a golang image as a builder and then copying the binary to an alpine-based image. However, Go programs have runtime dependencies on libc which alpine does not provide. 

The simplest solution is to therefore just use the golang image for both building and running Go programs.

{% code title="Dockerfile" %}
```dockerfile
FROM golang:1.14
WORKDIR '/src'
COPY main.go ./
RUN go build -o /usr/bin/simpleweb main.go && \
    rm -rf /src
EXPOSE 8090
CMD ["/usr/bin/simpleweb"]
```
{% endcode %}

To build and run the container, first make sure we are using the docker daemon inside minikube.

```bash
$ eval $(minikube -p minikube docker-env)
$ docker build -t simpleweb:v1.0 .
$ docker run --rm -p 8090:8090 -td simpleweb:v1.0
$ curl http://localhost:8090
Hello from Go!
```
## Step 3: Create a deployment

Next, we will create a deployment. The containerPort indicates that the container listens to port 8090; however, this section is not required as all ports will be mapped.

{% code title="simpleweb-deployment.yml" %}
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simpleweb-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: simpleweb-label
  template:
    metadata:
      labels:
        app: simpleweb-label
    spec:
      containers:
        - name: simpleweb
          image: simpleweb:v1.0
          ports:
          # Port used by this container
          - containerPort: 8090
```
{% endcode %}

## Step 3: Create the service

Next, we define the service. Note that we do not specify it as a ClusterIP type; this is already the default type. In this case, we set the port value equal to the targetPort value.

{% code title="simpleweb-clusterip-service.yml" %}
```yml
apiVersion: v1
kind: Service
metadata:
  name: simpleweb-clusterip-service
spec:
  selector:
    app: simpleweb-label
  ports:
    - port: 8090
      targetPort: 8090
```
{% endcode %}

## Step 4: Create the ingress

Finally we define the ingress. The port value should be set to the same port that the service listens to.

{% code title="ingress-service.yml" %}
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simpleweb-ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /?(.*)
            backend:
              serviceName: simpleweb-clusterip-service
              servicePort: 8090
```
{% endcode %}

## Step 5: Deploy the objects to the cluster

After making sure we are still connected to the docker daemon inside minikube, apply the objects.

```bash
$ eval $(minikube -p minikube docker-env)
$ kubectl apply -f .
$ curl http://172.17.0.2
Hello from Go!
```

## Additional Notes

There seems to be one important rule for exposing services: the servicePort in the ingress rule MUST match the targetPort for the service. We do not need to worry about the port value for our particular deployment as we don't have any other services that use our service. As an experiment, I was able to change the port value to a number like 4444 or 8091, and observed no change in the behavior of my deployment.