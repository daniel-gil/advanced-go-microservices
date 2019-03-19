# Go Microservices Frameworks

## Running Gin Web Framework

For this example, we will need the Gin web framework:

``` bash
go get github.com/gin-gonic/gin
```

We can run the gin webserver with 

``` bash
$ cd ./Frameworks/Gin-Web/
$ go run *.go
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /ping                     --> main.main.func1 (3 handlers)
[GIN-debug] GET    /hello                    --> main.main.func2 (3 handlers)
[GIN-debug] GET    /api/books                --> main.main.func3 (3 handlers)
[GIN-debug] POST   /api/books                --> main.main.func4 (3 handlers)
[GIN-debug] GET    /api/books/:isbn          --> main.main.func5 (3 handlers)
[GIN-debug] PUT    /api/books/:isbn          --> main.main.func6 (3 handlers)
[GIN-debug] DELETE /api/books/:isbn          --> main.main.func7 (3 handlers)
[GIN-debug] Loaded HTML Templates (2): 
        - 
        - index.html

[GIN-debug] GET    /favicon.ico              --> github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (3 handlers)
[GIN-debug] HEAD   /favicon.ico              --> github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (3 handlers)
[GIN-debug] GET    /                         --> main.main.func8 (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080

```

The API calls processed are displayed in the console:
```
[GIN] 2019/03/18 - 15:14:02 | 200 |      52.607µs |             ::1 | GET      /ping
[GIN] 2019/03/18 - 15:14:53 | 200 |     228.558µs |             ::1 | GET      /hello
[GIN] 2019/03/18 - 15:15:50 | 200 |     186.498µs |             ::1 | GET      /api/books
[GIN] 2019/03/18 - 15:16:07 | 200 |      11.964µs |             ::1 | GET      /api/books/4432432432
[GIN] 2019/03/18 - 15:16:40 | 201 |     383.826µs |             ::1 | POST     /api/books
[GIN] 2019/03/18 - 15:19:39 | 200 |      44.836µs |             ::1 | PUT      /api/books/444444444
[GIN] 2019/03/18 - 15:20:15 | 200 |       2.623µs |             ::1 | DELETE   /api/books/444444444
````


## Service API

### Ping
Request:
```
GET http://localhost:8080/ping
```

Response:
```
Status: 200 OK
Body: pong
```

### Hello

Request:

```
GET http://localhost:8080/hello
```

Response:
```
Status: 200 OK
Body:
{
    "message": "Hello Gin Framework."
}
```

### Get all books
Request:

```GET http://localhost:8080/api/books``` 

Response:

```
Status: 200 OK
Body:
[
    {
        "title": "The Hitchhiker's Guide to the Galaxy",
        "author": "Douglas Adams",
        "isbn": "88854858345",
        "description": "a funny book"
    },
    {
        "title": "Advanced Cloud Native Go",
        "author": "M. Leander Reimer",
        "isbn": "0000000000"
    }
]
```

### Get book by ISBN

Request:
```
GET http://localhost:8080/api/books/{ISBN}
```

Response:
```
Status: 200 OK
Body:
{
    "title": "The Hitchhiker's Guide to the Galaxy",
    "author": "Douglas Adams",
    "isbn": "88854858345",
    "description": "a funny book"
}
```

### Create a book

Request:
```
POST http://localhost:8080/api/books
Body:
{
    "title": "Game of thrones",
    "author": "George R.R. Martin",
    "isbn": "444444444",
    "description": "an incredible story"
}

```

Response:
```
Status: 201 Created
```


### Update a book

Request:
```
PUT http://localhost:8080/api/books/{ISBN}
Body:
{
    "title": "Game of thrones",
    "author": "George R.R. Martin",
    "isbn": "444444444",
    "description": "an incredible story with a lot of action"
}
```

Response:
```
Status: 200 OK
```

### Delete a book

Request:
```
DELETE http://localhost:8080/api/books/{ISBN}
```

Response:
```
Status: 200 OK
```

