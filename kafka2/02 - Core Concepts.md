# 02 - Core Concepts

#kafka #fundamentals #events #topics #producers #consumers

← [[Kafka MOC]]

---

## The Four Building Blocks

Kafka revolves around four concepts. Everything else builds on these.

---

## 1. Events

An **event** is a record of something that happened.

- "A user placed an order"
- "A payment was processed"
- "A server CPU hit 90%"

Events are **immutable** — once written, they don't change. They are structured as:

```json
{
  "key": "user-123",
  "value": {
    "order_id": "abc-456",
    "item": "Mushroom Pizza",
    "quantity": 2,
    "user": "nana_tech"
  },
  "timestamp": 1712000000000,
  "headers": {
    "source-service": "order-service"
  }
}
```

- **Key** — used for partitioning (events with the same key go to the same partition). Can be null.
- **Value** — the actual payload (usually JSON or Avro)
- **Timestamp** — when the event occurred
- **Headers** — optional metadata (tracing IDs, source service, etc.)

> **Analogy:** An event is like a record in an append-only log. You never update row 5 — you add row 6 saying "row 5 was updated."

---

## 2. Topics

A **topic** is a named category where events are stored. Like a folder or a database table.

```
kafka/
├── topic: orders
├── topic: payments
├── topic: inventory
└── topic: user-activity
```

- Topics are **append-only** — new events are added to the end
- Events in a topic are **ordered by time** (within a partition)
- A topic can hold millions of events

**You** decide what topics to create, based on your app's needs. There's no one right answer, but a common approach is: one topic per event type.

---

## 3. Producers

A **producer** is any application or service that **writes events to a topic**.

```python
producer.produce(topic="orders", value=order_event)
```

Key properties:
- Producers are **decoupled from consumers** — they don't know or care who reads the events
- Producers can write to **multiple topics**
- Producers are **non-blocking** by default — fire and forget

---

## 4. Consumers

A **consumer** is any application or service that **reads events from a topic**.

```python
consumer.subscribe(["orders"])
message = consumer.poll(timeout=1.0)
```

Key properties:
- Consumers **pull** from Kafka (Kafka does not push to them)
- Consumers track where they are using an **offset**
- Multiple consumers can read the **same topic independently**
- A consumer can subscribe to **multiple topics**

---

## How They Fit Together

```
Producer                    Kafka                      Consumer
--------                    -----                      --------
Order Service  --event--▶  [orders topic]  --event--▶  Invoice Service
                           [orders topic]  --event--▶  Email Service
                           [orders topic]  --event--▶  Analytics Service
```

All three consumers get the **same event**. They process it independently, at their own pace.

---

## Kafka is NOT a Database

Common misconception. Kafka is not a replacement for PostgreSQL or MongoDB.

| Kafka | Database |
|---|---|
| Append-only event log | Read/write/update/delete |
| Optimised for streaming | Optimised for querying |
| Short-to-medium retention | Long-term storage |
| Consumers pull at their pace | Queries on demand |

Think of Kafka as a **pipe** that moves data between systems. The actual data lives in your databases. Kafka is the event highway between them.

---

> **Summary:** Producer writes event → event lands in a topic → consumer reads from that topic. That's the entire flow.

---

← [[01 - Why Kafka Exists]] | [[03 - Brokers and Clusters]] →
