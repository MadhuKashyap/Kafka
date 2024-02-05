### What is Kafka?
Kafka is a distributed publish-subscribe messaging system.Data is written to Kafka topics by producers and consumed from those topics by consumers.
### Why do we need Kafka ?
<img width="601" alt="image" src="https://github.com/MadhuKashyap/Kafka/assets/40714383/e0bdf5ee-fe67-4e78-a5e1-9a7fe8be8650">
Suppose we have these applications which need to communicate with each other. One bottlneck is maintaining multiple connections at all times. If 1 of the application goes down, all data coming to that application will be lost.
Here Kafka comes into picture
<img width="1251" alt="image" src="https://github.com/MadhuKashyap/Kafka/assets/40714383/d4e8fd80-3f0c-466d-9eb2-a8dbd3f5a134">
All the publishing applications will publish data and receiving applications will consume the data whenever they are avaialable, this way we maintain relatively lesser number of connections between applications

### Important members of kafka

- Producer
- Consumer
- Topic
- Partition
- Broker
- Consumer Group
- Offset




