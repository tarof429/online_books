# Part 8: Using MySQL as a service

In the previous sections, the deployments did not depend on any existing service such as a database. All we had to do was create some kind of application that could be accessible to the outside world and could respond to requests. 

In this section, we will create a service that depends on MySQL. We will create deployments for both, but I still find kubernetes a little intimidating so we'll start with using docker-compose. Even with just using docker-compose, I needed to spend significant time to create a client that could connect to MySQ, to populate the database with data, and overall to use a database as a backend. 

## Initial version with docker-compose

The initial version at https://github.com/tarof429/tron_legacy_cast/tree/v1.0 uses only docker-compose to create the services. When the compose file comes up, two services will be created. The first is based on a MySQL variant called mariadb. I've set some environment variables to set the initial user and database. The schema is created by volume mounting the directory db/dump to /docker-entrypoint-initdb.d. We'll find out how to do this for kubernetes. The second service is a REST client using Python. We pass in the same environment variables it needs to connect to mariadb. However, this was not part of the initial design. Initially, I was very eager to try to get the client working and so all these values were hard-coded. Although getting the REST client was not difficult to do in the beginning, I had lots of challenges in performing queries against mariadb, and figuring out how to return those results. I ended up creating a second client called `standalone_client.py` that I could use to develop my code. I found that this was a very productive approach and one that I probably should have tried earlier to save a lot of effort. I also created a library called api_client_lib.py which could be shared between the two clients. I also felt like I wish I had taken this approach much earlier to save a lot of duplicate code between the two clients. What was particularly difficult was figuring out how to write queries against mariadb. This is partially dicated by the library used. I ended up using PyMySQL. This is an open-source driver and I chose it because I wanted to return the results in JSON format. Other database drivers did not do this and I would have to manually create the JSON from the result set. 

In summary:

- Environment variables and volume mounts were used to initialize the database with a user, databaase, and initial schema.
- A library was created that could be shared between the two clients
- One client was a standalone and did not use REST. This allowed me to concentrate on testing the code and avoid any issues that using REST might introduce.
- Another client used REST. It took significant time to figure out what driver to use and how to invoke methods correctly. For example, to fetch a cast member by name or movie name, since most of these included spaces, the query had to be enclosed in double quotes and spaces had to be replaced with %20. For example, ` curl http://localhost:5000/name/Jarvis` was easy to run, but it really took time to figure out `curl "http://localhost:5000/name/Beau%20Garrett"`. 

All the testing was done manually; it would be nice in the future if tests could be added to our code base.

## Second version using Kubernetes

I first created my deployment for mariadb, with a service called `mariadb-deployment`. Based on previous experience where I created a deployment for Postgres, this deployment specifies a persistent volume claim, the equivalent of a docker volume in Kubernetes. I then specified my volume mount, referring to my PVC. This is followed by a list of environment variables. 

We use a LoadBalancer service to expose our REST API to the world. This means that we do need to start `minikube tunnel` in order to be able access the service.

Below are the steps for deploying this application:

```bash
$ minikube start
$ minikube tunnel (let this run in a separate terminal)
$ eval $(minikube -p minikube docker-env)
$ cd api-client (will be built using the docker daemoon inside minikube)
$ sh ./build.sh
kubectl apply -f k8s
$ 
```

In theory, this should have worked, but the apiclient-deployment failed and I couldn't figure out why. I tried rebuilding the docker image and changing the number of replicas from 3 to 1, but for some reason it was still failing. Then I tried to look into any logs from the container. That's when I discovered a clue:

```bash
[taro@zaxman kubernetes]$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
apiclient-deployment-947998575-s6kzd   0/1     CrashLoopBackOff   6          6m3s
mariadb-deployment-84499bcd78-bnzld    1/1     Running            0          6m3s
[taro@zaxman kubernetes]$ kubectl logs apiclient-deployment-947998575-s6kzd
Traceback (most recent call last):
  File "client.py", line 10, in <module>
    dbHost = os.environ['MYSQL_HOST']
  File "/usr/local/lib/python3.8/os.py", line 675, in __getitem__
    raise KeyError(key) from None
KeyError: 'MYSQL_HOST'
```

