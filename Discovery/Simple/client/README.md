# Implement Go Microservice Lookup with Consul

## client
The function `lookupServiceWithConsul` from the `simple-client.go` file, will connect to Consul, perform a lookup for the simple-server and construct the endpoint we need to call from the server registration.

The function build a consul client and access the agent API to query all the registered services within Consul. After this, we access to the service that we are interested in, with name `simple-server`, and then we can read the address and port of the service.


### Build docker image
Now we are building the docker image containing Consul, simple-server and simple-client.

```bash
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
 ---> 3c4752bf64e0
Step 6/9 : RUN cd ${SOURCES}server/ && CGO_ENABLED=0 go build
 ---> Running in 003847b13bda
Removing intermediate container 003847b13bda
 ---> ab712d865d54
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in cec44e47f853
Removing intermediate container cec44e47f853
 ---> f0dca955538d
Step 8/9 : WORKDIR ${SOURCES}server/
 ---> Running in b79a668bc366
Removing intermediate container b79a668bc366
 ---> 0a47ba18e19e
Step 9/9 : CMD ${SOURCES}server/server
 ---> Running in 47029cb8c509
Removing intermediate container 47029cb8c509
 ---> 0521488f8cf0
Successfully built 0521488f8cf0
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
 ---> 3c4752bf64e0
Step 6/9 : RUN cd ${SOURCES}client/ && CGO_ENABLED=0 go build
 ---> Running in 5641566b484e
Removing intermediate container 5641566b484e
 ---> b74f345f9ce8
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in c9ff837919ce
Removing intermediate container c9ff837919ce
 ---> f1de545b8a8a
Step 8/9 : WORKDIR ${SOURCES}client/
 ---> Running in 3fc764e5dd7e
Removing intermediate container 3fc764e5dd7e
 ---> 394308aad9ba
Step 9/9 : CMD ${SOURCES}client/client
 ---> Running in 5583a62c50f1
Removing intermediate container 5583a62c50f1
 ---> 815722d13cde
Successfully built 815722d13cde
Successfully tagged simple-client:1.0.1
```


### Start docker image
Fire a Consul as well as the simple microservice:

``` bash
zeus:Simple danielgil$ docker-compose up
Starting simple_consul_1 ... done
Recreating simple_simple-server_1 ... done
Recreating simple_simple-client_1 ... done
Attaching to simple_consul_1, simple_simple-server_1, simple_simple-client_1
consul_1         | ==> Starting Consul agent...
simple-server_1  | Starting Simple Server.
consul_1         | ==> Consul agent running!
consul_1         |            Version: 'v0.8.3'
simple-client_1  | Starting Simple Client
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
consul_1         |     2019/03/21 09:41:57 [DEBUG] Using unique ID "3c5eeeff-e5d6-afae-a297-5058f80a1b93" from host as node ID
consul_1         |     2019/03/21 09:41:57 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
consul_1         |     2019/03/21 09:41:57 [INFO] serf: EventMemberJoin: 412cdc44c745 127.0.0.1
consul_1         |     2019/03/21 09:41:57 [INFO] serf: EventMemberJoin: 412cdc44c745.dc1 127.0.0.1
consul_1         |     2019/03/21 09:41:57 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
consul_1         |     2019/03/21 09:41:57 [INFO] consul: Adding LAN server 412cdc44c745 (Addr: tcp/127.0.0.1:8300) (DC: dc1)
consul_1         |     2019/03/21 09:41:57 [INFO] consul: Handled member-join event for server "412cdc44c745.dc1" in area "wan"
consul_1         |     2019/03/21 09:41:57 [WARN] raft: Heartbeat timeout from "" reached, starting election
consul_1         |     2019/03/21 09:41:57 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
consul_1         |     2019/03/21 09:41:57 [DEBUG] raft: Votes needed: 1
consul_1         |     2019/03/21 09:41:57 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
consul_1         |     2019/03/21 09:41:57 [INFO] raft: Election won. Tally: 1
consul_1         |     2019/03/21 09:41:57 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
consul_1         |     2019/03/21 09:41:57 [INFO] consul: cluster leadership acquired
consul_1         |     2019/03/21 09:41:57 [DEBUG] consul: reset tombstone GC to index 3
consul_1         |     2019/03/21 09:41:57 [INFO] consul: member '412cdc44c745' joined, marking health alive
consul_1         |     2019/03/21 09:41:57 [INFO] consul: New leader elected: 412cdc44c745
consul_1         |     2019/03/21 09:41:57 [INFO] agent: Synced service 'consul'
consul_1         |     2019/03/21 09:41:57 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:41:58 [INFO] agent: Synced service 'simple-server'
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] http: Request PUT /v1/agent/service/register (954.4µs) from=172.20.0.3:45014
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Check 'service:simple-server' in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: Node info in sync
consul_1         |     2019/03/21 09:41:58 [DEBUG] agent: pausing 4.512637805s before first HTTP request of http://eea8d2111821:8080/info
consul_1         |     2019/03/21 09:41:59 [DEBUG] http: Request GET /v1/agent/services (314.8µs) from=172.20.0.4:48352
simple-server_1  | The /info endpoint is being called...
consul_1         |     2019/03/21 09:42:02 [DEBUG] agent: Check 'service:simple-server' is passing
consul_1         |     2019/03/21 09:42:02 [DEBUG] agent: Service 'consul' in sync
consul_1         |     2019/03/21 09:42:02 [DEBUG] agent: Service 'simple-server' in sync
consul_1         |     2019/03/21 09:42:02 [INFO] agent: Synced check 'service:simple-server'
consul_1         |     2019/03/21 09:42:02 [DEBUG] agent: Node info in sync
simple-server_1  | The /info endpoint is being called...
simple-client_1  | Hello Consul Discovery. Time is 2019-03-21 09:42:04.2643383 +0000 UTC
consul_1         | ==> Newer Consul version available: 1.4.3 (currently running: 0.8.3)
consul_1         |     2019/03/21 09:42:07 [DEBUG] agent: Check 'service:simple-server' is passing
simple-server_1  | The /info endpoint is being called...
simple-client_1  | Hello Consul Discovery. Time is 2019-03-21 09:42:09.2643637 +0000 UTC
simple-server_1  | The /info endpoint is being called...
simple-server_1  | The /info endpoint is being called...
consul_1         |     2019/03/21 09:42:12 [DEBUG] agent: Check 'service:simple-server' is passing
simple-client_1  | Hello Consul Discovery. Time is 2019-03-21 09:42:14.2646375 +0000 UTC
simple-server_1  | The /info endpoint is being called...
```

we can see in the logs that the client is calling each 5 seconds the server.