# 03 - Brokers and Clusters

#kafka #brokers #clusters #infrastructure

← [[Kafka MOC]]

---

## What is a Broker?

A **broker** is simply a Kafka server. It:
- Receives events from producers
- Stores events on disk
- Serves events to consumers
- Manages topic partitions assigned to it

Brokers are the physical storage layer of Kafka. Without brokers, there is no Kafka.

---

## What is a Kafka Cluster?

A **cluster** is a group of brokers working together. In production, you always run multiple brokers for:

1. **Fault tolerance** — if one broker dies, others keep serving data
2. **Scalability** — more brokers = more storage and throughput
3. **Load distribution** — partitions are spread across brokers

```
Kafka Cluster
┌──────────────────────────────────┐
│  [Broker 1]  [Broker 2]  [Broker 3]  │
│      │            │           │      │
│   partition0  partition1  partition2  │
│   (leader)    (leader)    (leader)   │
└──────────────────────────────────┘
```

---

## Cluster ID and Node ID

Every cluster has a **Cluster ID** — a unique string identifier shared by all brokers in that cluster.

Every individual broker has a **Node ID** — a unique integer within the cluster (1, 2, 3...).

```yaml
# Docker Compose example
KAFKA_CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"  # arbitrary unique string
KAFKA_NODE_ID: 1                               # integer, unique per broker
```

---

## The Controller

Within a cluster, one broker holds the **Controller** role. The controller is the cluster manager. It:

- Tracks which broker is the **leader** for each partition
- Handles **leader elections** when a broker fails
- Manages **partition reassignment**
- Handles all cluster administration tasks

At any time, **only one broker is the active controller**. If the controller broker crashes, another broker is automatically elected as the new controller.

In [[06 - KRaft Mode]], this coordination is managed internally. In older Kafka, it required Zookeeper.

---

## How Data is Distributed Across Brokers

Topics are split into **partitions** (see [[04 - Partitions]]), and each partition lives on a specific broker.

Example: Topic `orders` with 3 partitions across 3 brokers:

```
Broker 1: orders-partition-0 (leader)
Broker 2: orders-partition-1 (leader)
Broker 3: orders-partition-2 (leader)
```

Plus replicas on other brokers for fault tolerance (see [[07 - Replication and Fault Tolerance]]).

---

## Broker Configuration (Key Settings)

| Config | What it does |
|---|---|
| `listeners` | Which ports Kafka opens (client port, controller port) |
| `advertised.listeners` | The address clients are told to connect to |
| `log.dirs` | Where Kafka stores data on disk |
| `num.partitions` | Default number of partitions for new topics |
| `default.replication.factor` | Default replicas for new topics |
| `log.retention.hours` | How long to keep events (see [[09 - Retention Policies]]) |

---

## Single Broker vs Multi-Broker

| | Single Broker | Multi-Broker Cluster |
|---|---|---|
| Use case | Local dev, testing | Production |
| Fault tolerance | None | Yes |
| Replication factor | 1 | 2 or 3 |
| Scalability | Limited | Horizontal |

> **Rule of thumb:** Never run a single broker in production. Minimum 3 brokers for a production cluster, which allows replication factor of 3 and tolerates one broker failure.

---

← [[02 - Core Concepts]] | [[04 - Partitions]] →