This was it! A quick loook at my client deployment, and I realized that the deployment didn't have any of the required environment variables set. I did a quick copy-and-paste, making sure I ddin't copy the MYSQL_ROOT_PASSWORD variable, and redployed everything. Here I forgot again that I need to go through the load balancer instead of minikube ip. So instead of `http://172.17.0.2:6000/hello` it was `http://10.111.78.207:6000/hello`. Now I got a new error.

```bash
$ curl http://10.111.78.207:6000/all
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>

$ kubectl logs apiclient-deployment-69cfd89f84-cbg58
...
AttributeError: 'ApiClient' object has no attribute 'conn'
172.17.0.1 - - [26/Sep/2020 22:56:30] "GET /all HTTP/1.1" 500 -
INFO:werkzeug:172.17.0.1 - - [26/Sep/2020 22:56:30] "GET /all HTTP/1.1" 500 -
ERROR:client:Exception on /all [GET]
Traceback (most recent call last):
  File "/src/venv/lib/python3.8/site-packages/pymysql/connections.py", line 569, in connect
    sock = socket.create_connection(
  File "/usr/local/lib/python3.8/socket.py", line 787, in create_connection
    for res in getaddrinfo(host, port, 0, SOCK_STREAM):
  File "/usr/local/lib/python3.8/socket.py", line 918, in getaddrinfo
    for res in _socket.getaddrinfo(host, port, family, type, proto, flags):
socket.gaierror: [Errno -2] Name or service not known

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "client.py", line 32, in get_all
    client.connect()
  File "/src/api_client_lib.py", line 16, in connect
    self.conn = pymysql.connect(
  File "/src/venv/lib/python3.8/site-packages/pymysql/__init__.py", line 94, in Connect
    return Connection(*args, **kwargs)
  File "/src/venv/lib/python3.8/site-packages/pymysql/connections.py", line 327, in __init__
    self.connect()
  File "/src/venv/lib/python3.8/site-packages/pymysql/connections.py", line 619, in connect
    raise exc
pymysql.err.OperationalError: (2003, "Can't connect to MySQL server on 'db' ([Errno -2] Name or service not known)")

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/src/venv/lib/python3.8/site-packages/flask/app.py", line 2447, in wsgi_app
    response = self.full_dispatch_request()
  File "/src/venv/lib/python3.8/site-packages/flask/app.py", line 1952, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/src/venv/lib/python3.8/site-packages/flask/app.py", line 1821, in handle_user_exception
    reraise(exc_type, exc_value, tb)
  File "/src/venv/lib/python3.8/site-packages/flask/_compat.py", line 39, in reraise
    raise value
  File "/src/venv/lib/python3.8/site-packages/flask/app.py", line 1950, in full_dispatch_request
    rv = self.dispatch_request()
  File "/src/venv/lib/python3.8/site-packages/flask/app.py", line 1936, in dispatch_request
    return self.view_functions[rule.endpoint](**req.view_args)
  File "client.py", line 35, in get_all
    client.close_connection()
  File "/src/api_client_lib.py", line 42, in close_connection
    self.conn.close()
AttributeError: 'ApiClient' object has no attribute 'conn'
172.17.0.1 - - [26/Sep/2020 22:58:13] "GET /all HTTP/1.1" 500 -
INFO:werkzeug:172.17.0.1 - - [26/Sep/2020 22:58:13] "GET /all HTTP/1.1" 500 -
```

I was able to duplicate this error by SSH'ing into my container and running the standalone client.

```bash
root@apiclient-deployment-69cfd89f84-cbg58:/src# python ./standalone_client.py
standalone_client starting...
Traceback (most recent call last):
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 569, in connect
    sock = socket.create_connection(
  File "/usr/local/lib/python3.8/socket.py", line 787, in create_connection
    for res in getaddrinfo(host, port, 0, SOCK_STREAM):
  File "/usr/local/lib/python3.8/socket.py", line 918, in getaddrinfo
    for res in _socket.getaddrinfo(host, port, family, type, proto, flags):
socket.gaierror: [Errno -2] Name or service not known

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "./standalone_client.py", line 26, in <module>
    client.connect()
  File "/src/api_client_lib.py", line 16, in connect
    self.conn = pymysql.connect(
  File "/usr/local/lib/python3.8/site-packages/pymysql/__init__.py", line 94, in Connect
    return Connection(*args, **kwargs)
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 327, in __init__
    self.connect()
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 619, in connect
    raise exc
pymysql.err.OperationalError: (2003, "Can't connect to MySQL server on 'db' ([Errno -2] Name or service not known)")

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "./standalone_client.py", line 44, in <module>
    client.close_connection()
  File "/src/api_client_lib.py", line 42, in close_connection
    self.conn.close()
AttributeError: 'ApiClient' object has no attribute 'conn'
```

