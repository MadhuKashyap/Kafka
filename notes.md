### What is Kafka?
Kafka is a distributed publish-subscribe messaging system.Data is written to Kafka topics by producers and consumed from those topics by consumers.
### Why do we need Kafka ?
<img width="601" alt="image" src="https://github.com/MadhuKashyap/Kafka/assets/40714383/e0bdf5ee-fe67-4e78-a5e1-9a7fe8be8650">

Suppose we have these applications which need to communicate with each other. One bottlneck is maintaining multiple connections at all times. If 1 of the application goes down, all data coming to that application will be lost. Here Kafka comes into picture

<img width="1251" alt="image" src="https://github.com/MadhuKashyap/Kafka/assets/40714383/d4e8fd80-3f0c-466d-9eb2-a8dbd3f5a134">

All the publishing applications will publish data and receiving applications will consume the data whenever they are avaialable, this way we maintain relatively lesser number of connections between applications

### Important members of kafka

#### Broker : 
A Kafka broker is a single Kafka server. Its job is to:
- Receive messages from producers.
- Store them on disk (in partitions).
- Serve consumers who want to read the messages.
Each broker can handle hundreds or thousands of topics and millions of messages per second, depending on hardware and configuration.

Example:
If you have a Kafka system with 3 servers running, each running Kafka, then you have 3 brokers.

<img width="1279" alt="image" src="https://github.com/user-attachments/assets/657683b4-7932-4a8c-b962-bafb50611226" />

#### Cluster : 
- A Kafka cluster is a group of one or more Kafka brokers that work together.
- Brokers in a cluster share data and workload.
- Topics and their partitions are distributed across brokers.
- One broker acts as the MASTER, managing partition leadership and cluster metadata.
- Kafka uses ZooKeeper (or KRaft in newer versions) to manage broker coordination.
- The cluster ensures high availability, fault tolerance, and scalability.
- Each cluster has it's own metadata. This metadata contains info like 
  
Suppose producer is producing huge volume of data, then a single kafka broker may not be able to handle the load. In that case               we need to add multiple brokers who can consume requests parallely to levarage scalability.

<img width="1204" alt="image" src="https://github.com/user-attachments/assets/b219f408-b4db-4f8a-bb0e-46387832d00b" />

### Cluster Metadata : 
It refers to Kafka cluster’s current state and configuration. Cluster metadata contains :

- A unique identifier for the Kafka cluster.
- Broker List: IDs of all the Kafka brokers in the cluster.
- List of all topics available in the cluster.
- Number of partitions for each topic.
- The broker ID that is the leader for each partition. Clients always send writes or reads to the partition leader.
- The broker acting as the Kafka controller manages cluster metadata updates, partition assignments, and leader elections.

### Controller Quorum
- A controller quorum is set of nodes (called controller voters) that collectively manage cluster metadata.
- Multiple brokers together form a quorum. It is made up of multiple brokers to provide high availability and fault tolerance.
- However, at any given time, exactly one of these controllers acts as the “active leader controller”
- The other controllers in the quorum are standby controllers waiting to take over if the leader fails.

#### Producer : 
A Kafka producer is a client that sends (publishes) data to Kafka topics.
Producers are responsible for:
- Choosing the topic to send the message to.
- Optionally choosing the partition (or Kafka decides automatically).
- Serializing the message.
- Sending the message over the network to a Kafka broker.

```
producer.send(new ProducerRecord<>(<topic-name>, <partition-key>, <message>));
```
- If the partition-key is provided, Kafka will hash the key and map it to a partition. If no partition-key is provided, Kafka does a round-robin distribution across partitions.
- A topic can have multiple partitions, and each partition is hosted on a different broker. So messages published to a topic might go to different brokers.
- Before writing data to a topic, producer requests metadata about the cluster from a broker. The metadata tells on which broker is the partition leader residing and the producer always writes on the leader


