# 05 - Consumer Groups and Offsets

#kafka #consumer-groups #offsets #coordination

← [[kafka2/Kafka]]

---

## Consumer Groups

A **consumer group** is a set of consumers that cooperate to consume a topic together. All consumers in a group share the same `group.id`.

Kafka distributes the topic's partitions among the consumers in a group, so each partition is handled by **exactly one consumer** at a time.

```
Topic: orders (4 partitions)
Consumer Group: "invoice-service"

Consumer-1 → Partition 0
Consumer-2 → Partition 1
Consumer-3 → Partition 2
Consumer-4 → Partition 3
```

---

## Why Consumer Groups Matter

They enable **horizontal scaling** of consumers.

If Partition 0 suddenly gets 100x more events (e.g., US orders spike), you can add more consumers to the group. Kafka reassigns partitions automatically.

**Different groups are completely independent:**

```
Topic: orders
├── Group: "invoice-service" → reads all events (for invoicing)
├── Group: "email-service"   → reads all events (for confirmation emails)
└── Group: "analytics"       → reads all events (for dashboards)
```

Each group gets **its own copy of every event**. They don't interfere with each other.

---

## What Happens If There Are More Consumers Than Partitions?

The extra consumers sit **idle**. They are ready to take over if an active consumer dies.

```
Topic: orders (3 partitions)
Consumer Group (5 consumers):
  Consumer-1 → Partition 0
  Consumer-2 → Partition 1
  Consumer-3 → Partition 2
  Consumer-4 → idle (standby)
  Consumer-5 → idle (standby)
```

> **Rule:** Max useful parallelism = number of partitions. More consumers than partitions = wasted resources (but good for failover).

---

## Offsets

An **offset** is an integer that identifies the position of an event within a partition. Think of it as a line number in a file.

```
Partition 0:
  Offset 0: {"order_id": "aaa", ...}
  Offset 1: {"order_id": "bbb", ...}
  Offset 2: {"order_id": "ccc", ...}  ← consumer last read here
  Offset 3: {"order_id": "ddd", ...}  ← next to read
```

Kafka tracks **per-group, per-partition** offsets. This means:
- Group A can be at offset 50 on Partition 0
- Group B can be at offset 12 on Partition 0

They are completely independent.

---

## Committing Offsets

When a consumer reads an event and finishes processing it, it **commits the offset** — telling Kafka "I've processed up to here."

### Auto Commit (default)
Kafka automatically commits the offset every N milliseconds.
```python
consumer_config = {
    "enable.auto.commit": True,
    "auto.commit.interval.ms": 5000  # every 5 seconds
}
```
Risk: If consumer crashes between auto-commits, it may re-read some events.

### Manual Commit (safer)
You commit only after successfully processing the event.
```python
msg = consumer.poll(1.0)
process(msg)           # do your work first
consumer.commit(msg)   # then commit
```
This gives you full control over delivery guarantees. See [[08 - Delivery Semantics]].

---

## Auto Offset Reset

What happens when a consumer group is **new** and has no committed offset yet? The `auto.offset.reset` config decides:

| Value | Behaviour |
|---|---|
| `earliest` | Start from the very first event in the topic |
| `latest` | Start from new events only (ignore existing ones) |
| `none` | Throw an error if no offset is found |

```python
consumer_config = {
    "auto.offset.reset": "earliest"  # read from beginning
}
```

**Use `earliest`** when you want to process all historical events on first startup.
**Use `latest`** when you only care about new events going forward.

---

## Rebalancing

When a consumer **joins or leaves** a group, Kafka triggers a **rebalance** — redistributing partitions among the active consumers.

During a rebalance:
- Consumers stop consuming temporarily
- Kafka reassigns partitions
- Consumers resume from their committed offsets

Rebalances are normal and automatic, but they briefly pause processing. Minimise unnecessary rebalances by:
- Keeping consumer instances stable
- Increasing `session.timeout.ms` to avoid false deaths
- Using **static group membership** (`group.instance.id`) for stable consumers

---

## __consumer_offsets Topic

Kafka stores all committed offsets in an internal topic called `__consumer_offsets`. This is a regular Kafka topic that Kafka manages for you. You never write to it directly.

---

← [[04 - Partitions]] | [[06 - KRaft Mode]] →
