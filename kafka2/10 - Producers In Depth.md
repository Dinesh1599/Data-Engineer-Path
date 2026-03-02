# 10 - Producers In Depth

#kafka #producers #batching #compression #acks #callbacks

← [[kafka2/Kafka]]

---

## Producer Lifecycle

```
1. Create producer with config
2. Serialize the message (key + value → bytes)
3. Determine partition (via key hash, round-robin, or custom)
4. Add to internal buffer (batch)
5. Background thread sends batch to broker
6. Broker acknowledges (based on acks setting)
7. Delivery callback fires (success or failure)
```

---

## Basic Configuration

```python
from confluent_kafka import Producer

config = {
    "bootstrap.servers": "localhost:9092",  # Kafka broker address
    "acks": "all",                          # wait for all ISR replicas
    "enable.idempotence": True,             # prevent duplicate sends on retry
    "compression.type": "snappy",           # compress batches
    "linger.ms": 5,                         # wait 5ms to fill a batch
    "batch.size": 16384,                    # max bytes per batch (16KB)
}

producer = Producer(config)
```

---

## Batching

Producers don't send each event immediately — they **buffer** events and send in batches. This dramatically improves throughput.

Two settings control batching:

| Setting | Meaning |
|---|---|
| `batch.size` | Max bytes in a batch before it's sent (default: 16KB) |
| `linger.ms` | How long to wait for more events before sending (default: 0ms) |

```
linger.ms = 0:   send immediately (low latency, small batches)
linger.ms = 10:  wait 10ms to collect more events (higher throughput)
```

**Trade-off:** Higher `linger.ms` → better throughput, higher latency.

---

## Compression

Kafka can compress batches before sending to brokers.

```python
config = {"compression.type": "snappy"}  # or lz4, gzip, zstd
```

| Codec | Compression | Speed | Best For |
|---|---|---|---|
| `none` | None | Fastest | Low latency needs |
| `gzip` | High | Slow | Archival, cold data |
| `snappy` | Medium | Fast | General use |
| `lz4` | Medium | Very fast | High throughput |
| `zstd` | High | Fast | Best balance (modern) |

Compression is transparent — consumers decompress automatically.

---

## The `flush()` Method

Producers buffer events in memory. If your program exits without flushing, **buffered events are lost**.

```python
producer.produce(topic="orders", value=order_bytes)
producer.flush()  # block until all buffered events are delivered
```

Always call `flush()` before your producer program exits.

---

## Delivery Callbacks

A callback fires when Kafka confirms delivery (or reports an error). Essential for reliability.

```python
def delivery_report(err, msg):
    if err:
        print(f"❌ Delivery failed: {err}")
    else:
        print(f"✅ Delivered to {msg.topic()} "
              f"partition {msg.partition()} "
              f"offset {msg.offset()}")

producer.produce(
    topic="orders",
    value=order_bytes,
    callback=delivery_report
)

# Trigger delivery callbacks
producer.poll(0)  # non-blocking
# or
producer.flush()  # blocking until all sent
```

---

## Retries

If a send fails (transient network error), the producer can retry automatically.

```python
config = {
    "retries": 5,                  # retry up to 5 times
    "retry.backoff.ms": 100,       # wait 100ms between retries
    "delivery.timeout.ms": 120000, # total time to give up (2 min)
}
```

With `enable.idempotence = True`, retries are safe — Kafka deduplicates them.

---

## Key Serialization

Kafka only accepts **bytes**. You must serialise your data.

```python
import json

order = {"order_id": "abc-123", "item": "Pizza", "qty": 2}

# Serialise to bytes
value_bytes = json.dumps(order).encode("utf-8")
key_bytes = "user-123".encode("utf-8")

producer.produce(
    topic="orders",
    key=key_bytes,
    value=value_bytes,
    callback=delivery_report
)
```

For structured schemas across teams, use Avro + Schema Registry (see [[14 - Schema Registry and Avro]]).

---

## Producer Config Quick Reference

| Config | Default | Notes |
|---|---|---|
| `bootstrap.servers` | — | Required |
| `acks` | `1` | Use `all` for safety |
| `enable.idempotence` | `false` | Set `true` for safe retries |
| `compression.type` | `none` | Use `snappy` or `zstd` |
| `linger.ms` | `0` | Increase for throughput |
| `batch.size` | `16384` | Bytes per batch |
| `retries` | `2147483647` (idempotent) | Lower without idempotence |
| `delivery.timeout.ms` | `120000` | 2 minutes total |

---

← [[09 - Retention Policies]] | [[11 - Consumers In Depth]] →
