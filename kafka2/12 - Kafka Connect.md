# 12 - Kafka Connect

#kafka #kafka-connect #connectors #integration #ETL

← [[Kafka MOC]]

---

## What is Kafka Connect?

Kafka Connect is a framework built into Kafka for **moving data between Kafka and external systems** — without writing custom producer/consumer code.

Instead of writing code like:

```python
# Custom code to read from Postgres and write to Kafka
conn = psycopg2.connect(...)
rows = conn.execute("SELECT * FROM orders WHERE updated_at > ?")
for row in rows:
    producer.produce("orders", json.dumps(row))
```

You configure a **connector** with a JSON file, and Kafka Connect handles everything.

---

## Core Concepts

### Source Connector
Reads data **from an external system → into Kafka**.

Examples:
- PostgreSQL → Kafka (Debezium CDC connector)
- S3 → Kafka
- HTTP API → Kafka
- MySQL binlog → Kafka

### Sink Connector
Reads data **from Kafka → into an external system**.

Examples:
- Kafka → Elasticsearch
- Kafka → S3
- Kafka → BigQuery
- Kafka → PostgreSQL

```
[PostgreSQL] → Source Connector → [Kafka Topic] → Sink Connector → [Elasticsearch]
```

---

## Why Use Kafka Connect Instead of Custom Code?

| Custom Code | Kafka Connect |
|---|---|
| You handle retries | Built-in retry logic |
| You manage offsets/checkpoints | Managed automatically |
| You scale manually | Scale workers easily |
| You handle schema changes | Schema Registry integration built-in |
| Bespoke per integration | 200+ pre-built connectors |

---

## Architecture

```
Kafka Connect Cluster
┌────────────────────────────────────────────────┐
│  Worker 1          Worker 2         Worker 3   │
│  [task0][task1]    [task2][task3]   [task4]     │
└────────────────────────────────────────────────┘
         │                                 │
    Source System                      Sink System
    (PostgreSQL)                    (Elasticsearch)
```

- **Workers** — JVM processes that run connectors and tasks
- **Connectors** — the high-level config (what system, what topic, how often)
- **Tasks** — the actual parallel workers doing the data transfer

---

## Example: Debezium PostgreSQL CDC Connector

**CDC (Change Data Capture)** streams every INSERT/UPDATE/DELETE from a Postgres DB into Kafka in real time.

```json
{
  "name": "postgres-source",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "localhost",
    "database.port": "5432",
    "database.user": "kafka_user",
    "database.password": "secret",
    "database.dbname": "shop",
    "table.include.list": "public.orders",
    "topic.prefix": "dbserver1"
  }
}
```

Every change to `public.orders` table becomes an event in topic `dbserver1.public.orders`.

---

## Example: S3 Sink Connector

```json
{
  "name": "s3-sink",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "tasks.max": "3",
    "topics": "orders",
    "s3.region": "us-east-1",
    "s3.bucket.name": "my-kafka-archive",
    "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
    "flush.size": "1000"
  }
}
```

---

## Managing Connectors via REST API

Kafka Connect exposes a REST API on port 8083 by default.

```bash
# List all connectors
curl http://localhost:8083/connectors

# Create a connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @connector-config.json

# Check connector status
curl http://localhost:8083/connectors/postgres-source/status

# Restart a connector
curl -X POST http://localhost:8083/connectors/postgres-source/restart

# Delete a connector
curl -X DELETE http://localhost:8083/connectors/postgres-source
```

---

## Popular Connectors

| Connector | Direction | Use Case |
|---|---|---|
| Debezium PostgreSQL | Source | CDC from Postgres |
| Debezium MySQL | Source | CDC from MySQL |
| JDBC Source/Sink | Both | Generic SQL databases |
| S3 Sink | Sink | Archival to object storage |
| Elasticsearch Sink | Sink | Search and analytics |
| BigQuery Sink | Sink | Data warehouse |
| HTTP Source | Source | Pull from REST APIs |
| MQTT Source | Source | IoT device data |

Find more at [Confluent Hub](https://www.confluent.io/hub/).

---

← [[11 - Consumers In Depth]] | [[13 - Kafka Streams]] →
