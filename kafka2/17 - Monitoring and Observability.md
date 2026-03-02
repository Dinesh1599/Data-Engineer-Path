# 17 - Monitoring and Observability

#kafka #monitoring #observability #consumer-lag #metrics #JMX

← [[Kafka MOC]]

---

## What to Monitor in Kafka

Good Kafka monitoring answers three questions:
1. **Is Kafka healthy?** — broker metrics
2. **Is data flowing?** — producer/consumer throughput
3. **Are consumers keeping up?** — consumer lag

---

## 1. Consumer Lag (Most Important Metric)

**Consumer lag** = latest offset in partition − consumer's committed offset

```
Partition 0:
  Latest offset:    10000
  Consumer at:       9950
  Lag:                50   ← 50 events behind
```

High lag means consumers are falling behind producers. This can lead to:
- Delayed processing (emails sent late, inventory not updated in time)
- Disk space pressure if retention is short and lag grows past it
- Eventually: events are deleted before they're consumed

**Check lag with CLI:**
```bash
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group order-tracker-group \
  --describe
```

Output:
```
GROUP               TOPIC  PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
order-tracker-group orders 0          9950            10000           50
```

---

## 2. Broker Metrics (via JMX)

Kafka exposes metrics via **JMX (Java Management Extensions)**. Key broker metrics:

| Metric | What it tells you |
|---|---|
| `kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec` | Incoming message rate |
| `kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec` | Incoming byte rate |
| `kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec` | Outgoing byte rate |
| `kafka.controller:type=KafkaController,name=ActiveControllerCount` | Should always be 1 |
| `kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions` | Should be 0 |
| `kafka.server:type=ReplicaManager,name=OfflineReplicaCount` | Should be 0 |
| `kafka.network:type=RequestMetrics,name=RequestsPerSec` | Request throughput |

**`UnderReplicatedPartitions > 0`** is a red flag — replicas are falling behind, you're losing redundancy.

---

## 3. Producer Metrics

| Metric | What it tells you |
|---|---|
| `record-send-rate` | Events sent per second |
| `record-error-rate` | Failed sends per second (should be 0) |
| `request-latency-avg` | Average time to get acknowledgement |
| `buffer-available-bytes` | If near 0, producer is back-pressuring |

---

## Monitoring Tools

### Kafka UI (Open Source, Lightweight)
```yaml
# docker-compose.yml addition
kafka-ui:
  image: provectuslabs/kafka-ui:latest
  ports:
    - "8080:8080"
  environment:
    KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
```
View topics, messages, consumer groups, consumer lag at `http://localhost:8080`.

### Conduktor (Commercial + Free tier)
GUI for Kafka management. Good for teams. Includes consumer lag monitoring, data masking, alerts.

### Confluent Control Center (Confluent Platform)
Full-featured monitoring for Confluent Platform. Enterprise-grade but not free.

### Prometheus + Grafana (Production Standard)
Export Kafka JMX metrics via **JMX Exporter**, scrape with Prometheus, visualise in Grafana.

```yaml
# Add JMX exporter to your Kafka container
KAFKA_JMX_PORT: 9999
KAFKA_JMX_HOSTNAME: kafka
```

Import the official **Kafka Grafana dashboards** (ID: 7589 or search Grafana Labs).

### Datadog / New Relic / Dynatrace
Commercial APM tools with native Kafka integrations. Easy to set up, expensive at scale.

---

## Alerting — What to Alert On

| Alert | Threshold | Severity |
|---|---|---|
| Consumer lag | > 10,000 events or growing | Warning / Critical |
| UnderReplicatedPartitions | > 0 | Critical |
| ActiveControllerCount | ≠ 1 | Critical |
| Broker disk usage | > 80% | Warning |
| Producer error rate | > 0 | Warning |
| Broker offline | Any | Critical |

---

## Health Check Commands

```bash
# List all consumer groups and their lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --all-groups

# Check topic details
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic orders

# Check under-replicated partitions
kafka-topics.sh --bootstrap-server localhost:9092 --describe --under-replicated-partitions

# Check broker metadata
kafka-broker-api-versions.sh --bootstrap-server localhost:9092
```

---

← [[16 - Security]] | [[18 - Performance Tuning]] →