## Docker
Docker must be installed, you can check that docker is running with:

```bash
docker version
```

### Dockerfile
The Dockerfile is essentially the build instructions to build the image.

``` 
# golang docker base image
FROM golang:1.12.0-alpine

# install the gin web framework (and install required packages for executing `go get`)
RUN apk update && apk upgrade && apk add --no-cache bash git
RUN go get github.com/gin-gonic/gin

# copy local sources into this docker image (defining an environment variable SOURCES)
ENV SOURCES /go/src/github.com/daniel-gil/advanced-go-microservices/Frameworks/Gin-Web/
COPY . ${SOURCES}

# change into the SOURCES directory and call `go build`
RUN cd ${SOURCES} && CGO_ENABLED=0 go build

# define the work directory
WORKDIR ${SOURCES}

# define the entry-poing command (it is, the Gin-Web executable after building the binary)
CMD ${SOURCES}Gin-Web

# expose port 8080
EXPOSE 8080
```

### docker-componse.yml
docker-componse.yml is a config file for docker-compose. It allows to deploy, combine and configure multiple docker-container at the same time.

```
version: '3'

services:
  microservice:
    build: .
    image: gin-web:1.0.1
    environment:
    - PORT=8080
    ports:
    - "8080:8080"
```

### Building docker image using docker-compose
To list the docker images available:
```
docker images
```

To build a docker image:
```
$ docker-compose build
Building microservice
Step 1/9 : FROM golang:1.12.0-alpine
 ---> 2205a315f9c7
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Running in 571eec34615b
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
v3.9.2-21-g3dda2a36ce [http://dl-cdn.alpinelinux.org/alpine/v3.9/main]
v3.9.2-23-gb6bc53c8f8 [http://dl-cdn.alpinelinux.org/alpine/v3.9/community]
OK: 9756 distinct packages available
(1/2) Upgrading libcrypto1.1 (1.1.1a-r1 -> 1.1.1b-r1)
(2/2) Upgrading libssl1.1 (1.1.1a-r1 -> 1.1.1b-r1)
Executing ca-certificates-20190108-r0.trigger
OK: 6 MiB in 15 packages
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/11) Installing ncurses-terminfo-base (6.1_p20190105-r0)
(2/11) Installing ncurses-terminfo (6.1_p20190105-r0)
(3/11) Installing ncurses-libs (6.1_p20190105-r0)
(4/11) Installing readline (7.0.003-r1)
(5/11) Installing bash (4.4.19-r1)
Executing bash-4.4.19-r1.post-install
(6/11) Installing nghttp2-libs (1.35.1-r0)
(7/11) Installing libssh2 (1.8.0-r4)
(8/11) Installing libcurl (7.64.0-r1)
(9/11) Installing expat (2.2.6-r0)
(10/11) Installing pcre2 (10.32-r1)
(11/11) Installing git (2.20.1-r0)
Executing busybox-1.29.3-r10.trigger
OK: 29 MiB in 26 packages
Removing intermediate container 571eec34615b
 ---> a1d03237249c
Step 3/9 : RUN go get github.com/gin-gonic/gin
 ---> Running in f45a24075d94
Removing intermediate container f45a24075d94
 ---> ab97154e9078
Step 4/9 : ENV SOURCES /go/src/github.com/daniel-gil/advanced-go-microservices/Frameworks/Gin-Web/
 ---> Running in 271f693279e9
Removing intermediate container 271f693279e9
 ---> 76e0deae300b
Step 5/9 : COPY . ${SOURCES}
 ---> 85527054db89
Step 6/9 : RUN cd ${SOURCES} && CGO_ENABLED=0 go build
 ---> Running in 0766b6826bfc
Removing intermediate container 0766b6826bfc
 ---> 02391963ea65
Step 7/9 : WORKDIR ${SOURCES}
 ---> Running in 1e1d2249ec4b
Removing intermediate container 1e1d2249ec4b
 ---> f3eea799c147
Step 8/9 : CMD ${SOURCES}Gin-Web
 ---> Running in a91583983d32
Removing intermediate container a91583983d32
 ---> 75b70c3fcb7a
Step 9/9 : EXPOSE 8080
 ---> Running in c1a4034f9919
Removing intermediate container c1a4034f9919
 ---> c7e5eedbebcc
Successfully built c7e5eedbebcc
Successfully tagged gin-web:1.0.1
```

