# Implement Sync RPC calls with Binary Protocols
For this section, we will be using the [Go-Micro](https://github.com/micro/go-micro) framework.

## Generating protobuf library

Install protoc compiler, [here](https://github.com/golang/protobuf#user-content-installation) explains how to do it, and use it to generate go stubs from the `/proto/greeter.proto` file.

Here is the content of `greeter.proto` file:

```proto
syntax = "proto3";

service Greeter {
    rpc Hello(HelloRequest) returns (HelloResponse) {}
}

message HelloRequest {
    string name = 1;
}

message HelloResponse {
    string greeting = 2;
}
```

and here is how we compile it using `protoc`:
``` bash
$ cd ./Communication/Go-Micro/proto
$ protoc --go_out=. *.proto
```

this will generate the go file `greeter.pb.go` which will be used by the client and the server.

## server
The server is defined in `/server/main.go`. It implements the Greeter API function `Hello` and starts a server using Go-Micro.
``` go
func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error
```

## client
The client is defined in `/client/main.go`. It uses Consul for service discovery and creates a Greeter client to be able to call the `Hello` function every 3 seconds.

## Build docker images
We will use docker compose to build all docker images:
```bash
$ docker-compose build
consul uses an image, skipping
hystrix-dashboard uses an image, skipping
Building go-micro-server
Step 1/9 : FROM golang:1.8.1-alpine
 ---> 32efc118745e
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Using cache
 ---> 513a31355d6c
Step 3/9 : RUN go get -u github.com/micro/go-micro &&     go get -u github.com/micro/protobuf/proto &&     go get -u github.com/micro/protobuf/protoc-gen-go
 ---> Running in 23f8ad3d1394
# golang.org/x/net/internal/socket
src/golang.org/x/net/internal/socket/rawconn.go:17: undefined: syscall.RawConn
# github.com/miekg/dns
src/github.com/miekg/dns/dnssec_keyscan.go:290: undefined: strings.Builder
src/github.com/miekg/dns/msg_helpers.go:144: base32.HexEncoding.WithPadding undefined (type *base32.Encoding has no field or method WithPadding)
src/github.com/miekg/dns/msg_helpers.go:144: undefined: base32.NoPadding
src/github.com/miekg/dns/msg_helpers.go:270: undefined: strings.Builder
src/github.com/miekg/dns/serve_mux.go:43: undefined: strings.Builder
src/github.com/miekg/dns/types.go:440: undefined: strings.Builder
src/github.com/miekg/dns/types.go:464: undefined: strings.Builder
src/github.com/miekg/dns/types.go:492: undefined: strings.Builder
src/github.com/miekg/dns/types.go:513: undefined: strings.Builder
src/github.com/miekg/dns/types.go:523: undefined: strings.Builder
src/github.com/miekg/dns/types.go:492: too many errors
# golang.org/x/net/http2
src/golang.org/x/net/http2/server.go:220: s.RegisterOnShutdown undefined (type *http.Server has no field or method RegisterOnShutdown)
ERROR: Service 'go-micro-server' failed to build: The command '/bin/sh -c go get -u github.com/micro/go-micro &&     go get -u github.com/micro/protobuf/proto &&     go get -u github.com/micro/protobuf/protoc-gen-go' returned a non-zero code: 2
```

## Run
Start Consul, the server and the client using docker compose:
```bash 
$ docker-compose up
```
we can see the client/server interaction.