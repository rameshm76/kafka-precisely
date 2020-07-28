# Apache Kafka

## Kafka Components

- Zookeeper
- Kafka Brokers
- Topics
- Producers
- Consumers
  - Consumer Groups

## Zookeeper

### Zookeeper Configuration

Zookeeper configuration is located at: **kafka/config/zookeeper.properties**

Take a note of clientPort at which the clients will connect:

```properties
clientPort=2181
```

### Starting zookeeper

```sh
kafka/bin/zookeeper-server-start.sh kafka/config/zookeeper.properties
```

## Kafka Servers

### Kafka Server Configuration

Kafka server configuration is located at: **kafka/config/server.properties**

Take a note of the following properties. The zookeeper.connect should match zookeeper clientPort

```properties
broker.id=0
listeners=PLAINTEXT://localhost:9092
zookeeper.connect=localhost:2181
```

### Starting Kafka Server

```sh
kafka/bin/kafka-server-start.sh kafka/config/server.properties
```

## Topics

### Creating a topic

Create a non-replicated topic. And list the topics available. The replication-factor must be <= brokers available at the time of creation

```sh
kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181  --replication-factor 1 --partitions 4 --topic alphabet
```

### Listing the topics

List the topics available

```sh
kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

### Describing topics

The following command describes all the available topics

```sh
kafka/bin/kafka-topics.sh --describe --zookeeper localhost:2181
```

The following command describes **alphabet** topic.

```sh
kafka/bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic alphabet
```

## Producers

### Start a producer

Run the following command to produce to *alphabet*. Start entering a word per line.

```sh
kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic alphabet
```

## Consumers

### Start a consumer

Run the following command to consume from the *alphabet*. As the messages are not published to any
specific partition, the order of the words you consumed could be different

```sh
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic alphabet --from-beginning
```

### Consumer OffSets Topic

Kafka will create **__consumer_offsets** topic to keep track of the offsets for each client. If we
donot pass the **--from-beginning** the consumer will receive only the latest messages publushed
after the consumer is available.

```sh
kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

There are three variants of offsets.

- from-beginning
- latest
- from a specific offset
  - Only possible using api calls

### Consumer Groups

Whenever a new consumer is created using console, a new consumer group will be created. If two
clients use the same group.id, then they concurrently process the messages published.

One can view list of consumer groups using the following command.

```sh
kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

To view all consumer groups and, their offsets, run the below command.

```sh
kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe  --all-groups
```

To view a specific consumer group(for example: **console-consumer-7160**) offset run the following
command using the correct consumer group id

```sh
kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group console-consumer-71600
```

### Consumer explicit Group

A group of consumers can use the same group id to consume messages collectively.

Stop the existing producers and consumers and, run the following. The messages produced will be
consumed by both the consumers. A single message wont be consumed by both consumers.

```sh
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic alphabet --group test
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic alphabet --group test
kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic alphabet
```

## Message Ordering

Messages are sent usually as key & value pairs. In the previous example, we didn't  provide the
optional key. Kafka will partion the messages if a key is provided.

### Producing message with key

```sh
kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic alphabet --property "key.separator=-" --property "parse.key=true"
```

Enter messages as below:

```sh
Vowel-A
Vowel-E
Consonant-B
Vowel-I
Vowel-O
Vowel-U
Consonant-C
Consonant-D
```

### Consuming using key

One should notice that, the messages maintains the ordering under the same key. The **print.key=true**
allows logging the key on to the logs/console.

```sh
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic alphabet --from-beginning -property "key.separator= - " --property "print.key=true"
```

## Clean up - I

Run the following to kill all components

```sh
kill $(lsof -ti:2181,9092)
rm -rf /tmp/{kafka-logs,zookeeper}
```

## Kafka Clustering

Using a single kafka server is a single point of failure. Failure of kafka server won't allow
producers from producing and, consumers from consuming from topics. To mitigate this issue, we should
add more kafka servers(brokers) to form a cluster.

Stop zookeeper, kafka server, producers & consumers. Run the following command in the order they are specified.

### Add 3 Kafka Server Configurations

```sh
for x in {0..2}; do cp kafka/config/server.properties "kafka/config/server-$x.properties"; done
```

#### Update kafka/config/sever-0.properties

```properties
broker.id=0
listeners=PLAINTEXT://localhost:9092
log.dirs=/tmp/kafka-logs-0
```

#### Update kafka/config/sever-1.properties

```properties
broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs=/tmp/kafka-logs-1
```

#### Update kafka/config/sever-2.properties

```properties
broker.id=2
listeners=PLAINTEXT://localhost:9094
log.dirs=/tmp/kafka-logs-2
```

### Start Zookeeper

```sh
kafka/bin/zookeeper-server-start.sh kafka/config/zookeeper.properties
```

### Start Kafka Servers

```sh
kafka/bin/kafka-server-start.sh kafka/config/server-0.properties
kafka/bin/kafka-server-start.sh kafka/config/server-1.properties
kafka/bin/kafka-server-start.sh kafka/config/server-2.properties
```

### Create replicated kafka topic

```sh
kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic my-topic-replicated
kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181
kafka/bin/kafka-topics.sh --describe --topic my-topic-replicated --zookeeper localhost:2181
```

### Start producer to produce to replicated topic

Unlike other commands, we are using multiple brokers in the broker-list (though it is not mandatory).

```sh
kafka/bin/kafka-console-producer.sh --broker-list localhost:9093,localhost:9094 --topic my-topic-replicated
```

### Start consumer to consume from replicated topic

Unlike other commands, we are using multiple brokers in the broker-list (though it is not mandatory).

```sh
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9093,localhost:9094 --topic my-topic-replicated --from-beginning
```

The produced messages will be processed by consumer.

### Consumer Groups replication topic

Start two more instances of consumers using the following commands

```sh
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic my-topic-replicated --group test
kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9093,localhost:9094 --topic my-topic-replicated --group test
```

When the producer produces messages, both the consumers will share the load.

One can view list of available consumer groups using below command.

```sh
kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9094 --list --all-groups
```

One can view information about a specific consumer group as below.

```sh
kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9094 --describe --group test
```

## InSync Replication(ISR)

Kafka cluster ensures that, the produced messages will be availble to all the consumers on an event
of a kafka server unavailability due to a failure or maintence. Kafka will elect a new partition
leader from the available kafka servers. However, there is a possibility some messages could be lost.
To prevent this we need to configure **min.insync.replica** property on the broker.

> replication-factor >= min.insync.replica > 1

The above setting will help to enable a message is delivered to more than one broker and there by prevents
loss of messages. The producer also must use ack=all to get acknowledgment that brokers  received the messages

```sh
kafka/bin/kafka-console-producer.sh --broker-list localhost:9093,localhost:9094 --topic my-topic-replicated  --request-required-acks all
```

On a failure of kafka node, the insyn replicas(isr) wil rebalance

## Cleanup - II

Run the following to kill all components

```sh
kill $(lsof -ti:2181,9093,9094,9095)
rm -rf /tmp/{kafka-logs-{0,1,2},zookeeper}
```

## References

[Kafka DefaultPartitioner algorithm](https://stackoverflow.com/questions/39791349/kafka-defaultpartitioner-algorithm)

[In Sync Replica - Throuhtput](https://stackoverflow.com/questions/52215953/does-min-insync-replica-configuration-affect-kafka-producer-throughput)
