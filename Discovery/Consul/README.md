# Service discovery

# Consul

Consul is a service mesh solution providing a full featured control plane with service discovery, configuration, and segmentation functionality. Each of these features can be used individually as needed, or they can be used together to build a full service mesh. Consul requires a data plane and supports both a proxy and native integration model. Consul ships with a simple built-in proxy so that everything works out of the box, but also supports 3rd party proxy integrations such as Envoy.

### docker-compose.yml
Contains the required definition to start up Consul by using docker compose.

Create a service definition specifiying a consul docker image, defining the 3 ports, linking consul to demo microservices and linking it to a network.

Fire up 2 instances of the microservices from /Frameworks/Gin-Web on ports 8080 and 9090.

```
version: '2'

services:
  consul:
    image: consul:0.8.3
    ports:
      - "8300:8300"
      - "8400:8400"
      - "8500:8500"
    links:
      - gin-web-01
      - gin-web-02
    networks:
      - sky-net

  gin-web-01:
    image: gin-web:1.0.1
    environment:
      - PORT=8080
    ports:
      - "8080:8080"
    networks:
      - sky-net

  gin-web-02:
    image: gin-web:1.0.1
    environment:
      - PORT=9090
    ports:
      - "9090:9090"
    networks:
      - sky-net

networks:
  sky-net:
    driver: bridge  
```

### Start Consul
Start Consul and the 2 microservices configured:

``` bash
$ cd ./Discovery/Consul
$ docker-compose up
Creating network "consul_sky-net" with driver "bridge"
Pulling consul (consul:0.8.3)...
0.8.3: Pulling from library/consul
6f821164d5b7: Pull complete
0def154e6a6c: Pull complete
3d8c5010f2a6: Pull complete
16a3ec6049b7: Pull complete
f5c11926a0d8: Pull complete
Creating consul_gin-web-01_1 ... done
Creating consul_gin-web-02_1 ... done
Creating consul_consul_1     ... done
Attaching to consul_gin-web-01_1, consul_gin-web-02_1, consul_consul_1
gin-web-02_1  | [GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.
gin-web-02_1  | 
gin-web-02_1  | [GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
gin-web-02_1  |  - using env:   export GIN_MODE=release
gin-web-02_1  |  - using code:  gin.SetMode(gin.ReleaseMode)
gin-web-02_1  | 
gin-web-02_1  | [GIN-debug] GET    /ping                     --> main.main.func1 (3 handlers)
gin-web-01_1  | [GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.
gin-web-01_1  | 
gin-web-01_1  | [GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
gin-web-01_1  |  - using env:   export GIN_MODE=release
gin-web-01_1  |  - using code:  gin.SetMode(gin.ReleaseMode)
gin-web-01_1  | 
gin-web-01_1  | [GIN-debug] GET    /ping                     --> main.main.func1 (3 handlers)
gin-web-01_1  | [GIN-debug] GET    /hello                    --> main.main.func2 (3 handlers)
gin-web-01_1  | [GIN-debug] GET    /api/books                --> main.main.func3 (3 handlers)
gin-web-01_1  | [GIN-debug] POST   /api/books                --> main.main.func4 (3 handlers)
gin-web-01_1  | [GIN-debug] GET    /api/books/:isbn          --> main.main.func5 (3 handlers)
gin-web-01_1  | [GIN-debug] PUT    /api/books/:isbn          --> main.main.func6 (3 handlers)
gin-web-01_1  | [GIN-debug] DELETE /api/books/:isbn          --> main.main.func7 (3 handlers)
gin-web-01_1  | [GIN-debug] Loaded HTML Templates (2): 
gin-web-01_1  |         - 
gin-web-01_1  |         - index.html
gin-web-01_1  | 
gin-web-01_1  | [GIN-debug] GET    /favicon.ico              --> github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (3 handlers)
gin-web-01_1  | [GIN-debug] HEAD   /favicon.ico              --> github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (3 handlers)
gin-web-01_1  | [GIN-debug] GET    /                         --> main.main.func8 (3 handlers)
gin-web-01_1  | [GIN-debug] Listening and serving HTTP on :8080
gin-web-02_1  | [GIN-debug] GET    /hello                    --> main.main.func2 (3 handlers)
gin-web-02_1  | [GIN-debug] GET    /api/books                --> main.main.func3 (3 handlers)
gin-web-02_1  | [GIN-debug] POST   /api/books                --> main.main.func4 (3 handlers)
gin-web-02_1  | [GIN-debug] GET    /api/books/:isbn          --> main.main.func5 (3 handlers)
gin-web-02_1  | [GIN-debug] PUT    /api/books/:isbn          --> main.main.func6 (3 handlers)
gin-web-02_1  | [GIN-debug] DELETE /api/books/:isbn          --> main.main.func7 (3 handlers)
gin-web-02_1  | [GIN-debug] Loaded HTML Templates (2): 
gin-web-02_1  |         - 
gin-web-02_1  |         - index.html
gin-web-02_1  | 
gin-web-02_1  | [GIN-debug] GET    /favicon.ico              --> github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (3 handlers)
gin-web-02_1  | [GIN-debug] HEAD   /favicon.ico              --> github.com/gin-gonic/gin.(*RouterGroup).StaticFile.func1 (3 handlers)
gin-web-02_1  | [GIN-debug] GET    /                         --> main.main.func8 (3 handlers)
gin-web-02_1  | [GIN-debug] Listening and serving HTTP on :9090
consul_1      | ==> Starting Consul agent...
consul_1      | ==> Consul agent running!
consul_1      |            Version: 'v0.8.3'
consul_1      |            Node ID: '3c5eeeff-e5d6-afae-a297-5058f80a1b93'
consul_1      |          Node name: '9014a96ed9bb'
consul_1      |         Datacenter: 'dc1'
consul_1      |             Server: true (bootstrap: false)
consul_1      |        Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 8600)
consul_1      |       Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
consul_1      |     Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
consul_1      |              Atlas: <disabled>
consul_1      | 
consul_1      | ==> Log data will now stream in as it occurs:
consul_1      | 
consul_1      |     2019/03/19 09:50:29 [DEBUG] Using unique ID "3c5eeeff-e5d6-afae-a297-5058f80a1b93" from host as node ID
consul_1      |     2019/03/19 09:50:29 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
consul_1      |     2019/03/19 09:50:29 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
consul_1      |     2019/03/19 09:50:29 [INFO] serf: EventMemberJoin: 9014a96ed9bb 127.0.0.1
consul_1      |     2019/03/19 09:50:29 [INFO] consul: Adding LAN server 9014a96ed9bb (Addr: tcp/127.0.0.1:8300) (DC: dc1)
consul_1      |     2019/03/19 09:50:29 [INFO] serf: EventMemberJoin: 9014a96ed9bb.dc1 127.0.0.1
consul_1      |     2019/03/19 09:50:29 [INFO] consul: Handled member-join event for server "9014a96ed9bb.dc1" in area "wan"
consul_1      |     2019/03/19 09:50:29 [WARN] raft: Heartbeat timeout from "" reached, starting election
consul_1      |     2019/03/19 09:50:29 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
consul_1      |     2019/03/19 09:50:29 [DEBUG] raft: Votes needed: 1
consul_1      |     2019/03/19 09:50:29 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
consul_1      |     2019/03/19 09:50:29 [INFO] raft: Election won. Tally: 1
consul_1      |     2019/03/19 09:50:29 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
consul_1      |     2019/03/19 09:50:29 [INFO] consul: cluster leadership acquired
consul_1      |     2019/03/19 09:50:29 [INFO] consul: New leader elected: 9014a96ed9bb
consul_1      |     2019/03/19 09:50:29 [DEBUG] consul: reset tombstone GC to index 3
consul_1      |     2019/03/19 09:50:29 [INFO] consul: member '9014a96ed9bb' joined, marking health alive
consul_1      |     2019/03/19 09:50:29 [INFO] agent: Synced service 'consul'
consul_1      |     2019/03/19 09:50:29 [DEBUG] agent: Node info in sync
```

Now we can open a web browser and navigate to the UI of Consul:
```
http://localhost:8500
```

At this point, we can see in the Consul dashboard (Service tab) that the only service registered is Consul itself.

## Registering new service
Using the REST API of consul itself to register few services. To achieve this, we will use the REST client Postman.

### Service catalog
URL to access the service catalog:
```
GET http://localhost:8500/v1/catalog/services
```

### Service agents
URL to access the service agents:
```
GET http://localhost:8500/v1/agent/services
```

### Register first microservice
Register a service for the first microservice:


```
PUT http://localhost:8500/v1/agent/service/register
Body:
{
	"ID": "gin-web-01",
	"Name": "gin-web",
	"Tags": [
		"cloud-native-go",
		"v1"
	],
	"Address": "gin-web-01",
	"Port": 8080,
	"EnableTagOverride": false,
	"check": {
		"id": "ping", 
		"name": "HTTP API on port 8080",
		"http": "http://gin-web-01:8080/ping",
		"interval": "5s",
		"timeout": "1s"
	}
}
```

