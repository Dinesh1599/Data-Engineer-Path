# 08 - Delivery Semantics

#kafka #delivery #guarantees #exactly-once #idempotent

← [[Kafka MOC]]

---

## The Core Question

When a producer sends an event to Kafka, and a consumer processes it — **how many times** is that event guaranteed to be processed?

Networks fail. Brokers crash. Consumers die mid-process. Kafka offers three answers:

---

## 1. At-Most-Once

> The event is processed **zero or one times**. It may be lost, but never duplicated.

**How it works:**
- Producer sends event → does NOT wait for acknowledgement
- Consumer commits offset **before** processing the event
- If consumer crashes after committing but before processing → event is skipped forever

**When to use:** When losing some data is acceptable and you absolutely cannot tolerate duplicates. Rare in practice. Example: high-volume metrics where losing a few data points is fine.

```python
# Producer: fire and forget (acks=0)
producer_config = {"acks": 0}

# Consumer: commit before processing
consumer.commit()
process(message)
```

---

## 2. At-Least-Once

> The event is processed **one or more times**. It may be duplicated, but never lost.

**How it works:**
- Producer sends event → waits for acknowledgement from broker
- Consumer processes event **first**, then commits offset
- If consumer crashes after processing but before committing → it re-reads and re-processes the event

**This is the most common setting** in production Kafka systems.

```python
# Producer: wait for leader acknowledgement
producer_config = {"acks": "1"}  # or "all" for stronger guarantee

# Consumer: process first, then commit
message = consumer.poll(1.0)
process(message)        # do work first
consumer.commit(message)  # then commit offset
```

**Implication:** Your consumer logic must be **idempotent** — processing the same event twice should produce the same result as processing it once. See [[22 - Idempotent Consumers]].

---

## 3. Exactly-Once

> The event is processed **exactly one time**. No loss, no duplicates.

This is the hardest to achieve and requires coordination at both the producer and consumer level.

### Exactly-Once at the Producer (Idempotent Producer)
Enable with `enable.idempotence = true`. Kafka assigns each producer a **Producer ID (PID)** and sequence numbers to detect and discard duplicate sends.

```python
producer_config = {
    "enable.idempotence": True,
    "acks": "all"  # required with idempotence
}
```

If a producer retries a send (network timeout), Kafka uses the sequence number to recognise it's a duplicate and ignores it.

### Exactly-Once Semantics (EOS) — End to End
Full exactly-once requires **Kafka Transactions**. The producer wraps its sends in a transaction, and the consumer only reads **committed** transactions.

```python
producer.init_transactions()
producer.begin_transaction()
producer.produce(topic="output", value=result)
producer.send_offsets_to_transaction(offsets, consumer_group)
producer.commit_transaction()
```

This ensures the output event and the offset commit happen **atomically** — either both happen or neither does.

**Use cases:** Financial systems, billing, inventory deduction — anywhere duplicate processing causes real-world harm.

---

## Producer Acknowledgements (`acks`)

The `acks` setting controls how many brokers must confirm receipt before the producer considers a send successful.

| `acks` value | Meaning | Risk |
|---|---|---|
| `0` | No acknowledgement waited | Highest data loss risk |
| `1` | Leader broker confirms | Data lost if leader crashes before replication |
| `all` (or `-1`) | All ISR replicas confirm | Safest, slightly slower |

Combine with `min.insync.replicas` on the broker side for strongest guarantees.

---

## Choosing Your Semantic

| Scenario | Recommended |
|---|---|
| Metrics, logs (some loss ok) | At-most-once |
| Order processing, emails | At-least-once + idempotent consumer |
| Financial transactions, billing | Exactly-once |
| Most production systems | At-least-once |

> **Practical advice:** Start with at-least-once and make your consumers idempotent. Exactly-once adds complexity and overhead. Only use it if duplicates genuinely cause harm.

---

← [[07 - Replication and Fault Tolerance]] | [[09 - Retention Policies]] →