🧠 Summary
- What is a producer?	A client that sends messages to Kafka topics
- Where does it publish?	To a topic, then routed to a partition on a broker
- Always to same broker?	❌ No — it depends on the partition's location
- Always to same topic?	❌ No — the producer can choose different topics
- Always to same partition?	✅ If the key is fixed
                            ❌ If no key is used

#### Consumer : 
- A Kafka consumer is a client that reads messages from one or more Kafka topics.
- Every message in a Kafka partition has a unique offset (like a position/index). The consumer keeps track of which offset it has read so it can resume where it left off.
- Kafka ensures each partition is only read by one consumer in the group.
- If we have 3 partitions and 2 consumers, 1 will sit idle until 1 more partition is added.
- Consumers can subscribe to multiple topics at a time.

```
consumer.subscribe(Collections.singletonList("user-events"));
```

#### Topic : 
Kafka topics are the categories used to organize messages. 

```
NOTE : If the broker's setting for 'auto.create.topics.enable' is 'true' (default) then a new topic will be created whenever a consumer or a producer tries to read or write a topic that is not present on the cluster. Same type of messages are pushed to 1 topic e.g. paymentinfo, orderData etc.
```
#### Partition : 
A partition is a physical subdivision of a topic. Every topic is split into one or more partitions. Why is a topic divided?
Each partition:

- Is an ordered, immutable sequence of records.
- Stores data sequentially.
- Is assigned to a broker (Kafka server) for load balancing.
- Has its own offsets (a unique ID for each record within that partition).
- They follow LIFO rule while processing the messages.
- Replication is done at the partition level (each partition has replicas).
- Each partition has 1 consumer out of a consumer group. Suppose there are n partitions of a topic. Each partition contain data of same category. So there will be 1 consumer group consisting of n consumers each reading from a different partition to enable parallel processing.
- One consumer in the group can consume more than one partition
- Each partition has replicas on multiple brokers. So even if a broker fails, another broker with a replica can take over.
- Replicas of a partition follow master slave architecture.n When producer writes message to a partition, the message is written to the replica leader and it's offset is increased. Leader handles all read and write to a topic and followers replicate the leader. 

#### Offset : 
- An offset is a unique identifier for each message within a partition. It acts like a pointer or index, helping Kafka keep track of the position of messages.
- It is needed to maintain an index to check what all data has been read by the consumer. Suppose, consumer reads till offset 3 and goes down for some time, after it again becomes active we will have a record of what data has been consumed and what is left.
  
```
NOTE : Unlike rabbitMQ, messages once read from a partition are not deleted because multiple consumers are allowed to read from a parition.
Instead of deletion, offsets increase per consumer so that next time they do not read the same message.
```


#### Consumer Group : 
- A group of consumers working together to read from same topic in parallel.
- Kafka ensures each partition is only read by one consumer in the group.
- This allows horizontal scaling of processing.
- Just like 1 consumer is associated with 1 partition, 1 consumer group is associated with 1 topic.
- Consumer group is needed to implement parallel processing. Suppose a topic receives 1000 delivery updates per minute for orders and it has only 1 partition. It will be consumed by a single consumer who processes 1 message at a time. This will lead to delay in message processing for older orders. If same task is being done by n consumers, n messages will be processed at the same time.

#### Zookeeper
- Kafka is a distributed system with multiple brokers. It needs:
- A central place to store metadata, like:
    - What topics exist.
    - Where partitions are located.
    - Which broker is the leader for each partition.
- Coordination between brokers, like:
    - Electing a controller broker.
    - Handling broker failures.

#### KRaft
- Kafka’s newer architecture (starting from Kafka 2.8+) eliminates the usage of zookeeper for maintaining metadata and broker coordination.
- A set of brokers form a quorum (called the controller quorum) that collectively manages cluster metadata .
- This quorum elects a controller leader among themselves to coordinate cluster metadata updates.
- One broker in this quorum acts as the controller leader to process metadata changes and coordinate the cluster.
- The quorum ensures high availability and fault tolerance of metadata management — if the current leader fails, another quorum member takes over seamlessly.

