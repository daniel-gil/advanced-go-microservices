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
$$ docker-compose build
consul uses an image, skipping
hystrix-dashboard uses an image, skipping
Building go-micro-server
Step 1/9 : FROM golang:1.12.1-alpine
 ---> c0e5aac9423b
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Using cache
 ---> 0636afd40b6b
Step 3/9 : RUN go get -u github.com/micro/go-micro &&     go get github.com/micro/protobuf/proto &&     go get -u github.com/micro/protobuf/protoc-gen-go
 ---> Using cache
 ---> 00eaf3b8c703
Step 4/9 : ENV SOURCES /go/src/github.com/daniel-gil/advanced-go-microservices/Communication/Go-Micro/
 ---> Using cache
 ---> 6047284e87c7
Step 5/9 : COPY . ${SOURCES}
 ---> bf9870558ebc
Step 6/9 : RUN cd ${SOURCES}server/ && CGO_ENABLED=0 go build
 ---> Running in c5101a94816e
Removing intermediate container c5101a94816e
 ---> cd937794fb4c
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in a923b3d5e52c
Removing intermediate container a923b3d5e52c
 ---> c3b5d0ff4621
Step 8/9 : WORKDIR ${SOURCES}server/
 ---> Running in b4341da8b4b6
Removing intermediate container b4341da8b4b6
 ---> 57b575341b1d
Step 9/9 : CMD ${SOURCES}server/server
 ---> Running in 302cfb63a72d
Removing intermediate container 302cfb63a72d
 ---> d640c013ac22
Successfully built d640c013ac22
Successfully tagged go-micro-server:1.0.1
Building go-micro-client
Step 1/9 : FROM golang:1.12.1-alpine
 ---> c0e5aac9423b
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Using cache
 ---> 0636afd40b6b
Step 3/9 : RUN go get -u github.com/micro/micro &&     go get github.com/micro/protobuf/proto &&     go get -u github.com/micro/protobuf/protoc-gen-go &&     go get github.com/micro/go-plugins/wrapper/breaker/hystrix &&     go get github.com/afex/hystrix-go/hystrix
 ---> Using cache
 ---> f5a7d190c0b3
Step 4/9 : ENV SOURCES /go/src/github.com/daniel-gil/advanced-go-microservices/Communication/Go-Micro/
 ---> Running in cce4421467b1
Removing intermediate container cce4421467b1
 ---> f8448deac077
Step 5/9 : COPY . ${SOURCES}
 ---> dc0951cede83
Step 6/9 : RUN cd ${SOURCES}client/ && CGO_ENABLED=0 go build
 ---> Running in 51cd45b323af
Removing intermediate container 51cd45b323af
 ---> 43960c6359ba
Step 7/9 : ENV CONSUL_HTTP_ADDR localhost:8500
 ---> Running in e44fcca33793
Removing intermediate container e44fcca33793
 ---> 89be72ad2368
Step 8/9 : WORKDIR ${SOURCES}client/
 ---> Running in d6013160d07a
Removing intermediate container d6013160d07a
 ---> 67d2fbc74345
Step 9/9 : CMD ${SOURCES}client/client
 ---> Running in 74c082821e44
Removing intermediate container 74c082821e44
 ---> 78788eef751f