I realized that my first mistake was setting MYSQL_HOST to db instead of mariadb-cluster-ip-service. However setting this environment variable in the container still resulted in an error:

```bash
root@apiclient-deployment-69cfd89f84-cbg58:/src# python ./standalone_client.py
standalone_client starting...
*************************
Traceback (most recent call last):
  File "./standalone_client.py", line 30, in <module>
    result = client.fetchAll()
  File "/src/api_client_lib.py", line 22, in fetchAll
    cursor.execute(sql)
  File "/usr/local/lib/python3.8/site-packages/pymysql/cursors.py", line 163, in execute
    result = self._query(query)
  File "/usr/local/lib/python3.8/site-packages/pymysql/cursors.py", line 321, in _query
    conn.query(q)
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 505, in query
    self._affected_rows = self._read_query_result(unbuffered=unbuffered)
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 724, in _read_query_result
    result.read()
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 1069, in read
    first_packet = self.connection._read_packet()
  File "/usr/local/lib/python3.8/site-packages/pymysql/connections.py", line 676, in _read_packet
    packet.raise_for_error()
  File "/usr/local/lib/python3.8/site-packages/pymysql/protocol.py", line 223, in raise_for_error
    err.raise_mysql_exception(self._data)
  File "/usr/local/lib/python3.8/site-packages/pymysql/err.py", line 107, in raise_mysql_exception
    raise errorclass(errno, errval)
pymysql.err.ProgrammingError: (1146, "Table 'movie.cast' doesn't exist")
```

This shows that the schema wasn't created. At this point I'm stuck. I don't know how to do a volume mount like in docker-compose. That's when I searched in Google and found a solution using ConfigMaps.

## Using ConfigMaps

According to https://stackoverflow.com/questions/45681780/how-to-initialize-mysql-container-when-created-on-kubernetes, we can create a ConfigMap object and moun the volume in our pod. The ConfigMap would look something like this:

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-initdb-config
data:
  initdb.sql: |
    CREATE TABLE friends (id INT, name VARCHAR(256), age INT, gender VARCHAR(3));
    INSERT INTO friends VALUES (1, 'John Smith', 32, 'm');
    INSERT INTO friends VALUES (2, 'Lilian Worksmith', 29, 'f');
    INSERT INTO friends VALUES (3, 'Michael Rupert', 27, 'm');
```

In our case, it should look something like this:

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-initdb-config
data:
  initdb.sql: |
    CREATE TABLE IF NOT EXISTS cast (
        id INT NOT NULL AUTO_INCREMENT,
        name VARCHAR(100) NOT NULL,
        movie_name VARCHAR(100) NOT NULL,
        PRIMARY KEY ( id ),
        UNIQUE KEY (movie_name)
    );

    INSERT INTO cast (name, movie_name) VALUES 
      ('Garrett Hedlund', 'Samuel "Sam" Flynn'), 
      ('Jeff Bridges', 'Kevin Flynn'),
      ('Olivia Wilde', 'Quorra'),
      ('Bruce Boxleitner', 'Alan Bradley'),
      ('Bruce Boxleitner', 'Rinzler'),
      ('Michael Sheen', 'Zuse'),
      ('Michael Sheen', 'Castor'),
      ('James Frain', 'Jarvis'),
      ('Beau Garrett', 'Gem'),
      ('Daft Punk', 'Daft Punk');
```

I was going to investigate ConfigMaps later as a way to set environment variables, but without a schema I had nothing to work with.

But even after creating the ConfigMap I wasn't getting any data back.

There was something very tricky about connecting to databases running in pods and I found this out the hard way. If you SSH to the container running MySQL, you cannot connect to it with MYSQL_HOST set to the service name. My first solution was to unset MYSQL_HOST. 

After deleting my deployments and trying again, it finally worked!