After registering the 2 instances (`gin-web-01` and `gin-web-02`) under the same name `gin-web`, we can verify the new registration by calling the agent services:

Request:
```
GET http://localhost:8500/v1/agent/services
``` 

Response:
```
{
    "consul": {
        "ID": "consul",
        "Service": "consul",
        "Tags": [],
        "Address": "",
        "Port": 8300,
        "EnableTagOverride": false,
        "CreateIndex": 0,
        "ModifyIndex": 0
    },
    "gin-web-01": {
        "ID": "gin-web-01",
        "Service": "gin-web",
        "Tags": [
            "cloud-native-go",
            "v1"
        ],
        "Address": "gin-web-01",
        "Port": 8080,
        "EnableTagOverride": false,
        "CreateIndex": 0,
        "ModifyIndex": 0
    },
    "gin-web-02": {
        "ID": "gin-web-02",
        "Service": "gin-web",
        "Tags": [
            "cloud-native-go",
            "v1"
        ],
        "Address": "gin-web-02",
        "Port": 9090,
        "EnableTagOverride": false,
        "CreateIndex": 0,
        "ModifyIndex": 0
    }
}
```

Consul is responsible for checking the healthy of the microservices, it calls constantly the ping URL of both services.

If we list the docker images, we will see the consul itself and the 2 services running:

``` bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                                      NAMES
9014a96ed9bb        consul:0.8.3        "docker-entrypoint.s…"   About an hour ago   Up About an hour    0.0.0.0:8300->8300/tcp, 0.0.0.0:8400->8400/tcp, 8301-8302/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp, 8600/tcp, 8600/udp   consul_consul_1
1a0467e07d44        gin-web:1.0.1       "/bin/sh -c ${SOURCE…"   About an hour ago   Up About an hour    0.0.0.0:8080->8080/tcp                                                                                                     consul_gin-web-01_1
3ee6a67e8123        gin-web:1.0.1       "/bin/sh -c ${SOURCE…"   About an hour ago   Up About an hour    8080/tcp, 0.0.0.0:9090->9090/tcp    
```

### Forcing a service failure

If we stop one of the containers:
``` bash
$ docker stop consul_gin-web-01_1
consul_gin-web-01_1
```

we will see in the consul logs that the ping is failing, the service became unhealthy:
``` bash
[WARN] agent: http request failed 'http://gin-web-01:8080/ping': Get http://gin-web-01:8080/ping: dial tcp: lookup gin-web-01 on 127.0.0.11:53: no such host
```

### Check service health

#### Get health of all services

Request: 
``` 
GET http://localhost:8500/v1/health/service/gin-web
```

