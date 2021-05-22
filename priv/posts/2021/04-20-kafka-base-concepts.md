%{
  title: "Kafka Basic concepts",
  author: "Luiz Cattani",
  tags: ~w(elixir phoenix process),
  description: "My notes explaning the basic concepts of Kafka"
}
---

# Kafka
Message Broker with high troughpout

### Concept
Producers - Send a message to a topic
Consumers - pull messages from a topic

Kafka topics are divided into a number of partitions
Partitions allows parallelize a topic by splitting the data in a particular
topic across multiple brokers

### Offset
Each message has a offset within a partitions has a offset(identifier), responsible
to ordering the messages the form immutable sequence.

### Logs
Another way to view a partition is as a `Logs`. 
A data sources writes messages to a log and one or ore consumers reads from the
log at the point in time they choose.

Kafka retains messages for a configurable period of time, if the consumer is down 
for an hour, it can begin read the messages again after restart in the correct offset

### Particions and Brokers
Each Broker holds a number of partitions and each of these 
partitions can be either a leader or a replica for a topic.
All writes and reads to a topic go through the leader and the leader coordinates updating replicas with new data.
If a `leader` fails, a replica take over as the new leader

### Consistency as a Kafka Client
Kafka clients come in two flavours: producer and consumer. 

#### Producer
For a producer we have three choices. On each message we can:
(1) wait for all in sync replicas to acknowledge the message, 
(2) wait for only the leader to acknowledge the message, or 
(3) do not wait for acknowledgement. 

Each of these methods have their merits and drawbacks and it is up to the 
system implementer to decide on the appropriate strategy for their system 
based on factors like consistency and throughput.

#### Consumer
On the consumer side, we can only ever read committed messages (i.e., those that have been written to all in sync replicas). Given that, we have three methods of providing consistency as a consumer: 
(1) receive each message at most once, 
(2) receive each message at least once, or 
(3) receive each message exactly once

# Conclusion
Kafka is quickly becoming the backbone of many organization’s data pipelines — 
and with good reason. 

By using Kafka as a message bus we achieve a high level of parallelism and 
decoupling between data producers and data consumers, making our architecture more 
flexible and adaptable to change