#### Summary
<img width="503" alt="image" src="https://github.com/user-attachments/assets/aee0653c-9e47-468a-b16f-58bd9d5fa0cd" />

### __consumer_offset

Internal topic where Kafka stores offsets that tracks how much of a topic's data a consumer has read.


### Starting and stoping kafka server
```
kafka-server-start ~/desktop/kraft-server.properties
Ctrl + C
```
### Kafka server properties
```
# Required roles
process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
controller.quorum.voters=1@localhost:9093

# Listeners
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
inter.broker.listener.name=PLAINTEXT

# Log directory
log.dirs=/tmp/kafka-logs
```
- listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093 
There are 2 types of communications in kafka, producer-consumer and within cluster-quorum. Client will exchange message in plaintext and on port 9092 and quorum members will communicate on 9093 about metadata info and quorum leader info.


- process.roles=broker,controller : 
The process.roles property tells Kafka what roles the current process (broker) should play. This node not only accept producer consumer messages but also participate in quorum communication. This can also possibly become the controller leader responsible for metadata updates.

- node.id=1 : 
assigns unique id to each broker

- controller.listener.names=CONTROLLER

Kafka starts a broker with two listeners: 1. PLAINTEXT on localhost:9092 → For producers, consumers, and inter-broker traffic
2. CONTROLLER on localhost:9093 → For controller quorum (Raft) communication : Tells kafka “For controller communication, use the listener named CONTROLLER, which is bound to port 9093.”

- controller.quorum.voters=1@localhost:9093
Defines members in quorum. Here only 1 member

### Common kafka commands
1. start kafka
```
kafka-server-start ~/desktop/kraft-server.properties
```
2. stop kafka
```
Ctrl + C
```
3. list all topics on kafka : 
```
kafka-topics --list --bootstrap-server localhost:9092
```
4. Error : java.lang.RuntimeException: No readable meta.properties files found while starting server

Run below command and then start server
```
kafka-storage format -t $(kafka-storage random-uuid) -c ~/Desktop/kraft-server.properties
```
### Kafka setup in windows

Create 2 files inside a folder kafka-kraft-docker
1. docker-compose.yml
```
version: '3.8'

services:
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-kraft
    ports:
      - "9092:9092"
      - "9093:9093"
    user: root
    command: >
      bash -c "
        kafka-storage format --ignore-formatted --cluster-id=$$(kafka-storage random-uuid) --config /etc/kafka/kraft-server.properties &&
        kafka-server-start /etc/kafka/kraft-server.properties
      "
    volumes:
      - ./kraft-server.properties:/etc/kafka/kraft-server.properties
      - kraft-data:/tmp/kraft-combined-logs

  kafdrop:
    image: obsidiandynamics/kafdrop
    container_name: kafdrop
    restart: always
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka-kraft:9092
    depends_on:
      - kafka

volumes:
  kraft-data:
```
2. kraft-server.properties
```
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@kafka-kraft:9093

listeners=PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
advertised.listeners=PLAINTEXT://kafka-kraft:9092
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
controller.listener.names=CONTROLLER

log.dirs=/tmp/kraft-combined-logs
auto.create.topics.enable=true
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
num.partitions=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
group.initial.rebalance.delay.ms=0
```

How does kafka and kafdrop start in containers using these files

```
kafka:
    image: confluentinc/cp-kafka:7.5.0
```
- This pulls the official Kafka Docker image with KRaft support.
- Kafka is running in KRaft mode (self-managed metadata, no ZooKeeper).
- KRaft config is passed using a custom config file (kraft-server.properties).

```
kafka-storage format --ignore-formatted --cluster-id=$(kafka-storage random-uuid)
```
This initializes Kafka storage with a random UUID. Required for KRaft mode to work.

```
kafka-server-start /etc/kafka/kraft-server.properties
```
Starts the Kafka server using your custom KRaft configuration.

```
volumes:
  - ./kraft-server.properties:/etc/kafka/kraft-server.properties
  - kraft-data:/tmp/kraft-combined-logs
```

Uses a named volume for persistent logs.

