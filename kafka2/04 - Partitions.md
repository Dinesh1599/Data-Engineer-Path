# 04 - Partitions

#kafka #partitions #scalability #parallelism #ordering

← [[kafka2/Kafka]]

---

## What is a Partition?

A **partition** is a subdivision of a topic. Each topic can have one or more partitions, and each partition is an **ordered, immutable sequence of events**.

```
Topic: orders
├── Partition 0: [event0] [event1] [event4] [event7] ...
├── Partition 1: [event2] [event5] [event8] ...
└── Partition 2: [event3] [event6] [event9] ...
```

Partitions are the key mechanism behind Kafka's scalability.

---

## Why Partitions Exist

Without partitions, one topic = one sequence = one consumer processing it = bottleneck.

With partitions:
- Multiple consumers process different partitions **in parallel**
- Each partition lives on a different broker → distributed storage and load
- More partitions = more parallelism (up to a limit)

**Analogy:** Think of a video editing team. One editor handling all videos is a bottleneck. So you split: one editor for Shorts, one for long-form, one for podcasts. They work in parallel. That's partitioning.

---

## How Events Are Assigned to Partitions

When a producer sends an event, Kafka decides which partition it goes to based on:

### 1. Explicit Key
```python
producer.produce(topic="orders", key="user-123", value=order_data)
```
Kafka hashes the key: `partition = hash(key) % num_partitions`

Events with the **same key always go to the same partition**. This guarantees ordering for that key.

> **Example:** All orders from `user-123` go to the same partition → they are processed in order.

### 2. No Key (Round-Robin)
```python
producer.produce(topic="orders", value=order_data)  # no key
```
Kafka distributes events across partitions in round-robin. No ordering guarantee across events.

### 3. Custom Partitioner
You can write your own logic to determine which partition an event goes to.

---

## Partitions and Ordering

**Important:** Kafka only guarantees ordering **within a partition**, not across partitions.

```
Partition 0: [order-A at 10:01] [order-A at 10:05]  ← ordered ✅
Partition 1: [order-B at 10:02] [order-B at 10:06]  ← ordered ✅

Cross-partition: 10:01, 10:02, 10:05, 10:06?  ← NOT guaranteed ❌
```

If ordering across all events matters, use a single partition (but you lose parallelism). Usually, per-key ordering (same user, same entity) is sufficient.

---

## Partition Leadership

Each partition has one **leader broker** and zero or more **replica brokers**.

- Producers write to the **leader**
- Consumers read from the **leader** (by default)
- Replicas stay in sync as followers
- If the leader broker fails, a follower is promoted to leader

See [[07 - Replication and Fault Tolerance]] for details.

---

## How Many Partitions Should You Create?

There's no universal answer, but guidelines:

| Factor | Guidance |
|---|---|
| Throughput needed | More throughput → more partitions |
| Number of consumers | Max parallelism = number of partitions |
| Retention size | More partitions = more file handles on broker |
| Message ordering needs | Strict ordering → fewer partitions (or 1) |

**Starting point:** If you expect 10 consumers max, start with 10-20 partitions. You can increase partitions later (but not decrease, and adding partitions breaks key-based ordering for existing data).

> **Tip:** It's easier to start with more partitions than to add them later. But don't go overboard — thousands of partitions per broker cause overhead.

---

## Consumer Groups and Partitions

Each partition is consumed by **exactly one consumer** within a consumer group at any time.

```
Topic: orders (3 partitions)
Consumer Group: order-processors (3 consumers)

Consumer A → Partition 0
Consumer B → Partition 1
Consumer C → Partition 2
```

If you add a 4th consumer, it sits idle (no partition for it). If you remove a consumer, its partition is reassigned.

See [[05 - Consumer Groups and Offsets]] for more.

---

← [[03 - Brokers and Clusters]] | [[05 - Consumer Groups and Offsets]] →
