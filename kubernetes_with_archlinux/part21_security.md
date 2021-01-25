# Security

All user access to the cluster is managed by the api-server. All actions done on the cluster, whether from kubectl or curl, is first authenticated by the api-server. There are multiple ways the api-server can authenticate users. For example, using simple files, files containing tokens, certificates, and also third-party identity services. 

## Basic Authentication

To authenticate users using basic authentication, you must create a file with three columns and pass it as an argument to the api-server. The three columns are: username, password, and user ID. 

When a user needs to authenticate to the api-server using curl aznd basic authentication, pass the username and password using the `-u` option. 

Another option is to use a static token file. When a user needs to authenticate to the api-server, pass the token in the header of the request. 

Basic authentication is not recommended. One possible reason is that the username and password are passed to the server in clear text. 

## Certificate-based Authentication

### Generating certificates

In Kubernetes, components are secured using TLS. The API server is a prime example of a component that has server-side certificates. The API server, or kube-API server, has a private key and a public identity in the form of a certificate. The etcd server is another server-side component that has a private key and certificate. But worker nodes, otherwise known as the kubelet, also function as a server-side component, and like the API server and etcd, has its own private key and certificate. 

If the API server, etcd, and kubelet comprise the server-side components, what are their corresponding client-side components? There is, for example, kubectl that administrator use to interact with the cluster. The scheduler is another client that talks to the API server and gets information about what pods need to be scheduled on nodes. Lest we forget, other components are the kube controller manager and kube proxy that run on the nodes. These, too, are clients to the API server. Even the API server is a client to etcd; in fact, it is the only client that talks directly to etcd. 

To secure a cluster, lets say we need to first create a CA, or certificate authority. Here use openssl to generate a self-signed certificate.

```
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES=CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

To view the contents of the certificate:

```
openssl x509 -in ca.crt -text -noout
```

What about the certificate for ourselves? The process is very similar with two exceptions. The first is that when generating our certiicate, we must specify our organization as `system:masters` to make the API server accept ourselves as an admin users. Also, when generating our certicate, we must sign the certificate using the CA certificate and key.

```
openssl genrsa -out myself.key 2048
openssl req -new -key myself.key -subj "/CN=kube-admin/O=system:masters" -out myself.csr
openssl x509 -req -in myself.csr -CA ca.crt -CAkey ca.key -out myself.crt
```

When generating certificates for the kube-scheduler, kube-controller manager, and kube-proxy, we must prefix the CN with `system:` because, while these are clients to the API server, they are considered system components. That is, any component of kubernetes must have a CN that begins with `system:`. 

etcd is a server-side component which requires peer certificates becuase it can be deployed in a cluster of its own. The etcd server's configuration file `etcd.yaml` has command-line arguments that point to its certificates and pper certificates. 

The kube-API server is special because everyone talks to the API server. The API server has multiple DNS names and IP addresses; all of these must be present in the certificate. The process to create the certificate is similar to what we have seen before. But how do we create a certificate with all the DNS names and IP addresses? To do this, we can put these attributes in a file and pass it to the openssl command.

```
openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -subj "/CN=KUBERNETES=CA" -out apiserver.csr -config openssl.cnf
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt
```

### Viewing certificates

Let's say there are issues in the cluster involving certificates. How do you view the certificates in the cluster? 

To solve this issue, we need to find out how the various components are started. For example, they may be started as system services. Or they may be started as static pods and we need to look for their definition files under /etc/kubernetes/manifests. Once we locate the certiifcates, we can use `openssl x509 -in taro.crt -text -noout` to view the contents. 

### Viewing CSR

To view a CSR, use the `req` command. The `req` is used for all CSRs. 

```
openssl req -in taro.csr -text -noout
```

### Checking logs

Another place to check for issues is the logs. Use `journalctl -u etcd.service -l` to view the logs for etcd. If kubeadm was used and the components are running as docker containers, you can use `kubectl logs <docker container>` to view those logs. But if the API server itself is down, you may have to use docker directly; for example, `docker logs <docker container>`. 

### Using the certificates API

Kubernetes provides an API to help generate certificates for administrators of the cluster. This is useful when there are multiple administrators of the cluster, and you want an automated way to do this.

The user must first have a private key. If they don't have one, he must run the following command.

```
openssl genrsa -out taro.key 2048
```

Then the user must create a CSR based on their private key.

```
openssl req -new taro.key -subj "CN=taro" -out taro.csr
```

The administrator takes this CSR and creates a certificate signing object. It should look something like this:

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  groups:


  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

Two things to note:

- The certificatte is base 64 encoded. Use: `cat john.csr | base64 | tr -d "\n"`,
- According to the official k8s doc, the usage must be `client auth`. However, the Udemy course says it must be `server auth`. 

Certificates can be viewed by running `kubectl get csr`. A CSR can be approved by running `kubectl certificate approve john`. 

View certificates by running `kubectl get csr john -o yaml`. The certiifcate will be in base64 format. You must convert it to text format by running `echo "certicate contents | base 64- --decode` 

Certificate signing is managed by the controller manager. To see details, see /etc/kubernetes/manifests/kube-controller-manager.yaml.


### kubeconfig

The ~/kube/config file is used to set the user and creditials when accessing a cluster. When you start minikube, this file is populated with the the user and cluster, and ties the two using a `context`. The `kubectl config` command is used to view, set, and delete cluster configurations specified in this file. If you want to set a default namespace when you set a context, that is also possible. 

Normally, certificate information is provided as a path in the kubeconfig file. However, you can also include the content in the kubeconfig file, as long as it is in base64 format. See https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/. 