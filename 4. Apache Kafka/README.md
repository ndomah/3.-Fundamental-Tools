# Apache Kafka Fundamentals
## What is Kafka?

Kafka is a **distributed message queue** that enables event/stream processing, buffering messages, and decoupling ingestion from processing/storage. It acts as a temporary store where messages have a TTL (Time-To-Live).

### Basic Kafka Components
- **Topics**: Channels where messages are written and read.
- **Partitions**: Subdivisions of topics for scalability.
- **In-Sync Replicas (ISR)**: Ensure fault tolerance.
- **Messages**: Data payloads that producers send and consumers read.
- **Brokers**: Kafka servers that handle message distribution.
- **Producers**: Publish messages to topics.
- **Consumers & Consumer Groups**: Read messages from topics.

## Kafka & Message Queue Basics

![message queue basics](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/img/message%20queue%20basics.png)

Kafka provides a **distributed, fault-tolerant message** queue, enabling event-driven architectures. Messages are serialized and stored in Kafka topics before being consumed.

## Apache Kafka Components
### Topics, Partitions, & Brokers

![topics, partitions, brokers](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/img/topics%20partitions%20and%20brokers.png)

Kafka topics consist of multiple **partitions** spread across **brokers** for scalability and parallel processing. ISR (In-Sync Replicas) ensure message durability.

### Brokers & Zookeeper

![brokers and zookeeper](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/img/brokers%20and%20zookeeper.png)

Kafka brokers handle message storage and retrieval, while **Zookeeper** manages:
- Broker states & quotas
- Topic configurations
- Access control lists
- Cluster membership
- Controller election
- Consumer offsets and registry

## Development Environment
### Setting up Kafka with Docker
We use **Bitnami Kafka** images to set up Kafka and Zookeeper:

[`docker-compose.yml`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/scripts/docker-compose.yml):
```yml
version: "3"
services:
  zookeeper:
    image: 'bitnami/zookeeper:3.7.0-debian-10-r70'
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:2.8.0-debian-10-r42'
    ports:
      - '9092:9092'
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zookeeper
```
To start Kafka and Zookeeper:
```
docker-compose up -d
```

## Kafka Commands
### Working with Topics
[`commands.md`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/scripts/commands.md):
```
# Create a new topic
./kafka-topics.sh --create --topic mytesttopic --bootstrap-server localhost:9092

# List all topics
./kafka-topics.sh --list --bootstrap-server localhost:9092

# Describe a topic
./kafka-topics.sh --describe --topic mytesttopic --bootstrap-server localhost:9092

# Consume messages from a topic
./kafka-console-consumer.sh --topic mytesttopic --bootstrap-server localhost:9092

# Check consumer offset
./kafka-consumer-groups.sh --bootstrap-server localhost:9092  --describe --group mypythonconsumer
```

## Python Producer & Consumer
### Python Producer
[`producer.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/scripts/producer.py):
```python
from kafka import KafkaProducer

# Create Message
msg = 'Hello this is a test message'

# Create a producer
producer = KafkaProducer(bootstrap_servers='localhost:9092')

def kafka_python_producer_async(producer, msg):
    producer.send('mytesttopic', msg).add_callback(success).add_errback(error)
    producer.flush()

def success(metadata):
    print(metadata.topic)

def error(exception):
    print(exception)  

print("start producing")
kafka_python_producer_async(producer, bytes(msg, 'utf-8'))
print("done")
```
### Python Consumer
[`consumer.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/scripts/consumer.py):
```python
from kafka import KafkaConsumer

def kafka_python_consumer():
    consumer = KafkaConsumer('mytesttopic', group_id='mypythonconsumer', bootstrap_servers='localhost:9092')
    for msg in consumer:
        print(msg)

print("start consuming")
kafka_python_consumer()
print("done")
```

## Kafka in Data Platforms
### Example: How Kafka fits in Data Platforms

![ingestion pipeline](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/img/ingestion%20pipeline.png)

This pipeline is commonly used in **microservices architectures**, **event-driven systems**, and **data streaming applications**.

- The **client** sends requests, which get processed by different API instances.
- These APIs publish messages to **Kafka Brokers**, which act as the messaging backbone.
- Kafka efficiently handles event-driven communication, making it scalable and reliable for real-time data processing.

### Multiple Processing as Consumers

![multiple processing as consumers](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/img/multiple%20processing%20as%20consumers.png)

Kafka allows multiple consumers to process messages independently, ensuring scalable and fault-tolerant data processing.

### Multistage Stream Processing

![multistage stream processing](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/4.%20Apache%20Kafka/img/multistage%20stream%20processing.png)

Messages pass through multiple processing stages before reaching their destination, ensuring efficient event-driven architectures.
