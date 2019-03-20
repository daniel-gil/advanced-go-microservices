# Implement Go Microservice Registration with Consul

## server
The function `registerServiceWithConsul` from the `simple-server.go` file, registers to an agent a service called `simple-server` and checks that the service is alive and healthy.

### Compile server
Compile the server with 
``` bash
cd ./Discovery/Simple/server
go build
```

### Build docker image
To check the service we need to start the server with consul using docker-compose.

```bash
$ docker-compose build
consul uses an image, skipping
Building simple-server
Step 1/9 : FROM golang:1.8.1-alpine
1.8.1-alpine: Pulling from library/golang
cfc728c1c558: Pull complete
4f9d39540db5: Pull complete
3a62b0235a60: Pull complete
e64c614fb8fc: Pull complete
028d597756bf: Pull complete
3fa96bfc6500: Pull complete
Digest: sha256:f71f38bdcf35dcb0fbe69730cb775bfd29662373d8c5ac1f6c684ee95f88e716
Status: Downloaded newer image for golang:1.8.1-alpine
 ---> 32efc118745e
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Running in 519f46810e20
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/community/x86_64/APKINDEX.tar.gz
v3.5.3-40-g389d0b359a [http://dl-cdn.alpinelinux.org/alpine/v3.5/main]
v3.5.3-40-g389d0b359a [http://dl-cdn.alpinelinux.org/alpine/v3.5/community]
OK: 7968 distinct packages available
Upgrading critical system libraries and apk-tools:
(1/1) Upgrading apk-tools (2.6.8-r2 -> 2.6.10-r0)
Executing busybox-1.25.1-r0.trigger
Continuing the upgrade transaction with new apk-tools:
(1/4) Upgrading musl (1.1.15-r6 -> 1.1.15-r8)
(2/4) Upgrading busybox (1.25.1-r0 -> 1.25.1-r2)
Executing busybox-1.25.1-r2.post-upgrade
(3/4) Upgrading zlib (1.2.8-r2 -> 1.2.11-r0)
(4/4) Upgrading musl-utils (1.1.15-r6 -> 1.1.15-r8)
Executing busybox-1.25.1-r2.trigger
OK: 5 MiB in 12 packages
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/community/x86_64/APKINDEX.tar.gz
(1/10) Installing ncurses-terminfo-base (6.0_p20171125-r1)
(2/10) Installing ncurses-terminfo (6.0_p20171125-r1)
(3/10) Installing ncurses-libs (6.0_p20171125-r1)
(4/10) Installing readline (6.3.008-r4)
(5/10) Installing bash (4.3.46-r5)
Executing bash-4.3.46-r5.post-install
(6/10) Installing libssh2 (1.7.0-r2)
(7/10) Installing libcurl (7.61.1-r1)
(8/10) Installing expat (2.2.0-r1)
(9/10) Installing pcre (8.39-r0)
(10/10) Installing git (2.11.3-r2)
Executing busybox-1.25.1-r2.trigger
OK: 32 MiB in 22 packages
Removing intermediate container 519f46810e20
 ---> 513a31355d6c
Step 3/9 : RUN go get github.com/hashicorp/consul/api
 ---> Running in 322da762451b
Removing intermediate container 322da762451b
 ---> 083df7e35455
Step 4/9 : ENV SOURCES /go/src/github.com/PacktPublishing/Advanced-Cloud-Native-Go/Discovery/Simple/
 ---> Running in 889a7f4d761e
Removing intermediate container 889a7f4d761e
 ---> 5b5dd790b972
Step 5/9 : COPY . ${SOURCES}
 ---> 6ccaefa7bb74
Step 6/9 : RUN cd ${SOURCES}server/ && CGO_ENABLED=0 go build
 ---> Running in 0380c47f0222
Removing intermediate container 0380c47f0222
 ---> 2ad26da191d3
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in 52944ffc7694
Removing intermediate container 52944ffc7694
 ---> d4b7bcea274c
Step 8/9 : WORKDIR ${SOURCES}server/
 ---> Running in 49947207bbba
Removing intermediate container 49947207bbba
 ---> e2127890956c
Step 9/9 : CMD ${SOURCES}server/server
 ---> Running in e32f33d9553d
Removing intermediate container e32f33d9553d
 ---> 5970b5885f12
Successfully built 5970b5885f12
Successfully tagged simple-server:1.0.1
Building simple-client
Step 1/9 : FROM golang:1.8.1-alpine
 ---> 32efc118745e
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Using cache
 ---> 513a31355d6c
Step 3/9 : RUN go get github.com/hashicorp/consul/api
 ---> Using cache
 ---> 083df7e35455
Step 4/9 : ENV SOURCES /go/src/github.com/PacktPublishing/Advanced-Cloud-Native-Go/Discovery/Simple/
 ---> Using cache
 ---> 5b5dd790b972
Step 5/9 : COPY . ${SOURCES}
 ---> Using cache
 ---> 6ccaefa7bb74
Step 6/9 : RUN cd ${SOURCES}client/ && CGO_ENABLED=0 go build
 ---> Running in bef9348ec46b
Removing intermediate container bef9348ec46b
 ---> 0f8466405eae
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in e4726d5c1662
Removing intermediate container e4726d5c1662
 ---> 06f0c9d05124
Step 8/9 : WORKDIR ${SOURCES}client/
 ---> Running in 361aae7ef9a4
Removing intermediate container 361aae7ef9a4
 ---> 25bf8b257ccd
Step 9/9 : CMD ${SOURCES}client/client
 ---> Running in 9265e4d62ee9
Removing intermediate container 9265e4d62ee9
 ---> ed6b13bf2f66
Successfully built ed6b13bf2f66
Successfully tagged simple-client:1.0.1
```