Successfully built 78788eef751f
Successfully tagged go-micro-client:1.0.1
```

## Run
Start Consul, the server and the client using docker compose:
```bash 
$ docker-compose up
Creating network "go-micro_sky-net" with driver "bridge"
Pulling hystrix-dashboard (mlabouardy/hystrix-dashboard:latest)...
latest: Pulling from mlabouardy/hystrix-dashboard
03e1855d4f31: Pull complete
a3ed95caeb02: Pull complete
5b3a2df3e63b: Pull complete
6ecee6444751: Pull complete
5b865d39f77d: Pull complete
e7e5c0273866: Pull complete
6a4effbc4451: Pull complete
4b6cb08bb4bc: Pull complete
7b07ad270e2c: Pull complete
f03f44139976: Pull complete
Creating go-micro_hystrix-dashboard_1 ... done
Creating go-micro_consul_1            ... done
Creating go-micro_go-micro-server_1   ... done
Creating go-micro_go-micro-client_1   ... done
Attaching to go-micro_hystrix-dashboard_1, go-micro_consul_1, go-micro_go-micro-server_1, go-micro_go-micro-client_1
consul_1             | ==> Starting Consul agent...
go-micro-server_1    | 2019/03/22 15:03:30 Transport [http] Listening on [::]:38487
consul_1             | ==> Consul agent running!
consul_1             |            Version: 'v0.8.3'
consul_1             |            Node ID: '3c5eeeff-e5d6-afae-a297-5058f80a1b93'
consul_1             |          Node name: 'dd2139f3b358'
consul_1             |         Datacenter: 'dc1'
consul_1             |             Server: true (bootstrap: false)
consul_1             |        Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 8600)
consul_1             |       Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
consul_1             |     Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
consul_1             |              Atlas: <disabled>
consul_1             | 
consul_1             | ==> Log data will now stream in as it occurs:
consul_1             | 
consul_1             |     2019/03/22 15:03:30 [DEBUG] Using unique ID "3c5eeeff-e5d6-afae-a297-5058f80a1b93" from host as node ID
consul_1             |     2019/03/22 15:03:30 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
consul_1             |     2019/03/22 15:03:30 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
consul_1             |     2019/03/22 15:03:30 [INFO] serf: EventMemberJoin: dd2139f3b358 127.0.0.1
consul_1             |     2019/03/22 15:03:30 [INFO] serf: EventMemberJoin: dd2139f3b358.dc1 127.0.0.1
consul_1             |     2019/03/22 15:03:30 [INFO] consul: Adding LAN server dd2139f3b358 (Addr: tcp/127.0.0.1:8300) (DC: dc1)
consul_1             |     2019/03/22 15:03:30 [INFO] consul: Handled member-join event for server "dd2139f3b358.dc1" in area "wan"
go-micro-server_1    | 2019/03/22 15:03:30 Broker [http] Connected to [::]:33229
consul_1             |     2019/03/22 15:03:30 [WARN] raft: Heartbeat timeout from "" reached, starting election
consul_1             |     2019/03/22 15:03:30 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
consul_1             |     2019/03/22 15:03:30 [DEBUG] raft: Votes needed: 1
consul_1             |     2019/03/22 15:03:30 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
consul_1             |     2019/03/22 15:03:30 [INFO] raft: Election won. Tally: 1
consul_1             |     2019/03/22 15:03:30 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
consul_1             |     2019/03/22 15:03:30 [INFO] consul: cluster leadership acquired
consul_1             |     2019/03/22 15:03:30 [DEBUG] consul: reset tombstone GC to index 3
go-micro-server_1    | 2019/03/22 15:03:30 Registry [mdns] Registering node: greeter-129b4606-8739-4df8-a61b-338b366cb270
consul_1             |     2019/03/22 15:03:30 [INFO] consul: member 'dd2139f3b358' joined, marking health alive
consul_1             |     2019/03/22 15:03:30 [INFO] consul: New leader elected: dd2139f3b358
consul_1             |     2019/03/22 15:03:30 [INFO] agent: Synced service 'consul'
consul_1             |     2019/03/22 15:03:30 [DEBUG] agent: Node info in sync
hystrix-dashboard_1  | 2019-03-22 15:03:31.773  INFO 1 --- [           main] c.l.HysterixDashboardApplication         : Starting HysterixDashboardApplication v0.0.1-SNAPSHOT on d83874cff53a with PID 1 (/hysterix-dashboard.jar started by root in /)
hystrix-dashboard_1  | 2019-03-22 15:03:31.791  INFO 1 --- [           main] c.l.HysterixDashboardApplication         : No active profile set, falling back to default profiles: default
hystrix-dashboard_1  | 2019-03-22 15:03:31.902  INFO 1 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@478267af: startup date [Fri Mar 22 15:03:31 UTC 2019]; root of context hierarchy
hystrix-dashboard_1  | 2019-03-22 15:03:32.518  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'configurationPropertiesRebinderAutoConfiguration' of type [class org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$658a8629] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
hystrix-dashboard_1  | 2019-03-22 15:03:32.895  INFO 1 --- [           main] c.l.HysterixDashboardApplication         : Started HysterixDashboardApplication in 2.001 seconds (JVM running for 3.425)
hystrix-dashboard_1  | 
hystrix-dashboard_1  |   .   ____          _            __ _ _
hystrix-dashboard_1  |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
hystrix-dashboard_1  | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
hystrix-dashboard_1  |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
hystrix-dashboard_1  |   '  |____| .__|_| |_|_| |_\__, | / / / /
hystrix-dashboard_1  |  =========|_|==============|___/=/_/_/_/
hystrix-dashboard_1  |  :: Spring Boot ::        (v1.3.3.RELEASE)
hystrix-dashboard_1  | 
hystrix-dashboard_1  | 2019-03-22 15:03:33.093  INFO 1 --- [           main] c.l.HysterixDashboardApplication         : No active profile set, falling back to default profiles: default
hystrix-dashboard_1  | 2019-03-22 15:03:33.118  INFO 1 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@51d67bb8: startup date [Fri Mar 22 15:03:33 UTC 2019]; parent: org.springframework.context.annotation.AnnotationConfigApplicationContext@478267af
hystrix-dashboard_1  | 2019-03-22 15:03:34.234  INFO 1 --- [           main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'beanNameViewResolver' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter; factoryMethodName=beanNameViewResolver; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]]
hystrix-dashboard_1  | 2019-03-22 15:03:34.443  INFO 1 --- [           main] o.s.cloud.context.scope.GenericScope     : BeanFactory id=90cce959-06d6-3884-bbec-bf3c66a78265
hystrix-dashboard_1  | 2019-03-22 15:03:34.474  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [class org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$658a8629] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:03:34.6488535 +0000 UTC m=+3.074486401
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:03:34.6488535 +0000 UTC m=+3.074486401
hystrix-dashboard_1  | 2019-03-22 15:03:34.982  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 9002 (http)
hystrix-dashboard_1  | 2019-03-22 15:03:35.009  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
hystrix-dashboard_1  | 2019-03-22 15:03:35.011  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.0.32
hystrix-dashboard_1  | 2019-03-22 15:03:35.165  INFO 1 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
hystrix-dashboard_1  | 2019-03-22 15:03:35.166  INFO 1 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2048 ms
hystrix-dashboard_1  | 2019-03-22 15:03:35.791  INFO 1 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'proxyStreamServlet' to [/proxy.stream]
hystrix-dashboard_1  | 2019-03-22 15:03:35.796  INFO 1 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
hystrix-dashboard_1  | 2019-03-22 15:03:35.803  INFO 1 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'characterEncodingFilter' to: [/*]
hystrix-dashboard_1  | 2019-03-22 15:03:35.804  INFO 1 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
hystrix-dashboard_1  | 2019-03-22 15:03:35.805  INFO 1 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'httpPutFormContentFilter' to: [/*]
hystrix-dashboard_1  | 2019-03-22 15:03:35.806  INFO 1 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'requestContextFilter' to: [/*]
hystrix-dashboard_1  | 2019-03-22 15:03:36.145  INFO 1 --- [           main] o.s.ui.freemarker.SpringTemplateLoader   : SpringTemplateLoader for FreeMarker: using resource loader [org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@51d67bb8: startup date [Fri Mar 22 15:03:33 UTC 2019]; parent: org.springframework.context.annotation.AnnotationConfigApplicationContext@478267af] and template loader path [classpath:/templates/]
hystrix-dashboard_1  | 2019-03-22 15:03:36.150  INFO 1 --- [           main] o.s.w.s.v.f.FreeMarkerConfigurer         : ClassTemplateLoader for Spring macros added to FreeMarker configuration
hystrix-dashboard_1  | 2019-03-22 15:03:36.780  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@51d67bb8: startup date [Fri Mar 22 15:03:33 UTC 2019]; parent: org.springframework.context.annotation.AnnotationConfigApplicationContext@478267af
hystrix-dashboard_1  | 2019-03-22 15:03:36.912  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/hystrix]}" onto public java.lang.String org.springframework.cloud.netflix.hystrix.dashboard.HystrixDashboardController.home(org.springframework.ui.Model,org.springframework.web.context.request.WebRequest)
hystrix-dashboard_1  | 2019-03-22 15:03:36.913  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/hystrix/{path}]}" onto public java.lang.String org.springframework.cloud.netflix.hystrix.dashboard.HystrixDashboardController.monitor(java.lang.String,org.springframework.ui.Model,org.springframework.web.context.request.WebRequest)
hystrix-dashboard_1  | 2019-03-22 15:03:36.915  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
hystrix-dashboard_1  | 2019-03-22 15:03:36.916  INFO 1 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
hystrix-dashboard_1  | 2019-03-22 15:03:36.982  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
hystrix-dashboard_1  | 2019-03-22 15:03:36.982  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
hystrix-dashboard_1  | 2019-03-22 15:03:37.042  INFO 1 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
hystrix-dashboard_1  | 2019-03-22 15:03:37.315  WARN 1 --- [           main] o.s.c.n.a.ArchaiusAutoConfiguration      : No spring.application.name found, defaulting to 'application'
hystrix-dashboard_1  | 2019-03-22 15:03:37.330  WARN 1 --- [           main] c.n.c.sources.URLConfigurationSource     : No URLs will be polled as dynamic configuration sources.
hystrix-dashboard_1  | 2019-03-22 15:03:37.330  INFO 1 --- [           main] c.n.c.sources.URLConfigurationSource     : To enable URLs as dynamic configuration sources, define System property archaius.configurationSource.additionalUrls or make config.properties available on classpath.
hystrix-dashboard_1  | 2019-03-22 15:03:37.363  WARN 1 --- [           main] c.n.c.sources.URLConfigurationSource     : No URLs will be polled as dynamic configuration sources.
hystrix-dashboard_1  | 2019-03-22 15:03:37.364  INFO 1 --- [           main] c.n.c.sources.URLConfigurationSource     : To enable URLs as dynamic configuration sources, define System property archaius.configurationSource.additionalUrls or make config.properties available on classpath.
hystrix-dashboard_1  | 2019-03-22 15:03:37.588  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
hystrix-dashboard_1  | 2019-03-22 15:03:37.649  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Bean with name 'configurationPropertiesRebinder' has been autodetected for JMX exposure
hystrix-dashboard_1  | 2019-03-22 15:03:37.652  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Bean with name 'refreshScope' has been autodetected for JMX exposure
hystrix-dashboard_1  | 2019-03-22 15:03:37.657  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Bean with name 'environmentManager' has been autodetected for JMX exposure
hystrix-dashboard_1  | 2019-03-22 15:03:37.674  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Located managed bean 'environmentManager': registering with JMX server as MBean [org.springframework.cloud.context.environment:name=environmentManager,type=EnvironmentManager]
hystrix-dashboard_1  | 2019-03-22 15:03:37.746  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Located managed bean 'refreshScope': registering with JMX server as MBean [org.springframework.cloud.context.scope.refresh:name=refreshScope,type=RefreshScope]
go-micro-server_1    | Responding with Hello Leander, calling at 2019-03-22 15:03:37.6489653 +0000 UTC m=+6.074599601
go-micro-client_1    | Hello Leander, calling at 2019-03-22 15:03:37.6489653 +0000 UTC m=+6.074599601
hystrix-dashboard_1  | 2019-03-22 15:03:37.791  INFO 1 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Located managed bean 'configurationPropertiesRebinder': registering with JMX server as MBean [org.springframework.cloud.context.properties:name=configurationPropertiesRebinder,context=51d67bb8,type=ConfigurationPropertiesRebinder]
hystrix-dashboard_1  | 2019-03-22 15:03:38.139  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 9002 (http)
hystrix-dashboard_1  | 2019-03-22 15:03:38.144  INFO 1 --- [           main] c.l.HysterixDashboardApplication         : Started HysterixDashboardApplication in 7.469 seconds (JVM running for 8.674)
```
we can see the client/server interaction.