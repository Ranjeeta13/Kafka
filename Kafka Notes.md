# Kafka vs Database – System Design Notes

## 1. Can Databases Handle Concurrency?

Modern databases like:

* MySQL
* PostgreSQL
* MongoDB

**can handle thousands of concurrent operations.**

They achieve this using:

* connection pools
* threads
* transaction managers
* locks / MVCC (Multi-Version Concurrency Control)

So the statement **"databases cannot handle multiple operations simultaneously" is incorrect.**

However, databases should **not be used directly for extremely high event streams**.

---

# 2. The Real Problem: Traffic Spikes

Example: Ride requests in a ride-sharing application.

Suppose:

```
100,000 users request rides at the same time
```

Without a message broker:

```
Users → API → Database
```

Problems that may occur:

* database connection pool exhaustion
* increased query latency
* disk I/O bottleneck
* CPU spikes
* degraded performance

Even though the database may not crash immediately, **performance degrades significantly**.

---

# 3. Kafka as a Buffer (Shock Absorber)

Apache Kafka acts as an **event streaming buffer** between services and databases.

Architecture:

```
Users → API → Kafka → Consumers → Database
```

Flow:

```
100,000 events/sec
        ↓
       Kafka
   (stores events)
        ↓
Consumers process
at controlled rate
        ↓
Database
```

Kafka absorbs spikes and allows downstream systems to process events at a **safe rate**.

---

# 4. Event-Driven Architecture

One event may be required by multiple services.

Example: "Ride Requested"

Services that may need this event:

* ride matching service
* pricing service
* notification service
* analytics service
* fraud detection system

Without Kafka:

```
Ride Service
   ↓
Call multiple APIs
```

With Kafka:

```
Ride Service → Kafka
                ↓
        Multiple Consumers
```

This enables **decoupled microservices**.

---

# 5. Why Kafka is Used

Kafka is useful for:

* handling traffic spikes
* asynchronous processing
* event streaming
* decoupling microservices
* replaying events
* high-throughput data pipelines

Companies using Kafka:

* Uber
* LinkedIn
* Netflix

---

# 6. Kafka Partitioning

Kafka topics are divided into partitions.

Example:

```
Topic: ride-requests
Partitions: 3
```

Kafka decides partition using hashing.

Formula:

```
partition = hash(key) % number_of_partitions
```

Example:

```
userId = 12345
hash(12345) = 789456

partition = 789456 % 3 = 0
```

Benefits:

* parallel processing
* ordering maintained per key
* scalable throughput

---

# 7. Kafka Message Processing (Sync vs Async)

Kafka producers **send messages asynchronously by default**.

Example:

```java
producer.send(record);
```

Producer sends the event and continues processing without waiting.

For synchronous behavior:

```java
producer.send(record).get();
```

Now the producer waits for acknowledgement.

---

# 8. Producer Acknowledgement Settings

Kafka reliability is controlled using `acks`.

### acks = 0

Producer does not wait for acknowledgement.

```
Producer → Kafka
(no confirmation)
```

Fastest but unsafe.

---

### acks = 1

Leader broker confirms write.

```
Producer → Leader → ACK
```

Most commonly used.

---

### acks = all

Leader and all replicas confirm write.

```
Producer → Leader → Replicas → ACK
```

Strongest durability but slower.

---

# 9. Why Kafka is Extremely Fast

Kafka performance comes from:

* sequential disk writes
* partition parallelism
* batching
* zero-copy transfer
* distributed architecture

Kafka stores events in an **append-only log**:

```
Partition log

event1
event2
event3
event4
```

Sequential writes are much faster than random writes.

---

# 10. **Why use Kafka if databases already support concurrency?**

Answer:

> Kafka is used to handle high-throughput event streams, buffer traffic spikes, decouple microservices, and process events asynchronously so that databases and services are not overwhelmed by sudden load.

---
