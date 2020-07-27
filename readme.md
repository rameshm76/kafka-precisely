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

## References

[Kafka DefaultPartitioner algorithm](https://stackoverflow.com/questions/39791349/kafka-defaultpartitioner-algorithm)

## Cleanup

Run the following to kill all components

```sh
kill $(lsof -ti:2181,9092)
rm -rf /tmp/{kafka-logs,zookeeper}
```
