# 09 - Retention Policies

#kafka #retention #storage #log-compaction

← [[Kafka MOC]]

---

## Kafka Persists Everything (By Default)

Unlike traditional message queues (RabbitMQ, SQS) where messages disappear after consumption, **Kafka keeps every event on disk** for a configurable period. This is one of Kafka's key differentiators.

This means:
- A consumer can re-read old events (replay)
- Multiple consumer groups can read the same event at different times
- You can add a new consumer and have it process all historical events

---

## Retention by Time

The default retention policy: delete events older than a set duration.

```properties
# Keep events for 7 days (default is 168 hours = 7 days)
log.retention.hours=168

# Or more precisely:
log.retention.ms=604800000  # 7 days in milliseconds
```

You can also configure per-topic:
```bash
kafka-topics.sh --alter --topic orders \
  --config retention.ms=86400000  # 1 day for this topic
```

---

## Retention by Size

Delete old segments when the total size of a partition exceeds a limit.

```properties
log.retention.bytes=1073741824  # 1 GB per partition
```

When **both** time and size are configured, Kafka deletes when **either** limit is reached.

---

## Log Segments

Kafka doesn't store a partition as one giant file. It splits it into **segments** — smaller files, typically 1 GB each (configurable).

```
Partition 0 on disk:
  segment-00000000000000000000.log  (offsets 0-99999)
  segment-00000000000000100000.log  (offsets 100000-199999)
  segment-00000000000000200000.log  (offsets 200000-...) ← active segment
```

The **active segment** is the one currently being written to. Retention only applies to **closed (inactive) segments**. So an active segment is never deleted mid-write.

---

## Log Compaction

An alternative retention strategy to time/size-based deletion.

**Log compaction** keeps the **latest value for each key**, discarding older values for the same key.

```
Before compaction:
  key: user-123 → {"name": "Alice", "city": "NYC"}     (offset 0)
  key: user-456 → {"name": "Bob", "city": "LA"}        (offset 1)
  key: user-123 → {"name": "Alice", "city": "Chicago"} (offset 2)  ← latest

After compaction:
  key: user-456 → {"name": "Bob", "city": "LA"}        (offset 1)
  key: user-123 → {"name": "Alice", "city": "Chicago"} (offset 2)
```

**Use case:** Maintaining the latest state of an entity. Perfect for:
- User profile updates
- Product price updates
- Configuration changes

Enable with:
```properties
log.cleanup.policy=compact  # instead of default "delete"
```

You can combine both:
```properties
log.cleanup.policy=compact,delete  # compact AND delete old compacted data after retention period
```

---

## Tombstone Events (Deleting with Compaction)

To delete a key from a compacted topic, produce an event with that key and **a null value**. This is called a **tombstone**.

```python
producer.produce(topic="user-profiles", key="user-123", value=None)
```

Kafka will eventually remove both the tombstone and any prior values for that key.

---

## Retention Settings Summary

| Config | Description | Default |
|---|---|---|
| `log.retention.hours` | Delete events older than N hours | 168 (7 days) |
| `log.retention.ms` | Same as above, in milliseconds | — |
| `log.retention.bytes` | Delete when partition exceeds N bytes | -1 (unlimited) |
| `log.cleanup.policy` | `delete`, `compact`, or `compact,delete` | `delete` |
| `log.segment.bytes` | Max size of a single segment file | 1 GB |

---

## Kafka vs Traditional Message Queues — Retention Comparison

| | Kafka | RabbitMQ / SQS |
|---|---|---|
| Message after consumption | Still on disk | Deleted |
| Replay old events | Yes | No |
| Multiple consumers same message | Yes (different groups) | No (one consumer gets it) |
| Retention control | Time or size | After acknowledgement |

> **Analogy:** Traditional queues are like TV broadcasts — miss it and it's gone. Kafka is like Netflix — it stays there, you can watch anytime, or rewatch.

---

← [[08 - Delivery Semantics]] | [[10 - Producers In Depth]] →
