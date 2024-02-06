### What is Kafka?
Kafka is a distributed publish-subscribe messaging system.Data is written to Kafka topics by producers and consumed from those topics by consumers.
### Why do we need Kafka ?
<img width="601" alt="image" src="https://github.com/MadhuKashyap/Kafka/assets/40714383/e0bdf5ee-fe67-4e78-a5e1-9a7fe8be8650">

Suppose we have these applications which need to communicate with each other. One bottlneck is maintaining multiple connections at all times. If 1 of the application goes down, all data coming to that application will be lost. Here Kafka comes into picture

<img width="1251" alt="image" src="https://github.com/MadhuKashyap/Kafka/assets/40714383/d4e8fd80-3f0c-466d-9eb2-a8dbd3f5a134">

All the publishing applications will publish data and receiving applications will consume the data whenever they are avaialable, this way we maintain relatively lesser number of connections between applications

### Important members of kafka

- Producer : Publishes message to kafka cluster
- Consumer : Consumes messages from topics
- Topic : Kafka topics are the categories used to organize messages. If the broker's setting for 'auto.create.topics.enable' is 'true' (default) then a new topic will be created whenever a consumer or a producer tries to read or write a topic that is not present on the cluster.
- Partition : Partitions are fractions of a topic. They follow LIFO rule while processing the messages.
- Broker : A Kafka broker is a server in the cluster this will receive and send the data.
- Cluster : Group of brokers working together. This is done to serve the purpose of scalability and availability(no single point of failure)
- Consumer Group : Consumer groups work together and process events from a topic in parallel. 
- Offset : The offset is a unique ID assigned to the partitions, which contains messages. 

### Important points to note

- each partition is consumed by exactly one consumer in the group
- one consumer in the group can consume more than one partition
- the number of consumer processes in a group must be <= number of partitions
- Just like 1 consumer is associated with 1 partition, 1 consumer group is associated with 1 topic. If group has n consumers, and topic has m partitions where n > m, some consumers will sit idle and read nothing

### What is a partition key?
This is used to make consumers read from a particular partition. While calling Kafka.consume(topic, key) we can provide key and topic name to specify the topic and partition from which the consumer will read

### Why do we need to maintain offset in partition?
It is needed to maintain an index to check what all data has been read by the consumer. Suppose, consumer reads till offset 3 and goes down for some time, after it again becomes active we will have a record of what data has been consumed and what is left

