# 20 - Event Sourcing and CQRS

#kafka #event-sourcing #CQRS #architecture #design-patterns

← [[Kafka MOC]]

---

## Event Sourcing

### The Core Idea

In traditional systems, you store **current state**:

```sql
-- Orders table: current state only
UPDATE orders SET status = 'shipped' WHERE id = 'abc-123';
```

The history is gone. You don't know when it was placed, when it was paid, or what happened in between.

**Event sourcing stores every change as an immutable event** (the "source of truth"), and derives current state by replaying those events.

```
Events (append-only):
  order.placed    → { order_id: abc-123, item: Pizza, qty: 2 }
  order.paid      → { order_id: abc-123, amount: 19.99 }
  order.shipped   → { order_id: abc-123, tracking: XYZ }

Current state = result of replaying all events for abc-123
```

---

### Why Kafka is Perfect for Event Sourcing

Kafka IS an append-only event log. You don't need a separate event store:

```
Kafka topic: orders (compacted)
  key: abc-123 → event: placed
  key: abc-123 → event: paid
  key: abc-123 → event: shipped
```

Replay from offset 0 to rebuild full history. Use compaction to keep only the latest version per key (if you only need current state).

---

### Benefits of Event Sourcing

| Benefit | Why It Matters |
|---|---|
| Full audit trail | Know exactly what happened and when |
| Time travel | Rebuild state as of any past point in time |
| Event replay | Reprocess with updated business logic |
| Decoupled side effects | New service? Subscribe and replay from start |
| Debugging | Reproduce any bug by replaying the same events |

---

### Challenges

- **Querying is harder** — you can't `SELECT * WHERE status = 'shipped'` directly
- **Schema evolution** — old events must still be replayable with new code
- **Event versioning** — need a strategy for changing event formats
- **Eventual consistency** — state is derived, not instantly consistent

---

## CQRS — Command Query Responsibility Segregation

CQRS separates your data model into two parts:

- **Command side** — handles writes (produces events to Kafka)
- **Query side** — handles reads (consumes events, builds a queryable view)

```
User Action
    │
    ▼
[Command Handler] ──event──▶ [Kafka] ──event──▶ [Read Model Builder]
  (write logic)                                       │
                                                      ▼
                                              [Queryable DB]
                                           (Postgres, Elastic, Redis)

User Query ──────────────────────────────────▶ [Queryable DB]
```

---

### Why Combine CQRS with Kafka?

1. **Write path** — Order service produces `order.placed` event to Kafka
2. **Read path** — Multiple consumers each build their own read model:
   - A Postgres table for "orders by user" queries
   - An Elasticsearch index for full-text search
   - A Redis cache for real-time dashboard

Each read model is **independently optimised** for its query pattern.

---

### Example: Order System

```
Write side (commands):
  POST /orders → validates → produces event to Kafka

Kafka topic: orders
  → event: { order_id, user_id, items, total, timestamp }

Read side (query models, all built by consuming Kafka):
  Consumer A: builds Postgres table → used by "my orders" page
  Consumer B: builds Elasticsearch → used by search
  Consumer C: builds Redis → used by real-time dashboard
  Consumer D: sends email confirmation
```

If you add a new feature (e.g., loyalty points calculation), you add a new consumer. You don't touch the write side at all.

---

### Eventual Consistency

In CQRS with Kafka, reads are **eventually consistent**. There's a small delay between an event being written and the read model being updated.

- Order is placed → event in Kafka
- Consumer processes event → updates Postgres
- User queries "my orders" → gets updated data

The delay is typically milliseconds to seconds. For most use cases, this is acceptable. For operations that need immediate consistency (e.g., checking if a username is taken), use a different approach.

---

← [[19 - Topic Design]] | [[21 - Dead Letter Queues]] →
