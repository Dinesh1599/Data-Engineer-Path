# 23 - Python + Kafka (confluent-kafka)

#kafka #python #confluent-kafka #code #hands-on

← [[kafka2/Kafka]]

---

## Setup

```bash
pip install confluent-kafka
```

For Avro support:
```bash
pip install confluent-kafka[avro]
# or
pip install confluent-kafka fastavro
```

---

## Full Producer Example

```python
# producer.py
import json
import uuid
from confluent_kafka import Producer


def delivery_report(err, msg):
    """Callback fired when a message is delivered or fails."""
    if err:
        print(f"❌ Delivery failed: {err}")
    else:
        print(
            f"✅ Delivered to {msg.topic()} "
            f"[partition {msg.partition()}] "
            f"at offset {msg.offset()}"
        )


def main():
    config = {
        "bootstrap.servers": "localhost:9092",
        "acks": "all",
        "enable.idempotence": True,
        "compression.type": "snappy",
        "linger.ms": 5,
    }

    producer = Producer(config)

    order = {
        "order_id": str(uuid.uuid4()),
        "user": "nana_tech",
        "item": "Mushroom Pizza",
        "quantity": 2,
    }

    value_bytes = json.dumps(order).encode("utf-8")
    key_bytes = order["user"].encode("utf-8")  # partition by user

    producer.produce(
        topic="orders",
        key=key_bytes,
        value=value_bytes,
        callback=delivery_report,
    )

    # Trigger delivery callbacks
    producer.poll(0)

    # Flush before exiting — don't leave buffered events unsent
    producer.flush()
    print("Producer finished.")


if __name__ == "__main__":
    main()
```

---

## Full Consumer Example

```python
# consumer.py
import json
import signal
from confluent_kafka import Consumer, KafkaException


running = True

def shutdown(sig, frame):
    global running
    print("🛑 Shutdown signal received...")
    running = False

signal.signal(signal.SIGTERM, shutdown)
signal.signal(signal.SIGINT, shutdown)


def process_order(order: dict):
    """Your business logic here."""
    print(
        f"📦 New order received: "
        f"{order['quantity']}x {order['item']} "
        f"from {order['user']}"
    )


def main():
    config = {
        "bootstrap.servers": "localhost:9092",
        "group.id": "order-tracker-group",
        "auto.offset.reset": "earliest",
        "enable.auto.commit": False,  # manual commit
    }

    consumer = Consumer(config)
    consumer.subscribe(["orders"])
    print("🟢 Consumer running, subscribed to 'orders'...")

    try:
        while running:
            msg = consumer.poll(timeout=1.0)

            if msg is None:
                continue  # no new events

            if msg.error():
                raise KafkaException(msg.error())

            # Deserialise
            order = json.loads(msg.value().decode("utf-8"))

            # Process
            process_order(order)

            # Commit offset after successful processing
            consumer.commit(msg)

    except KafkaException as e:
        print(f"❌ Kafka error: {e}")

    finally:
        # Always close cleanly — triggers rebalance and flushes offsets
        consumer.close()
        print("Consumer closed cleanly.")


if __name__ == "__main__":
    main()
```

---

## Reading Partition and Offset Info

```python
# From a consumed message:
msg.topic()       # → "orders"
msg.partition()   # → 0
msg.offset()      # → 42
msg.key()         # → b"nana_tech"
msg.value()       # → b'{"order_id": "abc", ...}'
msg.timestamp()   # → (TIMESTAMP_TYPE, unix_ms)
msg.headers()     # → [("key", b"value"), ...]
```

---

## Manual Partition Assignment

```python
from confluent_kafka import TopicPartition

# Assign specific partition (bypasses consumer group)
consumer.assign([TopicPartition("orders", partition=0)])

# Seek to specific offset
consumer.seek(TopicPartition("orders", partition=0, offset=100))

# Seek to beginning
consumer.seek_to_beginning(TopicPartition("orders", 0))
```

---

## Getting Topic Metadata

```python
metadata = producer.list_topics("orders")
topic_info = metadata.topics["orders"]
partitions = topic_info.partitions
print(f"Topic 'orders' has {len(partitions)} partitions")
```

---

## Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `KafkaException: Local: Unknown topic` | Topic doesn't exist and auto-create is off | Create topic first via CLI or enable auto-create |
| `KafkaException: Broker: Not enough in-sync replicas` | ISR size < min.insync.replicas | Reduce min.insync.replicas or fix your cluster |
| `KafkaException: Local: Queue full` | Producer buffer full | Increase `queue.buffering.max.messages` or slow down production |
| Consumer keeps rebalancing | Processing too slow, exceeds `max.poll.interval.ms` | Reduce `max.poll.records` or increase `max.poll.interval.ms` |
| Consumer reads from beginning every restart | Not committing offsets | Check `enable.auto.commit` and manual commit logic |

---

## Environment Variables for Config

Best practice: don't hardcode broker addresses.

```python
import os

config = {
    "bootstrap.servers": os.getenv("KAFKA_BOOTSTRAP_SERVERS", "localhost:9092"),
    "group.id": os.getenv("KAFKA_GROUP_ID", "my-consumer-group"),
}
```

---

← [[22 - Idempotent Consumers]] | [[24 - Kafka CLI Cheatsheet]] →
