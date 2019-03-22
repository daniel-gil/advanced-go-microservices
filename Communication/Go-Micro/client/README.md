# Using Circuit Breakers for Resilient Communication

## Hystrix
We will configure Hystrix circuit breaker in the client.

### Add imports
Add imports of hystrix and breaker packages to the `/client/main.go`:
``` go
hystrix "github.com/afex/hystrix-go/hystrix"
breaker "github.com/micro/go-plugins/wrapper/breaker/hystrix"
```

### Enable hystrix circuit breaker
Enable the Hystrix circuit breaker whithin the client adding this lines to the main function:
```go
service.Init(
    micro.WrapClient(breaker.NewClientWrapper()),
)
```

### Configure Hystrix
```go
// override some default values for the Hystrix breaker
hystrix.DefaultVolumeThreshold = 3
hystrix.DefaultErrorPercentThreshold = 75
hystrix.DefaultTimeout = 500
hystrix.DefaultSleepWindow = 3500
```

### Stream handler
Add a stream handler so we can monitor the circuit breaker externally by the dashboard:
```go
// export Hystrix stream
hystrixStreamHandler := hystrix.NewStreamHandler()
hystrixStreamHandler.Start()
go http.ListenAndServe(net.JoinHostPort("", "8081"), hystrixStreamHandler)
```

### Error handling
In case an error ocurs during the call, we will check for Hystrix timeout adding this code to the `hello` function:
```go
if err != nil {
    if err.Error() == "hystrix: timeout" {
        fmt.Printf("%s. Insert fallback logic here.\n", err.Error())
    } else {
        fmt.Println(err.Error())
    }
    return
}
```

## Docker compose
Add the following lines to the `docker-compose.yml` file for the hystrix dashboard:
```docker
  hystrix-dashboard:
    image: mlabouardy/hystrix-dashboard:latest
    ports:
      - "9002:9002"
    networks:
      - sky-net
```


## Forcing a server timeout
To test the circuit breaker, we will force a timeout in the server adding this condition in the `hello` function of `/server/main.go`:
```go 
if counter > 7 && counter < 15 {
    time.Sleep(1000 * time.Millisecond)
} else {
    time.Sleep(100 * time.Millisecond)
}
```
with this, the client will think that the server is down.

## Build docker image
```bash
$ docker-compose build
```

## Run
```bash
$ docker-compose up
```

## Hystrix dashboard
After running the docker images, we can open the Hystrix dashboard in a web browser:
```
http://localhost:9002/hystrix
```

and add as data stream the following URL:
```
http://go-micro-client:8081/hystrix.stream
```

The dashboard should indicate that the circuit is close and after awhile a server timeout will happen and the circuit breaker sets the server as down, so it open de circuit. These are the logs:
```
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:44:15.4486301 +0000 UTC m=+9.004001801
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:44:15.4486301 +0000 UTC m=+9.004001801
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:44:18.4488321 +0000 UTC m=+12.004203801
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:44:18.4488321 +0000 UTC m=+12.004203801
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:44:21.4149692 +0000 UTC m=+15.005069601
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:44:21.4149692 +0000 UTC m=+15.005069601
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:44:24.4161592 +0000 UTC m=+18.006258301
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:44:24.4161592 +0000 UTC m=+18.006258301
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:44:27.4142212 +0000 UTC m=+21.004320801
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:44:27.4142212 +0000 UTC m=+21.004320801
go-micro-client_1    | hystrix: timeout. Insert fallback logic here.
go-micro-client_1    | hystrix: timeout. Insert fallback logic here.
go-micro-client_1    | hystrix: timeout. Insert fallback logic here.
go-micro-client_1    | hystrix: circuit open
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:44:30.4139995 +0000 UTC m=+24.004098501
go-micro-server_1    | 2019/03/22 15:44:40 rpc: unable to write error response: write tcp 172.21.0.4:40831->172.21.0.5:35712: write: broken pipe
go-micro-client_1    | hystrix: timeout. Insert fallback logic here.
go-micro-server_1    | 2019/03/22 15:44:43 rpc: unable to write error response: write tcp 172.21.0.4:40831->172.21.0.5:35730: write: broken pipe
```

The circuit breaker shields the client from non-responding servers.