```
user: root
```
Needed so Kafka can write to /tmp/kraft-combined-logs on Windows (avoids permission issues).

```
  kafdrop:
    image: obsidiandynamics/kafdrop
```
pulls Kafdrop image

```
environment:
  KAFKA_BROKERCONNECT: kafka-kraft:9092
```
kafdrop connects to kafka using this

Run this yml file using this command
```
D:\kafka-kraft-docker>docker-compose up -d
```

check the containers active
```
docker ps
```
To execute any kafka command, now we have to enter the container's shell
```
docker exec -it kafka-kraft bash
[root@993f04b65d89 appuser]# kafka-topics --create \
>   --topic test-topic \
>   --bootstrap-server localhost:9092 \
>   --partitions 1 \
>   --replication-factor 1
```
### What is the role of the replication factor? How does Kafka ensure durability and availability?
Every Kafka topic is split into partitions.Each partition is stored on multiple brokers (servers) — this number is the replication factor.
Example: replication factor = 3 → each partition has 1 leader + 2 followers.

### How does kafka ensure message durability while publishing ?
Kafka can be configured with acks to ensure messages are written properly. When a producer sends a message, the broker replies with an acknowledgment (ack).
1. acks=0 :
     - Producer does not wait for any broker ack.
     - Fire-and-forget: fastest, but you may lose messages if broker/network fails.
2. ack = 1 :
     - Producer waits for leader broker only to write the message to its local log.
     - Followers may not have replicated it yet.
3. ack = all
     - leader waits until all in-sync replicas (ISR) write the message before acking (slower, very safe). This guarantees data isn’t lost even if one broker crashes.

### How does kafka ensure message durability while consuming ?
Suppose consumer reads a buggy message that throws an exception
case 1 : Offset committed before-hand : message is lost and silent failed<br/>
case 2 : Offset is not committed and message is read again. Thi smeans consumer keeps retrying this again n again. Im this scenario, Instead of re-consuming endlessly, consumer sends it to a retry topic, another consumer consumes from here and retries N times to push to the partition exponentially, if it fails indefinitely, it is marked permanent_lost;

### How does kafka ensure availability?
If the leader broker for a partition fails:

One of the in-sync follower replicas is elected as the new leader.Producers and consumers automatically switch to the new leader.This keeps the system running without downtime. Kafka only promotes an ISR(In-sync replica) follower to leader — ensuring no data loss during failover.

### How does consumer offset management work?
- Offsets are stored in a special internal topic called __consumer_offsets.
- Each record in __consumer_offsets is essentially a key-value pair.
      - Key : consumer_group + topic + partition
      - Value : which offset to be set next for this partition

- Offsets are not tracked per consumer but per consumer group.
- If current read offsets are not commited, then after system failure, consumer will start consuming from last committed offset.
  
### How does Kafka ensure ordering of messages? In what scenarios can ordering be lost?
Within a Partition : 
- Kafka appends messages sequentially to a partition’s log.
- Each message gets a monotonically increasing offset.

Outside of partition : 
- Kafka gives no guarantee to message ordering across partition, because message 1 can be (n + 10 )th message in partition 0 and ,essage 2 can be nth message in partition 1.
- So accross partitions, consumer group might process M1 before M1

### Explain the difference between ZooKeeper-based Kafka and KRaft (Kafka Raft mode)?
Zookeeper :
- used in leader selection
-  metadata management.
-  bottleneck while scaling on large clusters

KRaft :
- A set of brokers run as controllers.
- Controllers use Raft consensus to agree on cluster metadata and leader election

### When does a consumer group rebalance?
1. a new consumer joins the group
2. consumer leaves the group
3. new partition is created

### What happens in consumer group rebalance?
- All consumers in the group stop consuming messages temporarily.
- consumers commit offset and revoke their partitions
- Group Coordinator runs a partition assignment strategy (e.g., Range, RoundRobin, Sticky).
- Assigns partitions to consumers so each partition has exactly one owner.

