# Part 2: Hello Minikube

## Introduction

In this section, we will create a simple deployment based on the example at https://kubernetes.io/docs/tutorials/hello-minikube. But instead of following the steps straightaway, lets first try to understand what this example is.

## First attempt: deploying echoserver as a docker container

My first step was to pull k8s.gcr.io/echoserver:1.4 and inspect it. From my analysis, I cold see that there were two ports that were exposed: 80 and 443, so most likely this was a simple web server that could be accessed from these ports. 

```bash
        "ContainerConfig": {
            "Hostname": "5db5cb807d43",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "443/tcp": {},
                "80/tcp": {}
            },
```

I was able to confirm this by running `docker image history <image id>`.

```bash
$ docker image history a90209bb39e3
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a90209bb39e3        4 years ago         /bin/sh -c #(nop) ADD file:e81d532d985034e7c…   436B
<missing>           4 years ago         /bin/sh -c #(nop) ADD file:21c804191d758a6c0…   2.12kB
<missing>           4 years ago         /bin/sh -c #(nop) MAINTAINER Prashanth B <be…   0B
<missing>           4 years ago         /bin/sh -c #(nop) CMD ["nginx" "-g" "daemon …   0B
<missing>           4 years ago         /bin/sh -c #(nop) EXPOSE 443/tcp 80/tcp         0B
<missing>           4 years ago         /bin/sh -c ln -sf /dev/stderr /var/log/nginx…   11B
<missing>           4 years ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   11B
<missing>           4 years ago         /bin/sh -c /tmp/build.sh                        81MB
<missing>           4 years ago         /bin/sh -c #(nop) COPY file:b68eb82129e8e5a1…   6.39kB
<missing>           4 years ago         /bin/sh -c #(nop) CMD ["/bin/sh"]               0B
<missing>           4 years ago         /bin/sh -c #(nop) ADD file:7676e4706e4d499af…   59.4MB
```

So I ran the container and tried to curl it, but got nothing!

```bash
$ docker run --rm --name echoserver -p 80:80 -p 443:443 -d k8s.gcr.io/echoserver:1.4
$ curl http://localhost
curl: (56) Recv failure: Connection reset by peer
$ curl https://localhost
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to localhost:443
```

Confused, I looked at https://kubernetes.io/docs/tutorials/hello-minikube/ and took note of the fact that the instructions never say anything about exposing either port to the outside world. However, it did mention a LoadBalancer service for port 8080. After restarting the container and exposing this port, I was able to connect.

```bash
$ docker run --rm --name echoserver  -p 8080:8080 -d k8s.gcr.io/echoserver:1.4
0fb6e64939565a7be35b742c743ade0106515f4c2ac897a65a59cc5bbae05824
[taro@zaxman online_books (master)]$ curl http://localhost:8080
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://localhost:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=localhost:8080
user-agent=curl/7.72.0
BODY:
-no body in request-
```

I then SSH'ed into the container to see if I could find any more information about it.  In Visual Studio Code, I right-clicked on the container and attached a shell. Then I looked at /etc/nginx/nginx.conf, I found that the listener was hardcoded to listen to port 8080. I then felt that most likely the person who developed this image intended that the user maps an external port 80 to the internal port 8080 so stopped the container and started a new one with a small change. This finally gave me the result I was looking for.

```bash
$ docker run --rm --name echoserver -p 80:8080 -d k8s.gcr.io/echoserver:1.4
$ curl http://localhost
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://localhost:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=localhost
user-agent=curl/7.72.0
BODY:
-no body in request-
```

You can see that nginx only saw the request as http://localhost:8080. 

## Second attempt: creating a simple nginx image

Next I created a custom NGINX image. The base image is based upon nginx:latest and we'll overwrite index.html.

{% code title="Dockerfile" %}
```Dockerfile
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html
```
{% endcode %}

{% code title="index.html" %}
```
Hello nginx!
{% endcode %}

Now when I build and run the container we are able to see our custom HTML page.

```bash
$ docker build -t hellonginx .
$ docker run -td --rm -p 8090:80 hellonginx
$ curl http://localhost:8090
Hello nginx!
```

Going back to https://kubernetes.io/docs/tutorials/hello-minikube, we could deploy our image using an imperative command. Instead, we'll create a deployment yaml file and deploy it to minikube. After making sure minikube is running, we deploy the deployment. We base the file from https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#deployment-v1-apps.

{% code title="hellonginx-deployment.yml" %}
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: hellonginx-deployment
spec:
  # Create only one pod
  replicas: 1
  selector:
    matchLabels:
      app: hellonginx-label
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: hellonginx-label
    spec:
      containers:
      - name: hellonginx
        # Run this image
        image: hellonginx
```
{% endcode %}

```bash
$ kubectl apply -f hellonginx-deployment.yml
deployment.apps/hellonginx-deployment created
```

However, the deployment failed. Eventually we'll see that the issue is due to docker being unable to pull our custom image.

