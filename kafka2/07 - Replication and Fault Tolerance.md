# 07 - Replication and Fault Tolerance

#kafka #replication #fault-tolerance #ISR #reliability

← [[kafka2/Kafka]]

---

## Why Replication?

If Kafka stored each partition on only one broker, losing that broker means losing that data — permanently. Replication solves this by keeping **multiple copies** of each partition across different brokers.

---

## Replication Factor

The **replication factor** defines how many copies of a partition exist across brokers.

```
Topic: orders, Replication Factor: 3, 3 Brokers

Partition 0:
  Broker 1 → Leader (primary copy, handles reads/writes)
  Broker 2 → Follower (replica)
  Broker 3 → Follower (replica)
```

**Rule of thumb:**
- Dev/local: replication factor = 1
- Production: replication factor = 3 (minimum)
- This means you can lose 2 brokers and still have data

---

## Leader and Followers

For every partition:
- **One broker is the Leader** — producers write to it, consumers read from it (by default)
- **Other brokers are Followers** — they continuously replicate from the leader

Followers don't serve client requests. They exist purely as backups.

If the leader broker crashes:
1. Kafka detects the failure (via KRaft or ZooKeeper)
2. An in-sync follower is promoted to leader automatically
3. Producers and consumers continue without manual intervention

---

## In-Sync Replicas (ISR)

The **ISR (In-Sync Replicas)** is the set of replicas that are fully caught up with the leader.

```
Partition 0 Leader: Broker 1
ISR: [Broker 1, Broker 2, Broker 3]  ← all three are in sync
```

If a follower falls too far behind (network issue, slow disk), it's **removed from the ISR**:

```
ISR: [Broker 1, Broker 2]  ← Broker 3 fell behind, removed from ISR
```

Only ISR members can be elected as the new leader. This prevents data loss from a stale replica becoming leader.

---

## `offsets.topic.replication.factor`

This setting controls replication for Kafka's internal `__consumer_offsets` topic (which tracks consumer positions).

```yaml
KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1  # dev/single broker
# Default is 3, appropriate for production clusters
```

**Important:** If you run a single broker and don't set this to 1, Kafka will error because it can't create 3 replicas with only 1 broker.

---

## `min.insync.replicas`

This setting defines the **minimum number of replicas that must acknowledge a write** before it's considered successful.

```
Topic config: min.insync.replicas = 2

Write from producer:
  ✅ Leader (Broker 1) acknowledges
  ✅ Follower (Broker 2) acknowledges
  ❌ Follower (Broker 3) is down → doesn't matter, we have 2
→ Write succeeds ✅
```

But if only 1 replica is available and `min.insync.replicas = 2`, **the write is rejected**. This prevents silent data loss.

Works in conjunction with producer `acks` setting (see [[10 - Producers In Depth]]).

---

## Unclean Leader Election

What if **all ISR members** die, but an out-of-sync follower is still alive?

- **`unclean.leader.election.enable = false`** (default): Kafka waits for an ISR member to come back. No data loss, but topic is temporarily unavailable.
- **`unclean.leader.election.enable = true`**: Kafka elects the stale follower as leader. Topic becomes available, but **you may lose events** that the stale follower didn't have.

> **Recommendation:** Keep this `false` in production. Data loss is usually worse than brief downtime.

---

## Rack Awareness

In cloud environments (AWS, GCP, etc.), brokers run across **availability zones**. Kafka can be configured to spread replicas across AZs (racks), so a full AZ outage doesn't lose data.

```
Partition 0:
  Broker 1 (us-east-1a) → Leader
  Broker 2 (us-east-1b) → Follower
  Broker 3 (us-east-1c) → Follower
```

Configure with `broker.rack` setting per broker.

---

## Summary

| Setting | Purpose |
|---|---|
| `replication.factor` | How many copies of each partition |
| `min.insync.replicas` | Min replicas that must confirm a write |
| `offsets.topic.replication.factor` | Replication for consumer offset tracking |
| `unclean.leader.election.enable` | Whether to allow stale replicas to become leader |

---

← [[06 - KRaft Mode]] | [[08 - Delivery Semantics]] →
