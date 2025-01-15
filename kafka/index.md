# Kafka

## Background

Kafka is a highly available, reliable, and durable message broker. Very useful to build a high volume messaging system.

## Key Concepts

### Broker

Broker is a single kafka node/server, it is called broker because it is what essentially Kafka is - a message broker.

### Kafka Cluster

Collection of kafka server/node. 

### Topic

Messages in Kafka is sent into a topic. Topic is defined using an string id. Message producer send message into a topic, and message consumer could subcribe into a topic to receive message.

### Message

Message is the entity that get sent to Kafka cluster and stored in the topic partition.

A message consist of `key` and `value` and they are in bytes format. The `key` within a message is optional, it is useful for message ordering.

You can choose whatever serialisation method you want, e.g. protobuff, arvo, or a simple JSON serialisation for Kafka message.

### Message Immutability

Messages sent into Kafka topic is immutable, meaning it can't be altered once it is being sent. You can configure the time-to-live (TTL) of the messages though, the default TTL of the message is a week.

### Offset

Offset is an integer that signify the position of messages within a topic on a particular consumer. This is useful for the consumer so keep track of the messages it already reads in order to continue consuming the message.

With the offset mechanism, it allows:

- Replay messages consumption from the beginning by resetting the offset
- When a consumer interrupted/restarted, it knows how to resume consuming the data without data loss

Offset is unique in each topic partition, meaning they are not shared across topic partition.

Offset is tracked within a consumer group. The offset is stored within an internal Kafka topic.

Note:

Consuming a topic with a new consumer can be computationally heavy as it'll need to read the topic from the beginning.

#### Topic partition

Topic can be split into multiple partition. Think about partition as separate "bucket" of the topic. 

For example, a topic of "GPS position" may be configured to have 3 partitions.

Each message retrieved in a partition will be assigned an increment ID from 0 and so on.

Having multiple partitions allow you to split the read among multiple consumers. This is useful for horizontally scaling the application.

Worth to note that messages sent from producer will be spread across the partition. 

You can configure the strategy on how spread the messages across the partition, the most naive strategy is round robin for example.

? Do partition has to be spread across different broker ?

#### Topic replication

You can configure Kafka topic to have replicas. This allows high level of reliability due to the redundancy of the data. 

Kafka has its own smart mechanism of the leader election that allows it to re-elect the replication leader when it loses the leader.

? Do replicas are spread across different broker ?

? Does the replica also used for read operation or do they only acts as backup?

### Message ordering

Message ordering is guaranteed only within a topic partition, since the messages will be spread across multiple partitions by the producer.

In order to guarantee ordering for messages consumption, you can assign a message with a key.

Kafka uses murmur hashing algorithm to determine which topic a message will be sent to, according its key. If no key is set, it will be just be distributed across partitions.

Using key, a consumer then can consume messages with guaranteed order since they all will be sent to the same topic.

? How do we handle migration when we need to add new partition into a topic while our system relies on a message ordering ? Would the hashing algorithm will resulting the same topic when the size of the partition grows ?

### Producer

Producer is the component that send message to Kafka topic.

#### Message Durability

You can configure the ack strategy in the producer side:
- ack all: acked when message is received by partition leader and replicated across all replicas
- ack 1: acked when message is received only by partition leader. While it allows some degree of data durability, this strategy still exposes the risk of data loss when the leader happened to be down before the replication completed
- ack 0: no ack from the partition leader aka optimistic update. Allowing maximum throughput but there's no guarantee the message is stored in the replica

### Consumer

Consumer is a component that sends message to Kafka topic.

### Consumer Group

To consume Kafka topic, you need to consume it via a consumer within a consumer group.

A consumer group can consists of multiple consumers, each will read from mutually exlusive partitions.

Example:

Consumer group has 2 consumer A and B. They read Topic X that has 3 partition X-1, X-2, and X-3. Consumer A reads from X-1, Consumer B reads from X-2 and X-3.

? What about replicas ? Do they read from the replicas as well ?

### Consumer Group Offset Commit Strategies

#### Auto Commit

Scheduled auto commit, e.g. every 5 seconds

Pro: Simplicity
Cons: Risk of duplicates when consumer crashes before next auto commit
Best for: High throughput system that can handle duplicates message processing

#### Sync Manual Commit 

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }

    // call api here

    consumer.commitSync(); // Blocks until commit succeeds or fails
}
```

Pro: Allows full control
Cons: Blocking, performance penalty
Best for: Cases where order is critical and system can't allow duplicates processing

#### Async Manual Commit

Pro: Allows control with higher throughput
Cons: Weaker guarantee than async method
Best for high throughput system that can handle occassional duplicates

# Deployment

## Bare machine deployment

## Kubernetes

## Putting it into practice

### Case study: Food delivery service

### Kafka Producer in Java

### Kafka Consumer in Java