Now we can check that the new docker image has been created:
```bash
$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED              SIZE
gin-web                1.0.1               c7e5eedbebcc        About a minute ago   462MB
``` 

To run this docker image in background:
```
$ docker-compose up -d
Recreating gin-web_microservice_1 ... done
```

List all docker container running in the background:
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
c74b35148445        gin-web:1.0.1       "/bin/sh -c ${SOURCE…"   16 seconds ago      Up 15 seconds       0.0.0.0:8080->8080/tcp   gin-web_microservice_1
```

To stop everything we can:
```
$ docker-compose stop
Stopping gin-web_microservice_1 ... done
```

### Tagging and pushing to remote registry

#### Tag
We can tag a docker image with (using the docker hub username):
```
docker tag gin-web:1.0.1 danigilmayol/gin-web:1.0.1
```

Once tagged we can list again the docker images to verify that the new tagged is listed:
```
$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
danigilmayol/gin-web   1.0.1               c7e5eedbebcc        14 minutes ago      462MB
gin-web                1.0.1               c7e5eedbebcc        14 minutes ago      462MB
```

#### Push
Now we can use the following command for pushing the tagged image to the docker hub repository (using the docker.io username):
```
$ docker push danigilmayol/gin-web:1.0.1
The push refers to repository [docker.io/danigilmayol/gin-web]
2aa85349ad3e: Pushed 
9b0c5f6a6b31: Pushed 
4d4949d3b5fe: Pushed 
0a8c8ab90b25: Pushed 
a124fdb5e3b1: Layer already exists 
dbf140e0475c: Layer already exists 
41a6c6e6c287: Layer already exists 
4afd4ab746df: Layer already exists 
bcf2f368fe23: Layer already exists 
1.0.1: digest: sha256:bbb37c779a6eb997284db1af6d8c9993cb32709bda581b403c7a79065c4240a3 size: 2209
```

Now the docker image is available from docker hub.


## Kubernetes
Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. 

### Concepts

#### Pod
The `pod` is the smallest deployble unit computation within Kubernetes. The `pod` contains our containers and it can be described using labels.

#### Deployment
The `deployment` allows for the clarity of updates the pods. 

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gin-web
  labels: 
    app: gin-web
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: gin-web
        tier: service
    spec:
      containers:
      - name: gin-web
        image: "gin-web:1.0.1"
        ports: 
        - containerPort: 9090
        env:
        - name: PORT
          value: "9090"

        # define resource requests and limits
        resources: 
          requests:
            memory: "64Mi"
            cpu: "125m"
          limits:
            memory: "128Mi"
            cpu: "250m"
        # check of gin-web is alive and healthy
        readinessProbe:
          httpGet:
            path: /ping
            port: 9090
          initialDelaySeconds: 5
          timeoutSeconds: 5
        livenessProbe:
          httpGet:
            path: /ping
            port: 9090
          initialDelaySeconds: 5
          timeoutSeconds: 5
```

#### Service 
The `service` is an abstraction for a logical collection of pods. It is declassible within the Kubernetes cluster using a DNS name.

```
apiVersion: v1
kind: Service
metadata: 
  name: gin-web
  labels:
    app: gin-web
    tier: service
spec:
  # use NodePort here to be able to access a port on each node
  type: NodePort
  ports:
  - port: 9090
  selector:
    app: gin-web
```

#### Ingress
The `ingress` allows the external access to defined services from the outside world into a Kubernetes cluster.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gin-web
  labels:
    app: gin-web
    tier: frontend
spec:
  backend:
    serviceName: gin-web
    servicePort: 9090
