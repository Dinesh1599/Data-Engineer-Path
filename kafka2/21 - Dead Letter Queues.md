# 21 - Dead Letter Queues

#kafka #DLQ #dead-letter #error-handling #poison-pill

← [[Kafka MOC]]

---

## The Problem: Poison Pill Messages

A **poison pill** is an event that a consumer cannot process — it causes an error every time. This can happen because:

- Malformed JSON (serialisation error)
- A field that's null when the code expects a value
- A downstream service is unreachable
- Business logic rejects the event (e.g., invalid order)

Without a strategy, the consumer gets stuck: it keeps retrying the same broken event, blocking all subsequent events in that partition.

```
Partition 0:
  offset 5: {"order_id": "abc", ...}  ← valid, processed ✅
  offset 6: {"order_id": null, ...}   ← poison pill, crashes consumer ❌
  offset 7: {"order_id": "def", ...}  ← blocked, never processed ❌
  offset 8: {"order_id": "ghi", ...}  ← blocked, never processed ❌
```

---

## Solution: Dead Letter Queue (DLQ)

A **Dead Letter Queue** (also called Dead Letter Topic in Kafka) is a separate topic where you send events that could not be processed after a set number of retries.

```
Normal flow:
  [orders] → [consumer] → process event ✅

Error flow:
  [orders] → [consumer] → fails 3 times → [orders.DLQ] → consumer continues
```

The DLQ holds failed events for:
- Manual inspection
- Alerting
- Reprocessing after the bug is fixed

---

## Implementing a DLQ in Python

```python
from confluent_kafka import Consumer, Producer
import json

consumer = Consumer({
    "bootstrap.servers": "localhost:9092",
    "group.id": "order-processor",
    "auto.offset.reset": "earliest",
    "enable.auto.commit": False,
})

dlq_producer = Producer({"bootstrap.servers": "localhost:9092"})

consumer.subscribe(["orders"])
MAX_RETRIES = 3
retry_counts = {}

while True:
    msg = consumer.poll(1.0)
    if msg is None:
        continue
    if msg.error():
        continue

    event_key = f"{msg.topic()}-{msg.partition()}-{msg.offset()}"
    retry_counts.setdefault(event_key, 0)

    try:
        order = json.loads(msg.value().decode("utf-8"))

        # Simulate processing that might fail
        if order.get("order_id") is None:
            raise ValueError("Missing order_id")

        print(f"✅ Processed order: {order['order_id']}")
        consumer.commit(msg)
        del retry_counts[event_key]

    except Exception as e:
        retry_counts[event_key] += 1

        if retry_counts[event_key] >= MAX_RETRIES:
            print(f"❌ Sending to DLQ after {MAX_RETRIES} retries: {e}")
            dlq_producer.produce(
                topic="orders.DLQ",
                key=msg.key(),
                value=msg.value(),
                headers={
                    "original-topic": msg.topic(),
                    "original-partition": str(msg.partition()),
                    "original-offset": str(msg.offset()),
                    "error": str(e),
                    "retry-count": str(retry_counts[event_key]),
                }
            )
            dlq_producer.flush()
            consumer.commit(msg)  # commit to move past poison pill
            del retry_counts[event_key]
        else:
            print(f"⚠️ Retry {retry_counts[event_key]}/{MAX_RETRIES}: {e}")
```

---

## DLQ Headers

Always include metadata in DLQ messages so you know what happened:

| Header | Value |
|---|---|
| `original-topic` | Where the event came from |
| `original-partition` | Which partition |
| `original-offset` | Which offset |
| `error` | The exception message |
| `retry-count` | How many times it was retried |
| `failed-at` | Timestamp of failure |
| `consumer-group` | Which consumer group failed |

---

## Processing the DLQ

Two approaches:

### 1. Manual Review
An operator inspects the DLQ, fixes the data, and manually re-publishes corrected events to the original topic.

### 2. Automated Reprocessing
After a bug fix is deployed, a separate consumer reads from the DLQ and republishes events to the original topic:

```python
# DLQ reprocessor
consumer.subscribe(["orders.DLQ"])
while True:
    msg = consumer.poll(1.0)
    if msg:
        # Strip DLQ headers, re-publish to original topic
        producer.produce(
            topic=msg.headers().get("original-topic"),
            key=msg.key(),
            value=msg.value()
        )
```

---

## DLQ Naming Convention

```
<original-topic>.DLQ
<original-topic>.dead-letter
<original-topic>-dlq

# With retry stages
orders.retry.1
orders.retry.2
orders.retry.3
orders.dead-letter
```

Some systems use multiple retry topics with increasing delays before the final DLQ.

---

← [[20 - Event Sourcing and CQRS]] | [[22 - Idempotent Consumers]] →
