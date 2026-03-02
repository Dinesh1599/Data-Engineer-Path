# 06 - KRaft Mode

#kafka #kraft #zookeeper #metadata #cluster-coordination

← [[Kafka MOC]]

---

## The Problem KRaft Solves

Before Kafka 3.x, Kafka required an external tool called **Apache ZooKeeper** to manage cluster metadata and coordination. ZooKeeper handled:
- Tracking which brokers are alive
- Storing topic and partition metadata
- Electing the controller broker
- Storing consumer group offsets (older versions)

This meant you had to **run and maintain two separate systems** (Kafka + ZooKeeper) just to run Kafka. That added complexity, extra failure points, and scaling limitations.

```
Old Architecture:
[ZooKeeper Ensemble]  ←→  [Kafka Cluster]
  (separate system)         (your system)
```

---

## What is KRaft?

**KRaft** (Kafka Raft) is Kafka's built-in metadata management system, introduced in Kafka 2.8 and made production-ready in Kafka 3.3. It replaces ZooKeeper entirely.

The name comes from the **Raft consensus algorithm** — a well-known protocol for distributed systems to agree on a shared state even when some nodes fail.

```
New Architecture (KRaft):
[Kafka Cluster — brokers + controller built-in]
  (single system, no ZooKeeper needed)
```

---

## How KRaft Works

In KRaft mode, a subset of brokers take on the **controller role** (in addition to or instead of the broker role). These controllers:

- Store all cluster metadata internally in a special topic called `__cluster_metadata`
- Use the Raft protocol to elect an **active controller** (leader)
- Keep passive controllers (followers) in sync as backups
- If the active controller crashes, followers elect a new one automatically

### Node Roles in KRaft

A Kafka node in KRaft mode can have one or more roles:

| Role | Description |
|---|---|
| `broker` | Stores data, serves producers and consumers |
| `controller` | Manages cluster metadata and coordination |
| `broker,controller` | Both roles (common in small clusters / local dev) |

```yaml
# Setting roles in Docker Compose
KAFKA_PROCESS_ROLES: "broker,controller"  # single node does both
```

---

## KRaft Configuration (Key Settings)

```yaml
KAFKA_KRAFT_MODE: "true"
KAFKA_CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"  # unique per cluster
KAFKA_NODE_ID: 1                               # unique per broker
KAFKA_PROCESS_ROLES: "broker,controller"

# Who the controllers are and where to reach them
KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka:9093"
# Format: nodeId@hostname:port
# Multiple: "1@kafka1:9093,2@kafka2:9093,3@kafka3:9093"

# Ports
KAFKA_LISTENERS: "PLAINTEXT://:9092,CONTROLLER://:9093"
KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://localhost:9092"
KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
```

---

## Controller Quorum

In a multi-broker cluster with multiple controllers, they form a **quorum** — a voting group.

- **Active controller** — the current leader, makes all decisions
- **Passive controllers** — follow along, ready to take over, participate in votes

A quorum of `N` controllers can tolerate `(N-1)/2` failures:
- 3 controllers → tolerates 1 failure
- 5 controllers → tolerates 2 failures

For a single-node dev setup:
```yaml
KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka:9093"  # just one voter
```

---

## ZooKeeper vs KRaft

| Feature | ZooKeeper Mode | KRaft Mode |
|---|---|---|
| External dependency | Yes (ZooKeeper) | No |
| Operational complexity | Higher | Lower |
| Metadata scaling | Limited (~200k partitions) | Millions of partitions |
| Recovery speed | Slower | Faster |
| Production ready | Yes (legacy) | Yes (Kafka 3.3+) |
| Status | Deprecated | Recommended |

> **Bottom line:** Always use KRaft for new setups. ZooKeeper support was officially deprecated in Kafka 3.5 and removed in Kafka 4.0.

---

## The `__cluster_metadata` Topic

In KRaft mode, all cluster metadata (broker list, topic configs, partition assignments) is stored in an internal Kafka topic called `__cluster_metadata`. This is:

- Replicated across all controller nodes
- An append-only log (just like regular Kafka topics)
- Managed entirely by Kafka — you don't interact with it directly

This is elegant: Kafka uses its own log mechanism to manage itself.

---

← [[05 - Consumer Groups and Offsets]] | [[07 - Replication and Fault Tolerance]] →
