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

Next we'll create a custom NGINX image. The base image is based upon nginx:latest and we'll overwrite index.html.

{% code title="Dockerfile" %}
```
FROM nginx:latest
COPY ./index.html /usr/share/nginx/html/index.html
```
{% endcode %}

{% code title="index.html" %}
```
Hello nginx!
{% endcode %}

Now when we build and run the container we are able to see our custom HTML page.

```bash
$ docker build -t hellonginx .
$ docker run -td --rm -p 8090:80 hellonginx
$ curl http://localhost:8090
```