```bash
$ kubectl get deployments
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
hellonginx-deployment   0/1     1            0           4s
$ kubectl get pods
NAME                                    READY   STATUS              RESTARTS   AGE
hellonginx-deployment-d67d4f554-qxgnd   0/1     ContainerCreating   0          7s
$ kubectl get pods
NAME                                    READY   STATUS             RESTARTS   AGE
hellonginx-deployment-d67d4f554-qxgnd   0/1     ImagePullBackOff   0          51s
$ kubectl get pods
NAME                                    READY   STATUS         RESTARTS   AGE
hellonginx-deployment-d67d4f554-qxgnd   0/1     ErrImagePull   0          97s
```

This is because when want kubernetes to pull custom images, we need to interact with docker running inside minikube. This is explained at https://v1-16.docs.kubernetes.io/docs/setup/learning-environment/minikube/. 

After plenty of trial and error, I found that we need to do a bit of surgery to get things working.

Let's first delete our deployment so that we start fresh.

```bash
$ kubectl delete -f hellonginx-deployment.yml
```

Then set our environment.

```bash
$ eval $(minikube -p minikube docker-env)
```

Then to make sure we are really starting from scratch, lets SSH to minikube and delete our image.

```bash
$ docker rmi `docker images | grep hellonginx | awk -F" " '{print $3}'`
Untagged: hellonginx:v1.0
Deleted: sha256:643185f52bfaac69a7d57af4939c717e4e96b93953643cf80a54f2230438937d
Deleted: sha256:fd1b14273b733d2ad60056a601daba268e42003de95486951dee314142254c77
```

Then rebuild our image. 

```bash
$ docker build -t hellonginx:v1.0 .
```

Then make a quick edit to our deployment file to pick up this version.

{% code title="hellonginx-deployment.yml" %}
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: hellonginx-deployment
spec:
  # Create only one pod
  replicas: 1
  selector:
    matchLabels:
      app: hellonginx-label
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: hellonginx-label
    spec:
      containers:
      - name: hellonginx
        # Run this image
        image: hellonginx:v1.0
```
{% endcode %}

Then apply our deployment again.

```bash
$ kubectl apply -f hellonginx-deployment.yml
```

Finally our deployment works.

```bash
$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hellonginx-deployment-84f787f5b-bn4wn   1/1     Running   0          5s
$ kubectl get deployments
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
hellonginx-deployment   1/1     1            1           117s
```

## How to access hellonginx

We'll first install the [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) addon for Minikube. The steps to do this are described at https://kubernetes.github.io/ingress-nginx/deploy/. 

```bash
$ minikube addons enable ingress
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

Next we'll create a ClusterIP service to expose our deployment to the nginx ingress controller. What the ClusterIP service does is create an IP address .

{% code title="hellonginx-clusterip-service.yml" %}
```yml
apiVersion: v1
kind: Service
metadata:
  name: hellonginx-clusterip-service
spec:
  type: ClusterIP
  selector:
    app: hellonginx-label
  ports:
    - port: 80
      targetPort: 80
```
{% endcode %}


Finally we'll create our ingress service.

{% code title="ingress-service.yml" %}
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
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
              serviceName: hellonginx-clusterip-service
              servicePort: 80
```
{% endcode %}

```bash
$ kubectl apply -f hellonginx-clusterip-service.yml
$ kubectl apply -f ingress-service.yml
```

We'll get the IP of minikube to connect to our service.

```bash
$ minikube ip
172.17.0.3
$ curl http://172.17.0.3
Hello nginx!
```

## Additional Notes

After getting this far, I tried to figure out a way to change the port so that the web application would be reachable from, say, http://172.17.0.3:90. However, I discovered that this is not something that is possible with the ingress service. It does not appear to be a feature or a concern of the ingress service. I was able to change the route for the hellonginx-clusterip-service by making a small change to our ingress service file by replacing `path: /?(.*)` with  `path: /hello/?(.*)` and redeploying our service. Subsequently, I was able to access the service using `http://172.17.0.3/hello`. 

One thing that we can do is change the listener port for nginx. To do this, we need to first build a new image with a customized nginx.conf file.

{% code title="nginx.conf" %}
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

		server {
			listen 9090;
		}
}
```
{% endcode %}

Then rebuild the image.

```bash
$ docker build -t hellonginx:v1.1 .
```

Next, we modify the cluster IP service to forward requests to port 9090.

{% code title="hellonginx-clusterip-service.yml" %}
```yml
apiVersion: v1
kind: Service
metadata:
  name: hellonginx-clusterip-service
spec:
  type: ClusterIP
  selector:
    app: hellonginx-label
  ports:
    - port: 9090
      targetPort: 80
```
{% endcode %}

Finally we cahnge our route.

{% code title="ingress-service.yml" %}
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /helllo?(.*)
            backend:
              serviceName: hellonginx-clusterip-service
              servicePort: 9090
```
{% endcode %}

Our URL to our service is unchanged.

```bash
$ curl http://172.17.0.3/hello
Hello nginx!
```

## Conclusion

At this point, I still didn't feel comfortable with my deployment. The main reason is that I'm not comfortable enough with nginx configuration. In the next section, I will develop a simple web application using Go.
