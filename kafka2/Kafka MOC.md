# Kafka — Map of Content

#kafka #index #distributed-systems #messaging #MOC

> **What is Kafka?**
> Apache Kafka is a distributed event streaming platform built for high-throughput, fault-tolerant, real-time data pipelines. Think of it as a highly reliable conveyor belt that sits between your services, carrying events from producers to consumers without them ever needing to talk to each other directly.

---

## 🗺️ Navigation

### Foundations
- [[01 - Why Kafka Exists]] — The problem Kafka solves, tight coupling, synchronous bottlenecks
- [[02 - Core Concepts]] — Events, Topics, Producers, Consumers — the building blocks
- [[03 - Brokers and Clusters]] — How Kafka stores and manages data across servers
- [[04 - Partitions]] — How Kafka achieves parallelism and scale
- [[05 - Consumer Groups and Offsets]] — How consumers coordinate and track progress
- [[06 - KRaft Mode]] — Kafka's modern self-managed metadata system (no more Zookeeper)

### Reliability & Guarantees
- [[07 - Replication and Fault Tolerance]] — How Kafka survives broker failures
- [[08 - Delivery Semantics]] — At-most-once, at-least-once, exactly-once
- [[09 - Retention Policies]] — How long Kafka keeps your data and why it matters

### Building with Kafka
- [[10 - Producers In Depth]] — Batching, compression, acknowledgements, callbacks
- [[11 - Consumers In Depth]] — Polling, offset commits, rebalancing
- [[12 - Kafka Connect]] — Moving data between Kafka and external systems
- [[13 - Kafka Streams]] — Real-time stream processing inside Kafka
- [[14 - Schema Registry and Avro]] — Enforcing consistent message structure

### Operations & Production
- [[15 - Deploying Kafka]] — Docker, Kubernetes, managed cloud services
- [[16 - Security]] — TLS, SASL, ACLs
- [[17 - Monitoring and Observability]] — Consumer lag, broker metrics, tooling
- [[18 - Performance Tuning]] — Throughput vs latency, batch sizes, compression

### Design Patterns
- [[19 - Topic Design]] — Naming, structure, how many partitions
- [[20 - Event Sourcing and CQRS]] — Patterns Kafka enables
- [[21 - Dead Letter Queues]] — Handling poison pill messages
- [[22 - Idempotent Consumers]] — Handling duplicate messages safely

### Practical
- [[23 - Python + Kafka (confluent-kafka)]] — Hands-on producer and consumer in Python
- [[24 - Kafka CLI Cheatsheet]] — Useful commands for inspection and debugging

---

## 🧠 Quick Mental Model

```
[Order Service] ──produces──▶ [Kafka Broker]
                                    │
                         ┌──────────┼──────────┐
                         ▼          ▼          ▼
               [Invoice Svc]  [Email Svc]  [Analytics Svc]
                 (consumer)    (consumer)    (consumer)
```

- Services **do not call each other**
- They communicate **through Kafka**
- Kafka **persists** every event — consumers can replay

---

## 📌 Key Terms at a Glance

| Term | One-liner |
|---|---|
| Event | A thing that happened, stored as key-value data |
| Topic | A named category/folder where events live |
| Partition | A subdivision of a topic for parallelism |
| Producer | A service that writes events to a topic |
| Consumer | A service that reads events from a topic |
| Broker | A Kafka server that stores and serves data |
| Consumer Group | A set of consumers sharing the load of a topic |
| Offset | A pointer to where a consumer is in a partition |
| KRaft | Kafka's built-in cluster coordination (replaces Zookeeper) |
| Replication Factor | How many copies of data exist across brokers |

---

## 🔗 External Resources
- [Apache Kafka Official Docs](https://kafka.apache.org/documentation/)
- [Confluent Developer](https://developer.confluent.io/)
- [Kafka: The Definitive Guide (free PDF from Confluent)](https://www.confluent.io/resources/kafka-the-definitive-guide/)