### How do you decide the number of partitions for a topic? What trade-offs are involved?
More partitions = More parallelism = More throughput
But also <br/>
More partitions = more replicas = more metadata stored in ZooKeeper/KRaft

| Factor               | Fewer Partitions                                      | More Partitions                                                   |
|-----------------------|-------------------------------------------------------|-------------------------------------------------------------------|
| **Throughput (parallelism)** | Limited by partition count                           | Higher throughput (more consumers in parallel)                    |
| **Ordering Guarantees**      | Easier (fewer partitions to manage order per key)   | Still ordered per partition, but global order harder              |
| **Latency**                  | Less overhead                                      | Higher overhead in coordination & replication                     |
| **Cluster Overhead**         | Lower metadata, faster rebalance                   | Higher metadata load, slower rebalances                           |
| **Scalability**              | Hard to add consumers beyond partition count       | Scales better (but diminishing returns after a point)             |
| **Operational Cost**         | Lightweight                                        | Too many partitions = stress on controller, memory, and network   |

Start with a reasonable guess: e.g.,

1 partition per expected consumer thread,

or 2–3 × number of brokers.

Use Kafka’s partition reassignment tool to add partitions as needed later

### Difference between kafka and rabbitMQ.
# RabbitMQ vs Kafka

| Feature                 | **RabbitMQ**                                                                 | **Kafka**                                                                             |
| ----------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Type**                | Traditional **message broker** (message queue).                              | **Distributed event streaming platform** (log-based).                                 |
| **Message Model**       | **Push-based** – broker pushes messages to consumers.                        | **Pull-based** – consumers poll and read at their own pace.                           |
| **Use Case**            | Short-lived tasks, job queues, request-response, event-driven communication. | Event streaming, real-time analytics, log aggregation, data pipelines.                |
| **Message Storage**     | Message is deleted once consumed (unless persisted manually).                | Messages are persisted (default: 7 days or longer) and can be re-read multiple times. |
| **Throughput**          | Lower (10s–100s of thousands msgs/sec).                                      | Very high (millions msgs/sec).                                                        |
| **Ordering**            | Per-queue ordering guaranteed.                                               | Per-partition ordering guaranteed.                                                    |
| **Delivery Guarantees** | At-most-once, At-least-once, Exactly-once (with plugins).                    | At-least-once, Exactly-once (built-in).                                               |
| **Scalability**         | Good, but limited by broker architecture.                                    | Excellent – designed for distributed scale-out.                                       |
| **Latency**             | Very low (ms-level).                                                         | Low, but tuned more for **throughput** than single-digit latency.                     |
| **Durability**          | Optional (messages can be transient).                                        | Persistent by default (append-only log on disk).                                      |
| **Consumer Model**      | Competing consumers (queue distributes msgs across consumers).               | Pub-sub (many consumers can read the same stream independently).                      |
| **Maturity**            | Very mature, battle-tested (2007).                                           | Newer but highly popular (2011, by LinkedIn).                                         |
### Why is kafka throughput high while rabbit's is slow?
RabbitMQ throughput is lower because it pushes messages, deletes them after consumption, and the broker does more work (tracking acks, retries, queues).
Kafka throughput is higher because it just writes messages in a log (append-only), keeps them for a set time, and consumers pull at their own pace.

### what is At-most-once, At-least-once, Exactly-once delivery guarantee and how rabbit-mq and kafka implement it?
- At-most-once: Message may be lost, but never duplicated.

- At-least-once: Message is never lost, but may be duplicated.

- Exactly-once: Message is delivered once and only once, no loss, no duplication.

#### Rabbit : 
1. At most once : - consumer has set auto-ack: on, so the messages are always acknowledged iven if it crashes.
2. At least once : auto-ack : off, if consumer fails before reading, it is retried with dlq support.

#### Kafka : 
1. At-most-once : If consumer commits offset before processing the message.
2. At least once : Consumer commits offset after processing. if crash happens after reading but commiting offset, messages can be duplicated
3. Exactly once : using transactional writes. Producer and consumer work under same transaction, so if anything fails, it is reverted.
