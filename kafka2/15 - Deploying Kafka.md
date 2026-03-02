# 15 - Deploying Kafka

#kafka #deployment #docker #kubernetes #cloud

← [[kafka2/Kafka]]

---

## Deployment Options Overview

| Option | Best For |
|---|---|
| Docker Compose | Local development |
| Kubernetes (Strimzi/Helm) | Self-managed production |
| Confluent Cloud | Managed cloud (easiest) |
| AWS MSK | AWS-native managed Kafka |
| Aiven | Multi-cloud managed Kafka |
| Redpanda | Kafka-compatible, no JVM |

---

## 1. Docker Compose (Local Dev)

The fastest way to get Kafka running locally. Uses KRaft (no ZooKeeper needed).

```yaml
# docker-compose.yml
version: "3.8"
services:
  kafka:
    image: confluentinc/cp-kafka:7.8.0
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_KRAFT_MODE: "true"
      KAFKA_CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: "broker,controller"
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka:9093"
      KAFKA_LISTENERS: "PLAINTEXT://:9092,CONTROLLER://:9093"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://localhost:9092"
      KAFKA_CONTROLLER_LISTENER_NAMES: "CONTROLLER"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
    volumes:
      - kafka-data:/var/lib/kafka/data

volumes:
  kafka-data:
```

```bash
docker compose up -d      # start in background
docker compose down       # stop
docker compose down -v    # stop AND delete volumes (wipe data)
```

---

## 2. Docker Compose with UI (Kafka UI)

Add a web UI for inspecting topics, messages, consumer groups:

```yaml
services:
  kafka:
    # ... same as above ...

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
    depends_on:
      - kafka
```

Access at `http://localhost:8080`

---

## 3. Kubernetes with Strimzi

**Strimzi** is the most popular Kubernetes operator for Kafka. It manages Kafka clusters as Kubernetes custom resources.

```yaml
# kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 3.8.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
    config:
      offsets.topic.replication.factor: 3
      default.replication.factor: 3
    storage:
      type: persistent-claim
      size: 100Gi
  zookeeper:   # or use KRaft in newer Strimzi versions
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
```

```bash
kubectl apply -f kafka-cluster.yaml
```

---

## 4. Managed Cloud Services

### Confluent Cloud
- Fully managed by Confluent (Kafka's original creators)
- Schema Registry, ksqlDB, Kafka Connect included
- Pricing: per CKU (Confluent Kafka Unit)
- Best for: teams that want zero operational overhead

### AWS MSK (Managed Streaming for Kafka)
- AWS-native, integrates with IAM, VPC, CloudWatch
- You manage scaling, but AWS handles broker health
- No Schema Registry included (bring your own)
- Best for: teams already deep in AWS

### Aiven
- Multi-cloud (AWS, GCP, Azure)
- Includes monitoring, backups, Schema Registry
- Best for: cloud-agnostic teams

---

## Multi-Broker Production Setup

For a production cluster with 3 brokers:

```yaml
# 3 separate broker services
services:
  kafka-1:
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://kafka-1:9092"

  kafka-2:
    environment:
      KAFKA_NODE_ID: 2
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://kafka-2:9092"

  kafka-3:
    environment:
      KAFKA_NODE_ID: 3
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093"
      KAFKA_ADVERTISED_LISTENERS: "PLAINTEXT://kafka-3:9092"
```

---

## Redpanda — Kafka-Compatible Alternative

Redpanda is a Kafka-compatible streaming platform written in C++ (not Java). It's faster to start, uses less memory, and has no JVM overhead. Drop-in replacement for Kafka — same APIs, same tools.

```bash
docker run -p 9092:9092 redpandadata/redpanda:latest \
  redpanda start --overprovisioned --smp 1 --memory 1G
```

Good for: local dev where you want something even lighter than Kafka.

---

← [[14 - Schema Registry and Avro]] | [[16 - Security]] →
