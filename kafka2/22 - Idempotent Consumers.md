# 22 - Idempotent Consumers

#kafka #idempotent #exactly-once #duplicates #reliability

← [[Kafka MOC]]

---

## Why Duplicates Happen

With **at-least-once** delivery (the most common Kafka setting), a consumer may process the same event more than once:

**Scenario 1: Consumer crashes after processing, before committing offset**
```
1. Consumer reads offset 42
2. Consumer processes the event (charges credit card ✅)
3. Consumer CRASHES before committing offset
4. Consumer restarts, reads offset 42 again
5. Consumer processes the event AGAIN (charges credit card again ❌)
```

**Scenario 2: Network timeout on commit**
```
1. Consumer reads and processes offset 42
2. Consumer commits offset 42
3. Network times out — consumer thinks commit failed
4. Consumer retries, processes offset 42 again
5. Commit actually worked — duplicate processing ❌
```

---

## What is Idempotency?

An operation is **idempotent** if performing it multiple times produces the same result as performing it once.

```
Idempotent:     SET user.email = 'alice@example.com'
                (running it 10x = same as running it 1x)

NOT idempotent: INSERT INTO orders VALUES (...)
                (running it 10x = 10 duplicate rows)

NOT idempotent: charge_credit_card(user_id, amount)
                (running it 10x = 10 charges)
```

Your consumer must be idempotent to safely handle duplicates.

---

## Strategy 1: Natural Idempotency (Best)

Some operations are naturally idempotent. Design your systems to use them where possible.

```python
# Idempotent: UPSERT by order_id
db.execute("""
    INSERT INTO orders (order_id, user_id, item, status)
    VALUES (%s, %s, %s, %s)
    ON CONFLICT (order_id) DO UPDATE
    SET status = EXCLUDED.status
""", [order_id, user_id, item, status])
# Running this 10x = same result as running 1x ✅
```

---

## Strategy 2: Deduplication Table

Track which event IDs have already been processed. Skip if already seen.

```python
def process_event(order: dict):
    order_id = order["order_id"]

    # Check if already processed
    already_processed = db.execute(
        "SELECT 1 FROM processed_events WHERE event_id = %s",
        [order_id]
    ).fetchone()

    if already_processed:
        print(f"⏭️ Skipping duplicate: {order_id}")
        return

    # Process the event
    charge_customer(order)
    send_confirmation_email(order)

    # Mark as processed
    db.execute(
        "INSERT INTO processed_events (event_id, processed_at) VALUES (%s, NOW())",
        [order_id]
    )
```

**Important:** The check + process + mark should ideally be in a **database transaction** to avoid race conditions.

---

## Strategy 3: Transactional Outbox

For operations that must be atomic (process + mark as done together):

```python
with db.transaction():
    # Do the work
    db.execute("UPDATE inventory SET stock = stock - 1 WHERE item_id = %s", [item_id])

    # Mark event as processed in same transaction
    db.execute(
        "INSERT INTO processed_events (event_id) VALUES (%s) ON CONFLICT DO NOTHING",
        [event_id]
    )

# If transaction commits → both happened
# If transaction rolls back → neither happened
# Either way, running again is safe
```

---

## Strategy 4: Versioning / Optimistic Locking

For state updates, use a version number to prevent stale writes:

```python
# Event contains a version
order_event = {"order_id": "abc", "status": "shipped", "version": 3}

# Only update if current version matches expected
result = db.execute("""
    UPDATE orders SET status = %s, version = %s
    WHERE order_id = %s AND version = %s
""", ["shipped", 4, "abc", 3])

if result.rowcount == 0:
    print("Already updated (duplicate event) — skipping")
```

---

## Kafka's Idempotent Producer (Different Thing)

Note: the **idempotent producer** setting (`enable.idempotence=True`) handles **producer-side duplicates** — preventing the same event from being written to Kafka twice due to retries.

This is different from **consumer-side idempotency** — which handles the same event being processed twice by a consumer.

You need **both** for a truly robust system.

---

## Summary

| Strategy | When to Use |
|---|---|
| Natural idempotency (upsert) | When DB operations can be designed as upserts |
| Deduplication table | When events have unique IDs and you need to track processing |
| Transactional processing | When processing + marking must be atomic |
| Versioning | When updating state with version tracking |

> **Golden rule:** Always assume your consumer will see a message more than once. Design accordingly.

---

← [[21 - Dead Letter Queues]] | [[23 - Python + Kafka (confluent-kafka)]] →
