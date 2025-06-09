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

Suppose producer is producing huge volume of data, then a single kafka broker may not be able to handle the load. In that case               we need to add multiple brokers who can consume requests parallely to levarage scalability.

<img width="1204" alt="image" src="https://github.com/user-attachments/assets/b219f408-b4db-4f8a-bb0e-46387832d00b" />

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

#### Summary
<img width="503" alt="image" src="https://github.com/user-attachments/assets/aee0653c-9e47-468a-b16f-58bd9d5fa0cd" />


### Starting and stoping kafka server
```
kafka-server-start ~/desktop/kraft-server.properties
Ctrl + C
```

