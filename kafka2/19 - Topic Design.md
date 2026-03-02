# 19 - Topic Design

#kafka #topic-design #naming #architecture #best-practices

← [[Kafka MOC]]

---

## Topic Design Matters

Poor topic design is one of the most common Kafka mistakes. Unlike a database schema, topic design is **hard to change** after you've started producing data. Get it right (or at least thoughtful) upfront.

---

## Naming Conventions

No official standard, but consistency within your org matters. Common patterns:

```
# Pattern: <domain>.<entity>.<event-type>
ecommerce.orders.placed
ecommerce.orders.cancelled
ecommerce.payments.completed
ecommerce.inventory.updated

# Pattern: <team>-<entity>-<event>
checkout-team-order-created
payments-team-payment-failed

# Pattern: <environment>.<domain>.<entity>
prod.ecommerce.orders
staging.ecommerce.orders
```

**Rules to follow:**
- Use lowercase (case-sensitive in Kafka)
- Use `.` or `-` as separators (not spaces)
- Be descriptive — `orders` is better than `data`
- Include environment if sharing a cluster across envs
- Don't use special characters

---

## One Topic Per Event Type

Good practice: one topic = one type of event. Don't mix different event types in one topic.

```
❌ Bad: one topic for everything
  topic: ecommerce-events
    → order_placed, payment_done, item_shipped, refund_issued ... all mixed

✅ Good: separate topics per event type
  topic: orders.placed
  topic: orders.cancelled
  topic: payments.completed
  topic: shipments.dispatched
```

Mixing event types makes consumers parse every message to determine its type, and schema management becomes a nightmare.

---

## How Many Partitions?

```
Target throughput per topic
÷ throughput per partition
= minimum number of partitions

Then add headroom: multiply by 1.5–2x
```

**Example:**
- Target: 500 MB/s for `orders` topic
- Each partition handles ~50 MB/s
- 500 / 50 = 10 partitions minimum
- With headroom: 20 partitions

Also consider: **how many consumers** will you run in parallel? Max useful consumers = partitions. If you want 20 consumers, you need 20 partitions.

**Start with more partitions than you think you need.** You can increase partitions later, but:
- Adding partitions does not rebalance existing data
- Key-based partitioning breaks for existing data (new keys go to new partition, old keys don't change)

---

## Partition Keys — Choose Carefully

The key determines which partition an event goes to. Same key → same partition → ordered.

| Key Choice | Effect |
|---|---|
| `user_id` | All events for one user in same partition (ordered per user) |
| `order_id` | All events for one order in same partition |
| `null` | Round-robin, no ordering guarantee |
| `region` | Group events by geography |

**Watch out for hot partitions:** If 80% of your orders come from `user-001` (a bot?), partition 0 gets 80% of the load. Choose keys with high cardinality.

---

## Compacted vs Regular Topics

| Topic Type | Cleanup Policy | Use When |
|---|---|---|
| Regular | `delete` | Event stream (orders, clicks, logs) |
| Compacted | `compact` | Latest state of an entity (user profile, product price) |
| Compacted + Delete | `compact,delete` | Latest state with eventual deletion |

```bash
# Create compacted topic
kafka-topics.sh --create \
  --topic user-profiles \
  --config cleanup.policy=compact \
  --bootstrap-server localhost:9092
```

---

## Internal vs External Topics

| Type | Description |
|---|---|
| `__consumer_offsets` | Kafka internal — consumer offset tracking |
| `__cluster_metadata` | Kafka internal — KRaft metadata |
| `_schemas` | Schema Registry — schema storage |
| `connect-*` | Kafka Connect — connector status |

Don't write to internal topics (prefixed with `_` or `__`). Treat them as read-only.

---

## Topic Configuration Quick Reference

```bash
kafka-topics.sh --create \
  --topic orders \
  --partitions 12 \
  --replication-factor 3 \
  --config retention.ms=604800000 \        # 7 days
  --config cleanup.policy=delete \
  --config min.insync.replicas=2 \
  --bootstrap-server localhost:9092
```

---

## Common Mistakes

| Mistake | Why It's Bad |
|---|---|
| Too few partitions upfront | Can't scale consumers past partition count |
| All events in one topic | Consumers must parse everything; schema chaos |
| Inconsistent naming | Teams can't find/use each other's topics |
| No retention policy | Disk fills up unexpectedly |
| Hot partition keys | One partition gets all the load |
| Deleting and recreating topics to "fix" things | Destroys consumer offset history |

---

← [[18 - Performance Tuning]] | [[20 - Event Sourcing and CQRS]] →
