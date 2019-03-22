# Implement Publish/Subscribe with Apache Kafka

We will be using the library [sarama](https://github.com/Shopify/sarama) a Go library for Apache Kafka.

## Producer

First we create the configuration for the producer
``` go
config := sarama.NewConfig()
config.Producer.Return.Successes = true
config.Producer.Return.Errors = true
config.Producer.RequiredAcks = sarama.WaitForAll
config.Producer.Retry.Max = 5
```

Next we create a sync producer for given address (from environment variable `BROKER_ADDR`) and configuration:
```go
brokers := []string{brokerAddr()}
producer, err := sarama.NewSyncProducer(brokers, config)
if err != nil {
    panic(err)
}

defer func() {
    if err := producer.Close(); err != nil {
        panic(err)
    }
}()
```

After this we can create a simple message:
```go
msg := &sarama.ProducerMessage{
    Topic: topic,
    Value: sarama.StringEncoder(fmt.Sprintf("Hello Kafka %v", msgCount)),
}
```
where `topic` is retrieved from the environment variable `TOPIC`.

And in a final step, we will use the producer for sending the message:
```go 
partition, offset, err := producer.SendMessage(msg)
if err != nil {
    panic(err)
}
```

## Consumer
The consumer or subscriber creates its own configuration:
```go 
config := sarama.NewConfig()
config.Consumer.Offsets.Initial = sarama.OffsetOldest
config.Consumer.Offsets.CommitInterval = 5 * time.Second
config.Consumer.Return.Errors = true
```

Next creates a new consumer for given brokers and configuration
```go 
brokers := []string{brokerAddr()}
master, err := sarama.NewConsumer(brokers, config)
if err != nil {
    panic(err)
}

defer func() {
    if err := master.Close(); err != nil {
        panic(err)
    }
}()
```

After this we will create a Kafka partition consumer for given topic:
```go
consumer, err := master.ConsumePartition(topic, 0, sarama.OffsetOldest)
if err != nil {
    panic(err)
}
```

And finally we consume and process the messages:
```go 
// Get signal for finish
doneCh := make(chan struct{})
go func() {
    for {
        select {
        case err := <-consumer.Errors():
            fmt.Println(err)
        case msg := <-consumer.Messages():
            msgCount++
            fmt.Println("Received messages", string(msg.Key), string(msg.Value))
        case <-signals:
            fmt.Println("Interrupt is detected")
            doneCh <- struct{}{}
        }
    }
}()

<-doneCh
```

## Docker compose
We need to add the docker images of Zookeeper and Kafka from `dockerkafka`.

## Run
To fire the producer, subscriber, zookeeper and Kafka:
```bash
$ docker-compose up
```