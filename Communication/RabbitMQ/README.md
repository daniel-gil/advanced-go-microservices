# Implement Message Queuing with RabbitMQ

We will use the [Go RabbitMQ Client Library](https://github.com/streadway/amqp).

## Producer
First dial a connection to the broker, getting the broker address from the environment variable `BROKER_ADDR`:
```go
conn, err := amqp.Dial(brokerAddr())
failOnError(err, "Failed to connect to RabbitMQ")
defer conn.Close()
```    

Then we open a channel for communication:
```go
ch, err := conn.Channel()
failOnError(err, "Failed to open a channel")
defer ch.Close()
```

Next step we declare the queue:
```go
q, err := ch.QueueDeclare(
    queue(), // name
    true,    // durable
    false,   // delete when unused
    false,   // exclusive
    false,   // no-wait
    nil,     // arguments
)
failOnError(err, "Failed to declare a queue")
```    
where the queue name is configurable by the environment variable `QUEUE`.

Once we have the queue, we can start publishing messages to it.
```go
body := fmt.Sprintf("Hello RabbitMQ message %v", msgCount)

err = ch.Publish(
    "",     // exchange
    q.Name, // routing key
    false,  // mandatory
    false,  // immediate
    amqp.Publishing{
        ContentType: "text/plain",
        Body:        []byte(body),
    })

log.Printf(" [x] Sent %s", body)
failOnError(err, "Failed to publish a message")
```

## Consumer

The consumer will use the same amqp library as the producer. 

The following 3 steps are the same as in the producer: 
- Dial the connection to the broker
- Open communication channel
- Declare the queue

The next step is different, we need to register and get a message consumer
```go 
msgs, err := ch.Consume(
    q.Name, // queue
    "",     // consumer
    true,   // auto-ack
    false,  // exclusive
    false,  // no-local
    false,  // no-wait
    nil,    // args
)
failOnError(err, "Failed to register a consumer")
```

Consumption processing logic, simply read the message body from the `msgs` channel:
```go
go func() {
    for d := range msgs {
        log.Printf("Received a message: %s", d.Body)
    }
}()
```

## Build docker images
```bash
zeus:RabbitMQ danielgil$ docker-compose build
rabbitmq uses an image, skipping
Building rabbitmq-producer
Step 1/9 : FROM golang:1.8.1-alpine
 ---> 32efc118745e
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Using cache
 ---> 513a31355d6c
Step 3/9 : RUN go get github.com/streadway/amqp
 ---> Using cache
 ---> 5325b3afc763
Step 4/9 : ENV SOURCES /go/src/github.com/PacktPublishing/Advanced-Cloud-Native-Go/Communication/RabbitMQ/
 ---> Using cache
 ---> 60759c3cf34d
Step 5/9 : COPY . ${SOURCES}
 ---> 5b0b479e5978
Step 6/9 : RUN cd ${SOURCES}producer/ && CGO_ENABLED=0 go build
 ---> Running in 928f403eb3ce
Removing intermediate container 928f403eb3ce
 ---> 9fbcbf610650
Step 7/9 : ENV BROKER_ADDR amqp://guest:guest@localhost:5672/
 ---> Running in c39e9d95a1f2
Removing intermediate container c39e9d95a1f2
 ---> 46a51ec93a1a
Step 8/9 : WORKDIR ${SOURCES}producer/
 ---> Running in 2110f140d3b6
Removing intermediate container 2110f140d3b6
 ---> e3e5da40c2fc
Step 9/9 : CMD ${SOURCES}producer/producer
 ---> Running in db7724e9a831
Removing intermediate container db7724e9a831
 ---> 8eda880b9764
Successfully built 8eda880b9764
Successfully tagged rabbitmq-producer:1.0.1
Building rabbitmq-consumer
Step 1/9 : FROM golang:1.8.1-alpine
 ---> 32efc118745e
Step 2/9 : RUN apk update && apk upgrade && apk add --no-cache bash git
 ---> Using cache
 ---> 513a31355d6c
Step 3/9 : RUN go get github.com/streadway/amqp
 ---> Using cache
 ---> 5325b3afc763
Step 4/9 : ENV SOURCES /go/src/github.com/PacktPublishing/Advanced-Cloud-Native-Go/Communication/RabbitMQ/
 ---> Using cache
 ---> 60759c3cf34d
Step 5/9 : COPY . ${SOURCES}
 ---> Using cache
 ---> 5b0b479e5978
Step 6/9 : RUN cd ${SOURCES}consumer/ && CGO_ENABLED=0 go build
 ---> Running in 6c4279ffdde5
Removing intermediate container 6c4279ffdde5
 ---> b6478c3e731e
Step 7/9 : ENV BROKER_ADDR amqp://guest:guest@localhost:5672/
 ---> Running in 30f33f35b976
Removing intermediate container 30f33f35b976
 ---> 86e97bd787c6
Step 8/9 : WORKDIR ${SOURCES}consumer/
 ---> Running in 37fcaf04af51
Removing intermediate container 37fcaf04af51
 ---> 351e869440f2
Step 9/9 : CMD ${SOURCES}consumer/consumer
 ---> Running in 9198b50cf1f3
Removing intermediate container 9198b50cf1f3
 ---> c6f2e0cbddd7
Successfully built c6f2e0cbddd7
Successfully tagged rabbitmq-consumer:1.0.1
zeus:RabbitMQ danielgil$ 
```

## Run

``` bash
$ docker-compose up
Starting rabbitmq_rabbitmq_1 ... done
Recreating rabbitmq_rabbitmq-producer_1 ... done
Recreating rabbitmq_rabbitmq-consumer_1 ... done
Attaching to rabbitmq_rabbitmq_1, rabbitmq_rabbitmq-producer_1, rabbitmq_rabbitmq-consumer_1
rabbitmq-producer_1  | Starting RabbitMQ producer...
rabbitmq-consumer_1  | Starting RabbitMQ consumer...
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:55 ===
rabbitmq_1           | Starting RabbitMQ 3.6.9 on Erlang 19.1
rabbitmq_1           | Copyright (C) 2007-2016 Pivotal Software, Inc.
rabbitmq_1           | Licensed under the MPL.  See http://www.rabbitmq.com/
rabbitmq_1           | 
rabbitmq_1           |               RabbitMQ 3.6.9. Copyright (C) 2007-2016 Pivotal Software, Inc.
rabbitmq_1           |   ##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
rabbitmq_1           |   ##  ##
rabbitmq_1           |   ##########  Logs: tty
rabbitmq_1           |   ######  ##        tty
rabbitmq_1           |   ##########
rabbitmq_1           |               Starting broker...
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:55 ===
rabbitmq_1           | node           : rabbit@38ad328b1a5e
rabbitmq_1           | home dir       : /var/lib/rabbitmq
rabbitmq_1           | config file(s) : /etc/rabbitmq/rabbitmq.config
rabbitmq_1           | cookie hash    : MY4Ae1MS5LYaz68Hjmx/Hg==
rabbitmq_1           | log            : tty
rabbitmq_1           | sasl log       : tty
rabbitmq_1           | database dir   : /var/lib/rabbitmq/mnesia/rabbit@38ad328b1a5e
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Memory limit set to 799MB of 1998MB total.
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Disk free limit set to 50MB
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Limiting to approx 1048476 file handles (943626 sockets)
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | FHC read buffering:  OFF
rabbitmq_1           | FHC write buffering: ON
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Waiting for Mnesia tables for 30000 ms, 9 retries left
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Waiting for Mnesia tables for 30000 ms, 9 retries left
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Priority queues enabled, real BQ is rabbit_variable_queue
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Starting rabbit_node_monitor
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Management plugin: using rates mode 'basic'
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | msg_store_transient: using rabbit_msg_store_ets_index to provide index
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | msg_store_persistent: using rabbit_msg_store_ets_index to provide index
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | started TCP Listener on [::]:5672
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Management plugin started. Port: 15672
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Statistics database started.
rabbitmq_1           |  completed with 6 plugins.
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:58 ===
rabbitmq_1           | Server startup complete; 6 plugins started.
rabbitmq_1           |  * rabbitmq_management
rabbitmq_1           |  * rabbitmq_web_dispatch
rabbitmq_1           |  * rabbitmq_management_agent
rabbitmq_1           |  * amqp_client
rabbitmq_1           |  * cowboy
rabbitmq_1           |  * cowlib
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:59 ===
rabbitmq_1           | accepting AMQP connection <0.503.0> (172.22.0.4:41476 -> 172.22.0.2:5672)
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:59 ===
rabbitmq_1           | connection <0.503.0> (172.22.0.4:41476 -> 172.22.0.2:5672): user 'guest' authenticated and granted access to vhost '/'
rabbitmq-producer_1  | 2019/03/22 16:08:59  [x] Sent Hello RabbitMQ message 1
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:59 ===
rabbitmq_1           | accepting AMQP connection <0.516.0> (172.22.0.3:57582 -> 172.22.0.2:5672)
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:08:59 ===
rabbitmq_1           | connection <0.516.0> (172.22.0.3:57582 -> 172.22.0.2:5672): user 'guest' authenticated and granted access to vhost '/'
rabbitmq-consumer_1  | 2019/03/22 16:08:59  [*] Waiting for messages. To exit press CTRL+C
rabbitmq-consumer_1  | 2019/03/22 16:08:59 Received a message: Hello RabbitMQ message 1
rabbitmq-producer_1  | 2019/03/22 16:09:04  [x] Sent Hello RabbitMQ message 2
rabbitmq-consumer_1  | 2019/03/22 16:09:04 Received a message: Hello RabbitMQ message 2
rabbitmq-producer_1  | 2019/03/22 16:09:09  [x] Sent Hello RabbitMQ message 3
rabbitmq-consumer_1  | 2019/03/22 16:09:09 Received a message: Hello RabbitMQ message 3
rabbitmq-producer_1  | 2019/03/22 16:09:14  [x] Sent Hello RabbitMQ message 4
rabbitmq-consumer_1  | 2019/03/22 16:09:14 Received a message: Hello RabbitMQ message 4
```

Check that we have the producer, consumer and RabbitMQ running:
```bash
$ docker ps
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                                                                                                                      NAMES
62a6d0e10b69        rabbitmq-producer:1.0.1            "/bin/sh -c ${SOURCE…"   2 minutes ago       Up About a minute                                                                                                                              rabbitmq_rabbitmq-producer_1
db5f01a14895        rabbitmq-consumer:1.0.1            "/bin/sh -c ${SOURCE…"   2 minutes ago       Up About a minute                                                                                                                              rabbitmq_rabbitmq-consumer_1
38ad328b1a5e        rabbitmq:3.6.9-management-alpine   "docker-entrypoint.s…"   4 minutes ago       Up About a minute   0.0.0.0:4369->4369/tcp, 0.0.0.0:5671-5672->5671-5672/tcp, 0.0.0.0:15671-15672->15671-15672/tcp, 0.0.0.0:25672->25672/tcp   rabbitmq_rabbitmq_1
```


## RabbitMQ Management Console
We can access the management console under this URL:
```
http://localhost:15672/
```
where 
```
Username: guest
Password: guest
```

Now we could stop the producer with
```
$ docker stop rabbitmq_rabbitmq-producer_1
rabbitmq_rabbitmq-producer_1
```

and RabbitMQ will detect it:
```
rabbitmq_1           | 
rabbitmq_1           | =WARNING REPORT==== 22-Mar-2019::16:17:20 ===
rabbitmq_1           | closing AMQP connection <0.470.0> (172.22.0.4:41482 -> 172.22.0.2:5672):
rabbitmq_1           | client unexpectedly closed TCP connection
rabbitmq_rabbitmq-producer_1 exited with code 137
```

If instead of stopping the producer, we stop the consumer:
``` bash
$ docker stop rabbitmq_rabbitmq-consumer_1
rabbitmq_rabbitmq-consumer_1
````

the messages are queued but not consumed 
```
rabbitmq_1           | 
rabbitmq_1           | =WARNING REPORT==== 22-Mar-2019::16:19:41 ===
rabbitmq_1           | closing AMQP connection <0.476.0> (172.22.0.3:57640 -> 172.22.0.2:5672):
rabbitmq_1           | client unexpectedly closed TCP connection
rabbitmq_rabbitmq-consumer_1 exited with code 137
rabbitmq-producer_1  | 2019/03/22 16:19:46  [x] Sent Hello RabbitMQ message 5
rabbitmq-producer_1  | 2019/03/22 16:19:51  [x] Sent Hello RabbitMQ message 6
rabbitmq-producer_1  | 2019/03/22 16:19:56  [x] Sent Hello RabbitMQ message 7
rabbitmq-producer_1  | 2019/03/22 16:20:01  [x] Sent Hello RabbitMQ message 8
rabbitmq-producer_1  | 2019/03/22 16:20:06  [x] Sent Hello RabbitMQ message 9
rabbitmq-producer_1  | 2019/03/22 16:20:11  [x] Sent Hello RabbitMQ message 10
rabbitmq-producer_1  | 2019/03/22 16:20:16  [x] Sent Hello RabbitMQ message 11
rabbitmq-producer_1  | 2019/03/22 16:20:20  [x] Sent Hello RabbitMQ message 12
```

If after awhile we start again the consumer we will see something like this:
```
rabbitmq-producer_1  | 2019/03/22 16:25:35  [x] Sent Hello RabbitMQ message 75
rabbitmq-producer_1  | 2019/03/22 16:25:40  [x] Sent Hello RabbitMQ message 76
rabbitmq-producer_1  | 2019/03/22 16:25:45  [x] Sent Hello RabbitMQ message 77
rabbitmq-producer_1  | 2019/03/22 16:25:50  [x] Sent Hello RabbitMQ message 78
rabbitmq-producer_1  | 2019/03/22 16:25:55  [x] Sent Hello RabbitMQ message 79
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:25:56 ===
rabbitmq_1           | accepting AMQP connection <0.515.0> (172.22.0.3:57658 -> 172.22.0.2:5672)
rabbitmq_1           | 
rabbitmq_1           | =INFO REPORT==== 22-Mar-2019::16:25:56 ===
rabbitmq_1           | connection <0.515.0> (172.22.0.3:57658 -> 172.22.0.2:5672): user 'guest' authenticated and granted access to vhost '/'
rabbitmq-consumer_1  | 2019/03/22 16:25:56  [*] Waiting for messages. To exit press CTRL+C
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 5
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 6
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 7
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 8
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 9
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 10
rabbitmq-consumer_1  | 2019/03/22 16:25:56 Received a message: Hello RabbitMQ message 11
```