Response:
```
[
    {
        "Node": {
            "ID": "3c5eeeff-e5d6-afae-a297-5058f80a1b93",
            "Node": "9014a96ed9bb",
            "Address": "127.0.0.1",
            "Datacenter": "dc1",
            "TaggedAddresses": {
                "lan": "127.0.0.1",
                "wan": "127.0.0.1"
            },
            "Meta": {},
            "CreateIndex": 5,
            "ModifyIndex": 6
        },
        "Service": {
            "ID": "gin-web-01",
            "Service": "gin-web",
            "Tags": [
                "cloud-native-go",
                "v1"
            ],
            "Address": "gin-web-01",
            "Port": 8080,
            "EnableTagOverride": false,
            "CreateIndex": 171,
            "ModifyIndex": 171
        },
        "Checks": [
            {
                "Node": "9014a96ed9bb",
                "CheckID": "serfHealth",
                "Name": "Serf Health Status",
                "Status": "passing",
                "Notes": "",
                "Output": "Agent alive and reachable",
                "ServiceID": "",
                "ServiceName": "",
                "ServiceTags": [],
                "CreateIndex": 5,
                "ModifyIndex": 5
            },
            {
                "Node": "9014a96ed9bb",
                "CheckID": "service:gin-web-01",
                "Name": "Service 'gin-web' check",
                "Status": "critical",
                "Notes": "",
                "Output": "Get http://gin-web-01:8080/ping: dial tcp: lookup gin-web-01 on 127.0.0.11:53: no such host",
                "ServiceID": "gin-web-01",
                "ServiceName": "gin-web",
                "ServiceTags": [
                    "cloud-native-go",
                    "v1"
                ],
                "CreateIndex": 171,
                "ModifyIndex": 233
            }
        ]
    },
    {
        "Node": {
            "ID": "3c5eeeff-e5d6-afae-a297-5058f80a1b93",
            "Node": "9014a96ed9bb",
            "Address": "127.0.0.1",
            "Datacenter": "dc1",
            "TaggedAddresses": {
                "lan": "127.0.0.1",
                "wan": "127.0.0.1"
            },
            "Meta": {},
            "CreateIndex": 5,
            "ModifyIndex": 6
        },
        "Service": {
            "ID": "gin-web-02",
            "Service": "gin-web",
            "Tags": [
                "cloud-native-go",
                "v1"
            ],
            "Address": "gin-web-02",
            "Port": 9090,
            "EnableTagOverride": false,
            "CreateIndex": 188,
            "ModifyIndex": 188
        },
        "Checks": [
            {
                "Node": "9014a96ed9bb",
                "CheckID": "serfHealth",
                "Name": "Serf Health Status",
                "Status": "passing",
                "Notes": "",
                "Output": "Agent alive and reachable",
                "ServiceID": "",
                "ServiceName": "",
                "ServiceTags": [],
                "CreateIndex": 5,
                "ModifyIndex": 5
            },
            {
                "Node": "9014a96ed9bb",
                "CheckID": "service:gin-web-02",
                "Name": "Service 'gin-web' check",
                "Status": "passing",
                "Notes": "",
                "Output": "HTTP GET http://gin-web-02:9090/ping: 200 OK Output: pong",
                "ServiceID": "gin-web-02",
                "ServiceName": "gin-web",
                "ServiceTags": [
                    "cloud-native-go",
                    "v1"
                ],
                "CreateIndex": 188,
                "ModifyIndex": 234
            }
        ]
    }
] 
```

#### Get information of healthy services
Only returns the services where the health is OK.

Request:
``` 
GET http://localhost:8500/v1/health/service/gin-web?passing=
```

Response:
```
[
    {
        "Node": {
            "ID": "3c5eeeff-e5d6-afae-a297-5058f80a1b93",
            "Node": "9014a96ed9bb",
            "Address": "127.0.0.1",
            "Datacenter": "dc1",
            "TaggedAddresses": {
                "lan": "127.0.0.1",
                "wan": "127.0.0.1"
            },
            "Meta": {},
            "CreateIndex": 5,
            "ModifyIndex": 6
        },
        "Service": {
            "ID": "gin-web-02",
            "Service": "gin-web",
            "Tags": [
                "cloud-native-go",
                "v1"
            ],
            "Address": "gin-web-02",
            "Port": 9090,
            "EnableTagOverride": false,
            "CreateIndex": 188,
            "ModifyIndex": 188
        },
        "Checks": [
            {
                "Node": "9014a96ed9bb",
                "CheckID": "serfHealth",
                "Name": "Serf Health Status",
                "Status": "passing",
                "Notes": "",
                "Output": "Agent alive and reachable",
                "ServiceID": "",
                "ServiceName": "",
                "ServiceTags": [],
                "CreateIndex": 5,
                "ModifyIndex": 5
            },
            {
                "Node": "9014a96ed9bb",
                "CheckID": "service:gin-web-02",
                "Name": "Service 'gin-web' check",
                "Status": "passing",
                "Notes": "",
                "Output": "HTTP GET http://gin-web-02:9090/ping: 200 OK Output: pong",
                "ServiceID": "gin-web-02",
                "ServiceName": "gin-web",
                "ServiceTags": [
                    "cloud-native-go",
                    "v1"
                ],
                "CreateIndex": 188,
                "ModifyIndex": 345
            }
        ]
    }
] 
```

#### Get the unhealthy services
Check for the critical states:

Request:
``` 
GET http://localhost:8500/v1/health/state/critical
```

Response: 
``` 
[
    {
        "Node": "9014a96ed9bb",
        "CheckID": "service:gin-web-01",
        "Name": "Service 'gin-web' check",
        "Status": "critical",
        "Notes": "",
        "Output": "Get http://gin-web-01:8080/ping: dial tcp: lookup gin-web-01 on 127.0.0.11:53: no such host",
        "ServiceID": "gin-web-01",
        "ServiceName": "gin-web",
        "ServiceTags": [
            "cloud-native-go",
            "v1"
        ],
        "CreateIndex": 171,
        "ModifyIndex": 359
    }
]
```