### Start docker image
Fire a Consul as well as the simple microservice:

``` bash
$ docker-compose up
Starting simple_consul_1 ... done
Creating simple_simple-server_1 ... done
Creating simple_simple-client_1 ... done
Attaching to simple_consul_1, simple_simple-server_1, simple_simple-client_1
simple-server_1  | Starting Simple Server.
consul_1         | ==> Starting Consul agent...
simple-client_1  | Starting Simple Client
consul_1         | ==> Consul agent running!
consul_1         |            Version: 'v0.8.3'
consul_1         |            Node ID: '3c5eeeff-e5d6-afae-a297-5058f80a1b93'
consul_1         |          Node name: '412cdc44c745'
consul_1         |         Datacenter: 'dc1'
consul_1         |             Server: true (bootstrap: false)
consul_1         |        Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 8600)
consul_1         |       Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
consul_1         |     Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
consul_1         |              Atlas: <disabled>
consul_1         | 
consul_1         | ==> Log data will now stream in as it occurs:
consul_1         | 
consul_1         |     2019/03/20 16:33:29 [DEBUG] Using unique ID "3c5eeeff-e5d6-afae-a297-5058f80a1b93" from host as node ID
consul_1         |     2019/03/20 16:33:29 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
consul_1         |     2019/03/20 16:33:29 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
consul_1         |     2019/03/20 16:33:29 [INFO] serf: EventMemberJoin: 412cdc44c745 127.0.0.1
consul_1         |     2019/03/20 16:33:29 [INFO] consul: Adding LAN server 412cdc44c745 (Addr: tcp/127.0.0.1:8300) (DC: dc1)
consul_1         |     2019/03/20 16:33:29 [INFO] serf: EventMemberJoin: 412cdc44c745.dc1 127.0.0.1
consul_1         |     2019/03/20 16:33:29 [INFO] consul: Handled member-join event for server "412cdc44c745.dc1" in area "wan"
consul_1         |     2019/03/20 16:33:29 [WARN] raft: Heartbeat timeout from "" reached, starting election
consul_1         |     2019/03/20 16:33:29 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
consul_1         |     2019/03/20 16:33:29 [DEBUG] raft: Votes needed: 1
consul_1         |     2019/03/20 16:33:29 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
consul_1         |     2019/03/20 16:33:29 [INFO] raft: Election won. Tally: 1
consul_1         |     2019/03/20 16:33:29 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
consul_1         |     2019/03/20 16:33:29 [INFO] consul: cluster leadership acquired
consul_1         |     2019/03/20 16:33:29 [INFO] consul: New leader elected: 412cdc44c745
consul_1         |     2019/03/20 16:33:29 [DEBUG] consul: reset tombstone GC to index 3
consul_1         |     2019/03/20 16:33:29 [INFO] consul: member '412cdc44c745' joined, marking health alive
consul_1         |     2019/03/20 16:33:30 [INFO] agent: Synced service 'consul'
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: pausing 4.595551903s before first HTTP request of http://9ee38bb6b8e8:8080/info
consul_1         |     2019/03/20 16:33:30 [INFO] agent: Synced service 'simple-server'
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] http: Request PUT /v1/agent/service/register (958.6µs) from=172.20.0.3:44802
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/20 16:33:30 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/20 16:33:31 [DEBUG] http: Request GET /v1/agent/services (142.7µs) from=172.20.0.4:48140
simple-server_1  | The /info endpoint is being called...
consul_1         |     2019/03/20 16:33:35 [DEBUG] agent: Check 'service:simple-server' is passing
consul_1         |     2019/03/20 16:33:35 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/20 16:33:35 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/20 16:33:35 [INFO] agent: Synced check 'service:simple-server'
consul_1         |     2019/03/20 16:33:35 [DEBUG] agent: Node info in sync
simple-server_1  | The /info endpoint is being called...
simple-client_1  | Hello Consul Discovery. Time is 2019-03-20 16:33:36.3924589 +0000 UTC
simple-server_1  | The /info endpoint is being called...
consul_1         |     2019/03/20 16:33:40 [DEBUG] agent: Check 'service:simple-server' is passing
simple-client_1  | Hello Consul Discovery. Time is 2019-03-20 16:33:41.3912262 +0000 UTC
simple-server_1  | The /info endpoint is being called...
```

We can see in the logs that the simple-server is echoing periodically the following output:

```
simple-server_1  | The /info endpoint is being called...
```

because Consul is checking the info endpoint regularly to verify that the simple-service is alive.

We can test Consul stopping the server:
``` bash
$ docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                                                                                                      NAMES
9ee38bb6b8e8        simple-server:1.0.1   "/bin/sh -c ${SOURCE…"   3 minutes ago       Up 3 minutes                                                                                                                                   simple_simple-server_1
412cdc44c745        consul:0.8.3          "docker-entrypoint.s…"   4 minutes ago       Up 3 minutes        0.0.0.0:8300->8300/tcp, 0.0.0.0:8400->8400/tcp, 8301-8302/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp, 8600/tcp, 8600/udp   simple_consul_1
```

```bash
$ docker stop simple_simple-server_1
simple_simple-server_1
```

and Consul realize that the simple-server is not running anymore and that the service is unhealthy:
```bash
consul_1         |     2019/03/20 16:39:55 [WARN] agent: http request failed 'http://9ee38bb6b8e8:8080/info': Get http://9ee38bb6b8e8:8080/info: dial tcp: lookup 9ee38bb6b8e8 on 127.0.0.11:53: no such host
```