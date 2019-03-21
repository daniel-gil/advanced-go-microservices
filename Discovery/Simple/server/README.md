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
$ cd ./Discovery/Simple
$ docker-compose build
consul uses an image, skipping
Building simple-server
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
 ---> d77acfdfb8ec
Step 6/9 : RUN cd ${SOURCES}server/ && CGO_ENABLED=0 go build
 ---> Running in cda45f254eb8
Removing intermediate container cda45f254eb8
 ---> 9e4c31243065
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in bbd19620d98d
Removing intermediate container bbd19620d98d
 ---> 4869691ed2f7
Step 8/9 : WORKDIR ${SOURCES}server/
 ---> Running in f41e64c7c1a0
Removing intermediate container f41e64c7c1a0
 ---> 8f66c7bed13a
Step 9/9 : CMD ${SOURCES}server/server
 ---> Running in d40d219f0838
Removing intermediate container d40d219f0838
 ---> 573776b437ea
Successfully built 573776b437ea
Successfully tagged simple-server:1.0.1
```

### Start docker image
Fire a Consul as well as the simple microservice:

``` bash
$ docker-compose up
Starting simple_consul_1 ... done
Recreating simple_simple-server_1 ... done
Attaching to simple_consul_1, simple_simple-server_1
simple-server_1  | Starting Simple Server.
consul_1         | ==> Starting Consul agent...
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
consul_1         |     2019/03/21 09:37:15 [DEBUG] Using unique ID "3c5eeeff-e5d6-afae-a297-5058f80a1b93" from host as node ID
consul_1         |     2019/03/21 09:37:15 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
consul_1         |     2019/03/21 09:37:15 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
consul_1         |     2019/03/21 09:37:15 [INFO] serf: EventMemberJoin: 412cdc44c745 127.0.0.1
consul_1         |     2019/03/21 09:37:15 [INFO] serf: EventMemberJoin: 412cdc44c745.dc1 127.0.0.1
consul_1         |     2019/03/21 09:37:15 [INFO] consul: Handled member-join event for server "412cdc44c745.dc1" in area "wan"
consul_1         |     2019/03/21 09:37:15 [INFO] consul: Adding LAN server 412cdc44c745 (Addr: tcp/127.0.0.1:8300) (DC: dc1)
consul_1         |     2019/03/21 09:37:15 [WARN] raft: Heartbeat timeout from "" reached, starting election
consul_1         |     2019/03/21 09:37:15 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
consul_1         |     2019/03/21 09:37:15 [DEBUG] raft: Votes needed: 1
consul_1         |     2019/03/21 09:37:15 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
consul_1         |     2019/03/21 09:37:15 [INFO] raft: Election won. Tally: 1
consul_1         |     2019/03/21 09:37:15 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
consul_1         |     2019/03/21 09:37:15 [INFO] consul: cluster leadership acquired
consul_1         |     2019/03/21 09:37:15 [INFO] consul: New leader elected: 412cdc44c745
consul_1         |     2019/03/21 09:37:15 [DEBUG] consul: reset tombstone GC to index 3
consul_1         |     2019/03/21 09:37:15 [INFO] consul: member '412cdc44c745' joined, marking health alive
consul_1         |     2019/03/21 09:37:15 [INFO] agent: Synced service 'consul'
consul_1         |     2019/03/21 09:37:15 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: pausing 456.697941ms before first HTTP request of http://ff1cc77b53e9:8080/info
consul_1         |     2019/03/21 09:37:16 [INFO] agent: Synced service 'simple-server'
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] http: Request PUT /v1/agent/service/register (4.3729ms) from=172.20.0.3:44968
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Node info in sync
simple-server_1  | The /info endpoint is being called...
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Check 'service:simple-server' is passing
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/21 09:37:16 [INFO] agent: Synced check 'service:simple-server'
consul_1         |     2019/03/21 09:37:16 [DEBUG] agent: Node info in sync
simple-server_1  | The /info endpoint is being called...
consul_1         |     2019/03/21 09:37:21 [DEBUG] agent: Check 'service:simple-server' is passing
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
consul_1         |     2019/03/21 09:39:55 [WARN] agent: http request failed 'http://9ee38bb6b8e8:8080/info': Get http://9ee38bb6b8e8:8080/info: dial tcp: lookup 9ee38bb6b8e8 on 127.0.0.11:53: no such host
```