# **Kafka** :

**Apache Kafka** is a distributed event-streaming platform used to:

- Publish and consume messages/events
- Build event-driven microservices
- Process large volumes of real-time data
- Decouple services
- Provide reliable, scalable asynchronous communication

### How Kafka Works?

1️⃣ Producer publishes messages/events to a Kafka Topic.

2️⃣ Kafka stores the events in partitions.

3️⃣ Consumers subscribe to topics and process the events.

4️⃣ Messages remain in Kafka for a configurable retention period.

### Understanding Pub-Sub (Publish-Subscribe)

Pub-Sub is a messaging pattern where:

📤 Publisher (Producer) sends messages to a topic.

📥 Subscribers (Consumers) receive messages from that topic.

The producer doesn't need to know who the consumers are, making systems highly flexible and scalable.

### What is Event-Driven Architecture (EDA)?

Event-Driven Architecture is a design pattern where services communicate through events instead of direct API calls.

Each service reacts independently to events, improving system flexibility and resilience.

### Architecture

```
Producer
   |
   v
+-------------------+
|      Kafka        |
|                   |
| Topic: orders     |
|  Partition 0 ---> |
|  Partition 1 ---> |
|  Partition 2 ---> |
+-------------------+
   |
   v
Consumer Group
   |
   +--> Consumer 1
   +--> Consumer 2
```

---

### Important Kafka Components

#### Producer

Application that **publishes messages** to Kafka.

```
Order Service → Kafka
```

Example:

```java
kafkaTemplate.send("orders",order);
```

#### Consumer

Application that **reads messages** from Kafka.

```
Kafka → Payment Service
```

#### Topic

A logical category/name to which messages are published.

Examples:

```
orders
payments
customer-events
notifications
```

#### Partition

A topic is divided into **partitions** for scalability and parallel processing.

```
orders
 |
 +--- Partition 0
 +--- Partition 1
 +--- Partition 2
```

**Messages within a partition are ordered.**

#### Broker

A Kafka server that stores and serves messages.

A Kafka cluster can have multiple brokers:

```
Kafka Cluster

Broker 1
Broker 2
Broker 3
```

#### Consumer Group

A group of consumers working together to consume messages.

```
Topic: orders

P0 → Consumer 1
P1 → Consumer 2
P2 → Consumer 3
```

**One partition can be consumed by only one consumer within the same consumer group at a time.**

---

### Topic vs Partition

### Topic

Logical stream/category of messages.

### Partition

Physical/logical subdivision of a topic used for:

- Scalability
- Parallelism
- Ordering

Example:

```
Topic: orders

Partition 0 → Order 1, Order 4, Order 7
Partition 1 → Order 2, Order 5, Order 8
Partition 2 → Order 3, Order 6, Order 9
```

Kafka distributes messages across partitions.

---

#### Why does Kafka use partitions?

Mainly for **horizontal scalability**.

Suppose:

```
1 partition → 1 consumer
```

Only one consumer processes messages.

With:

```
10 partitions → 10 consumers
```

you can process messages in parallel.

Therefore:

> More partitions → greater potential consumer parallelism.
> 

---

#### Kafka Message Structure

A Kafka record can contain:

```
Key
Value
Timestamp
Headers
Offset
```

Example:

```
Topic: orders
Key: customer123
Value: Order details
```

---

#### What is an Offset?

An **offset is the position of a message within a partition**.

Example:

```
Partition 0

Offset 0 → Order A
Offset 1 → Order B
Offset 2 → Order C
Offset 3 → Order D
```

Kafka consumers track offsets to know where they are in the stream.

### Important

Offset is **unique only within a partition**, not across the entire Kafka topic.

---

#### Consumer Group

A consumer group allows multiple consumers to process messages in parallel.

Example:

```
Topic: orders
Partitions: 3

       Kafka
         |
   +-----+-----+
   |     |     |
  P0    P1    P2
   |     |     |
  C1    C2    C3
```

All three consumers belong to:

```
order-processing-group
```

If there are:

```
3 partitions
5 consumers
```

only **3 consumers can actively consume**, while 2 remain idle.

If there are:

```
5 partitions
3 consumers
```

some consumers will consume multiple partitions.