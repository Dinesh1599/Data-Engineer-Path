# 11 - Consumers In Depth

#kafka #consumers #polling #rebalancing #offset-management

← [[Kafka MOC]]

---

## Consumer Lifecycle

```
1. Create consumer with config (group.id required)
2. Subscribe to one or more topics
3. Poll loop — ask Kafka for new events
4. Deserialise the message bytes
5. Process the event
6. Commit the offset
7. Repeat
8. On shutdown: close() to release resources
```

---

## Basic Configuration

```python
from confluent_kafka import Consumer

config = {
    "bootstrap.servers": "localhost:9092",
    "group.id": "order-tracker-group",       # consumer group name
    "auto.offset.reset": "earliest",          # start from beginning if new
    "enable.auto.commit": False,              # manual commit (safer)
}

consumer = Consumer(config)
consumer.subscribe(["orders"])
```

---

## The Poll Loop

Consumers **pull** from Kafka. Kafka never pushes to consumers.

```python
try:
    while True:
        msg = consumer.poll(timeout=1.0)  # wait up to 1s for a message

        if msg is None:
            continue  # no new events, keep polling

        if msg.error():
            print(f"❌ Error: {msg.error()}")
            continue

        # Deserialise
        value = json.loads(msg.value().decode("utf-8"))
        print(f"📦 Received: {value}")

        # Commit offset after successful processing
        consumer.commit(msg)

except KeyboardInterrupt:
    print("🛑 Stopping consumer...")

finally:
    consumer.close()  # always close cleanly
```

---

## `consumer.close()` — Why It Matters

Closing the consumer properly:
1. **Commits any pending offsets** — prevents re-processing on restart
2. **Triggers a rebalance** — tells Kafka "I'm leaving, reassign my partitions"
3. **Releases broker connections** — prevents resource leaks

Always call `close()` in a `finally` block or with a context manager.

---

## Manual vs Auto Offset Commit

### Auto Commit (simpler, riskier)
```python
config = {
    "enable.auto.commit": True,
    "auto.commit.interval.ms": 5000  # every 5 seconds
}
```
Risk: Consumer crashes between auto-commits → events may be processed twice OR missed depending on timing.

### Manual Commit (safer)
```python
config = {"enable.auto.commit": False}

# After processing:
consumer.commit(msg)              # synchronous (blocks)
consumer.commit(msg, asynchronous=True)  # async (faster, less safe)
```

**Synchronous commit** blocks until Kafka confirms. Slower but guarantees the offset is saved.

**Asynchronous commit** returns immediately. Faster, but if it fails silently, you may re-process events.

---

## Subscribing vs Assigning

### Subscribe (recommended — automatic partition assignment)
```python
consumer.subscribe(["orders", "payments"])
```
Kafka handles partition assignment within the consumer group. Supports rebalancing.

### Assign (manual — specific partition)
```python
from confluent_kafka import TopicPartition
consumer.assign([TopicPartition("orders", 0)])  # only partition 0
```
Bypasses consumer groups and rebalancing. Used for specific use cases like replaying a specific partition.

---

## Rebalancing in Detail

A rebalance is triggered when:
- A new consumer joins the group
- A consumer leaves or crashes
- New partitions are added to a subscribed topic
- A consumer fails to poll within `max.poll.interval.ms`

During a rebalance, **all consumers stop processing**. This is called a **stop-the-world rebalance**.

Kafka 2.4+ introduced **incremental cooperative rebalancing** which minimises disruption — only the partitions that need to move are reassigned, others continue processing.

Enable with:
```python
config = {
    "partition.assignment.strategy": "cooperative-sticky"
}
```

---

## Heartbeating

Consumers send **heartbeats** to Kafka to signal they're alive.

| Config | Default | Meaning |
|---|---|---|
| `heartbeat.interval.ms` | 3000 | How often to send heartbeat |
| `session.timeout.ms` | 45000 | If no heartbeat in this time → consumer is dead |
| `max.poll.interval.ms` | 300000 | Max time between polls before consumer is removed from group |

If your consumer takes a long time to process each event (e.g., calling an external API), increase `max.poll.interval.ms` to avoid being kicked out of the group.

---

## Consumer Lag

**Consumer lag** is how far behind a consumer is — the difference between the latest offset in a partition and the consumer's committed offset.

```
Latest offset in partition: 1000
Consumer's committed offset:  950
Lag: 50 events behind
```

High lag = consumer is struggling to keep up. Monitor this in production — see [[17 - Monitoring and Observability]].

---

## Deserialisation

Kafka delivers bytes. Convert them back to usable data:

```python
# JSON
value = json.loads(msg.value().decode("utf-8"))
key   = msg.key().decode("utf-8") if msg.key() else None

# Avro (with schema registry)
# See [[14 - Schema Registry and Avro]]
```

---

← [[10 - Producers In Depth]] | [[12 - Kafka Connect]] →