```bash
$ curl http://10.106.39.215:6000/all
"[{\"name\": \"Garrett Hedlund\", \"movie_name\": \"Samuel \\\"Sam\\\" Flynn\"}, {\"name\": \"Jeff Bridges\", \"movie_name\": \"Kevin Flynn\"}, {\"name\": \"Olivia Wilde\", \"movie_name\": \"Quorra\"}, {\"name\": \"Bruce Boxleitner\", \"movie_name\": \"Alan Bradley\"}, {\"name\": \"Bruce Boxleitner\", \"movie_name\": \"Rinzler\"}, {\"name\": \"Michael Sheen\", \"movie_name\": \"Zuse\"}, {\"name\": \"Michael Sheen\", \"movie_name\": \"Castor\"}, {\"name\": \"James Frain\", \"movie_name\": \"Jarvis\"}, {\"name\": \"Beau Garrett\", \"movie_name\": \"Gem\"}, {\"name\": \"Daft Punk\", \"movie_name\": \"Daft Punk\"}]"
$ curl "http://10.106.39.215:6000/name/Michael%20Sheen"
"[{\"name\": \"Michael Sheen\", \"movie_name\": \"Zuse\"}, {\"name\": \"Michael Sheen\", \"movie_name\": \"Castor\"}]"
```

Another solution was to set MYSQL_HOST to localhost. This also worked. But what if I really wanted to connect to the database using a hostname? What then?

Then I found https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#use-pod-fields-as-values-for-environment-variables which explained how to use pod fields as environment variables. 

Among the three choices, I settled on setting MYSQL_HOST to localhost because it's the easiest to understand.

## ConfigMaps for Setting MYSQL properties

https://github.com/tarof429/tron_legacy_cast/tree/v1.1 is a minor but very significant update. It uses an additional ConfigMap to centralize properties used by both Mariadb and the API client. I had significant challenges in getting this working and a significant issue was figuring out why my misconfigured pod wouldn't start. If a pod or deployment is misconfigured for any reason, `kubectl` will not report it. If you run `kubectl get pods` you will see a CreateContainerConfigError. The way I tried to investigate this was to come up with another deployment which I called `envclient-deployment` which I used to print environment variables in an attempt to get more information. Later I found a much better approach. This is to run the command `kubectl describe pod <pod name>`. Excerpts of this output are shown below.

```bash
$ kubectl describe pod mariadb-deployment-f4867bb77-qqs9b
Name:         mariadb-deployment-f4867bb77-qqs9b
...
    Environment:
      MYSQL_HOST:           localhost
      MYSQL_ROOT_PASSWORD:  portal
      MYSQL_USER:           <set to the key 'mysqluser' of config map 'mariadb-configmapx'>     Optional: false
      MYSQL_PASSWORD:       <set to the key 'mysqlpassword' of config map 'mariadb-configmap'>  Optional: false
      MYSQL_DATABASE:       <set to the key 'mysqldatabase' of config map 'mariadb-configmap'>  Optional: false
    Mounts:
...
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  3m19s                 default-scheduler  Successfully assigned default/mariadb-deployment-f4867bb77-qqs9b to minikube
  Warning  Failed     78s (x12 over 3m18s)  kubelet            Error: configmap "mariadb-configmapx" not found
  Normal   Pulled     63s (x13 over 3m18s)  kubelet            Container image "mariadb:10.4" already present on machine
...
```

At a quick glance, all the properties looked like they are being set to some value. However there was an event that said configmap "mariadb-configmapx" not found. And if i scrolled up, it seems odd that MYSQL_USER tries to use config map 'mariadb-configmapx'. This is not exactly what happened, but it illustrates how `kubectl describe` can be used to debug Kubernetes issues. What actually happend was that I tried to use the volume name instead of the configmap name for each property. Why trying to figure out how to use ConfigMaps, I referred to https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/. Perhaps there is a better resource out there that explains ConfigMaps?

## Using Secrets

After getting this far, I really wanted to take a break. But I wasn't done yet! We need to use Secrets to encrypt our database passwords. This avoids having to hardcode these values in our deployment files. I've created a script that asks the user for the root and user password and based on these inputs the Secrets file is generated. Then all I have to do is run `kubetl deploy -f k8s` to deploy the Secret. 

This version has been released at https://github.com/tarof429/tron_legacy_cast/tree/v1.2.