```

### Minikube
Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

#### Installation
To install minikube, we need first to install a Hypervisor (eg. VirtualBox) and then kubectl. For more information check this link: https://kubernetes.io/docs/tasks/tools/install-minikube/.

##### VirtualBox
Install VirtualBox from this link: https://www.virtualbox.org/wiki/Downloads.


##### kubectl

To install kubectl run the following command:
```
brew install kubernetes-cli 
```

Check if it has been installed correctly using
```
kubectl version
```

##### Minikube

To install minikube, run the following command:

```
$ brew cask install minikube
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/cask).
No changes to formulae.

==> Satisfying dependencies
All Formula dependencies satisfied.
==> Downloading https://storage.googleapis.com/minikube/releases/v0.35.0/minikube-darwin-amd64
######################################################################## 100.0%
==> Verifying SHA-256 checksum for Cask 'minikube'.
==> Installing Cask minikube
==> Linking Binary 'minikube-darwin-amd64' to '/usr/local/bin/minikube'.
🍺  minikube was successfully installed!

```


#### Starting the cluster
```
$ minikube start
😄  minikube v0.35.0 on darwin (amd64)
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
💿  Downloading Minikube ISO ...
 184.42 MB / 184.42 MB [============================================] 100.00% 0s
💣  Unable to start VM: create: precreate: VBoxManage not found. Make sure VirtualBox is installed and VBoxManage is in the path

💡  Make sure to install all necessary requirements, according to the documentation:
👉  https://kubernetes.io/docs/tasks/tools/install-minikube/
zeus:~ danielgil$ minikube start
😄  minikube v0.35.0 on darwin (amd64)
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
📶  "minikube" IP address is 192.168.99.100
🐳  Configuring Docker as the container runtime ...
✨  Preparing Kubernetes environment ...
💾  Downloading kubeadm v1.13.4
💾  Downloading kubelet v1.13.4
🚜  Pulling images required by Kubernetes v1.13.4 ...
🚀  Launching Kubernetes v1.13.4 using kubeadm ... 
⌛  Waiting for pods: apiserver proxy etcd scheduler controller addon-manager dns
🔑  Configuring cluster permissions ...
🤔  Verifying component health .....
💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!
```

#### Check IP Kubernetes is running

Display the local IP Kubernetes is running on my machine:
```
$ minikube ip
192.168.99.100
```

To check that the Kubernetes master is running at this IP:
```
kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

#### Kubernetes dashboard

To open a Kubernetes dashboard on a web browser run the following command:
``` 
$ minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:51388/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

#### kubectl (Kubernetes control)

##### Apply
Apply a configuration to a resource by filename (pass the directory where our project has the Kubernetes YML files).
```
$ kubectl apply -f kubernetes/
deployment.extensions "gin-web" created
ingress.extensions "gin-web" created
service "gin-web" created
```

##### List deployments
List the deployments in Kubernetes (we have 3 available because we specify 3 replicas):
```
$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
gin-web   3         3         3            0           2d
```

##### List pods
Display the list of pods:
```
$ kubectl get pods
NAME                       READY     STATUS             RESTARTS   AGE
gin-web-567fd44c84-8n65r   0/1       ImagePullBackOff   0          2d
gin-web-567fd44c84-pm2t6   0/1       ImagePullBackOff   0          2d
gin-web-567fd44c84-x57jg   0/1       ImagePullBackOff   0          2d
```

##### Logs
Display the logs of a pod:
``` 
$ kubectl logs gin-web-567fd44c84-8n65r
``` 

here we can see that Kubernetes pings our microservice regularly to check if it is alive and healthy.

##### List services
Display the list of services:
```
$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
gin-web      NodePort    10.110.28.47   <none>        9090:31497/TCP   19m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          2d
``` 

##### Access a service
So we have the `gin-web` service running on port 9090 (and the node port 31497). To access to the service:
``` 
$ minikube service gin-web
```
and it will accesss this Kubernetes service from within a web browser.

##### Scaling
Rescale the number of replicas:
```
$ kubectl scale deployment gin-web --replicas=8
```
