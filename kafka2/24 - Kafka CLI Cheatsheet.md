# 24 - Kafka CLI Cheatsheet

#kafka #CLI #commands #cheatsheet #debugging

← [[Kafka MOC]]

---

## Connecting to Kafka Inside Docker

```bash
# Get shell inside the Kafka container
docker exec -it kafka bash

# All commands below are run inside this shell
# Or prefix with: docker exec -it kafka <command>
```

---

## Topics

```bash
# List all topics
kafka-topics.sh --bootstrap-server localhost:9092 --list

# Create a topic
kafka-topics.sh --bootstrap-server localhost:9092 \
  --create \
  --topic orders \
  --partitions 3 \
  --replication-factor 1

# Describe a topic (partitions, replicas, leader)
kafka-topics.sh --bootstrap-server localhost:9092 \
  --describe \
  --topic orders

# Describe ALL topics
kafka-topics.sh --bootstrap-server localhost:9092 --describe

# Delete a topic
kafka-topics.sh --bootstrap-server localhost:9092 \
  --delete \
  --topic orders

# List under-replicated partitions (should return nothing in healthy cluster)
kafka-topics.sh --bootstrap-server localhost:9092 \
  --describe \
  --under-replicated-partitions

# Alter topic (change partition count — can only increase)
kafka-topics.sh --bootstrap-server localhost:9092 \
  --alter \
  --topic orders \
  --partitions 6

# Alter topic config
kafka-topics.sh --bootstrap-server localhost:9092 \
  --alter \
  --topic orders \
  --config retention.ms=86400000
```

---

## Producing Messages (Console Producer)

```bash
# Produce to a topic (type message and press Enter, Ctrl+C to stop)
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders

# Produce with a key (key:value format)
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --property "key.separator=:" \
  --property "parse.key=true"
# Then type: user-123:{"order_id":"abc","item":"Pizza"}
```

---

## Consuming Messages (Console Consumer)

```bash
# Consume new messages (from now)
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders

# Consume from beginning
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --from-beginning

# Consume and show key + value
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --from-beginning \
  --property print.key=true \
  --property key.separator=" | "

# Consume from specific partition and offset
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --partition 0 \
  --offset 42

# Consume as part of a consumer group
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --group my-test-group \
  --from-beginning

# Limit number of messages
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --from-beginning \
  --max-messages 10
```

---

## Consumer Groups

```bash
# List all consumer groups
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list

# Describe a consumer group (shows lag per partition)
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group order-tracker-group \
  --describe

# Describe ALL consumer groups
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --all-groups

# Reset offsets to beginning (replay all events)
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group order-tracker-group \
  --topic orders \
  --reset-offsets \
  --to-earliest \
  --execute

# Reset offsets to specific offset
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group order-tracker-group \
  --topic orders:0 \
  --reset-offsets \
  --to-offset 100 \
  --execute

# Delete a consumer group (must be inactive)
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --delete \
  --group order-tracker-group
```

---

## Cluster and Broker Info

```bash
# List broker API versions (also confirms connectivity)
kafka-broker-api-versions.sh \
  --bootstrap-server localhost:9092

# Describe cluster (broker IDs, controller)
kafka-metadata-quorum.sh \
  --bootstrap-server localhost:9092 \
  describe --status  # KRaft mode
```

---

## Performance Testing

```bash
# Producer performance test (1M messages, 1KB each)
kafka-producer-perf-test.sh \
  --topic test-perf \
  --num-records 1000000 \
  --record-size 1000 \
  --throughput -1 \
  --producer-props bootstrap.servers=localhost:9092

# Consumer performance test
kafka-consumer-perf-test.sh \
  --bootstrap-server localhost:9092 \
  --topic test-perf \
  --messages 1000000
```

---

## Log Dirs (Disk Usage)

```bash
# Check disk usage per topic/partition on each broker
kafka-log-dirs.sh \
  --bootstrap-server localhost:9092 \
  --topic-list orders \
  --describe
```

---

## Quick Reference Summary

| Task | Command prefix |
|---|---|
| Manage topics | `kafka-topics.sh` |
| Produce messages | `kafka-console-producer.sh` |
| Consume messages | `kafka-console-consumer.sh` |
| Consumer groups + lag | `kafka-consumer-groups.sh` |
| Performance test | `kafka-producer-perf-test.sh` / `kafka-consumer-perf-test.sh` |
| Cluster info (KRaft) | `kafka-metadata-quorum.sh` |
| ACLs | `kafka-acls.sh` |
| Disk usage | `kafka-log-dirs.sh` |

---

← [[23 - Python + Kafka (confluent-kafka)]] | [[Kafka MOC]] →
