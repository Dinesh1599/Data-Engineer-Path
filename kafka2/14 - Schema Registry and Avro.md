# 14 - Schema Registry and Avro

#kafka #schema-registry #avro #serialisation #schema-evolution

← [[Kafka MOC]]

---

## The Problem: Schemaless Chaos

If every team produces JSON events however they like, you get:

```json
// Team A's order event
{"order_id": "abc", "userId": "u123", "amount": 29.99}

// Team B's order event (different field names!)
{"orderId": "def", "user_id": "u456", "price": 19.99}
```

Consumers break because field names changed. A missing field causes a null pointer exception. A type change (string → int) causes a parse error. You have no idea what a topic contains without reading the code.

**Schema Registry + Avro solves this.**

---

## What is Apache Avro?

**Avro** is a compact binary serialisation format with a defined schema. Instead of sending human-readable JSON, you send compact bytes described by a schema.

```json
// Avro schema (orders.avsc)
{
  "type": "record",
  "name": "Order",
  "namespace": "com.streamstore",
  "fields": [
    {"name": "order_id", "type": "string"},
    {"name": "user_id",  "type": "string"},
    {"name": "item",     "type": "string"},
    {"name": "quantity", "type": "int"},
    {"name": "amount",   "type": "double"}
  ]
}
```

Benefits over JSON:
- **Smaller** — binary encoding, no field names in the payload
- **Faster** — less data to send and parse
- **Schema-enforced** — can't accidentally send wrong types

---

## What is Schema Registry?

**Schema Registry** (by Confluent) is a centralized service that stores and versions all your Avro (or JSON Schema / Protobuf) schemas.

```
Producer ──(schema v1)──▶ Schema Registry
                               │
                          validates &
                          assigns ID
                               │
Producer ──(ID + bytes)──▶ Kafka Topic

Consumer ──(ID)──▶ Schema Registry ──(schema v1)──▶ Consumer
Consumer uses schema to deserialise bytes
```

Each message only carries a **schema ID** (4 bytes), not the full schema. Consumers look up the schema by ID when needed.

---

## Schema Evolution

Schemas change over time. Schema Registry enforces **compatibility rules** to prevent breaking changes.

### Backward Compatible (default)
New schema can read data written with old schema. Consumers can be updated before producers.
```json
// Old schema
{"name": "order_id", "type": "string"}

// New schema (add optional field with default)
{"name": "order_id",  "type": "string"},
{"name": "promo_code","type": ["null","string"], "default": null}  ← safe addition
```

### Forward Compatible
Old schema can read data written with new schema. Producers can be updated before consumers.

### Full Compatible
Both backward and forward compatible. Most restrictive but safest.

### Breaking Change (NOT allowed by default)
```json
// Renaming a field — breaks consumers expecting the old name
{"name": "orderId", "type": "string"}  // was "order_id" — BREAKING ❌
```

---

## Python Example with Avro + Schema Registry

```python
from confluent_kafka import Producer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer
from confluent_kafka.serialization import SerializationContext, MessageField

# Schema
schema_str = """
{
  "type": "record",
  "name": "Order",
  "fields": [
    {"name": "order_id", "type": "string"},
    {"name": "item",     "type": "string"},
    {"name": "quantity", "type": "int"}
  ]
}
"""

# Schema Registry client
sr_client = SchemaRegistryClient({"url": "http://localhost:8081"})

# Serialiser
avro_serializer = AvroSerializer(sr_client, schema_str)

# Producer
producer = Producer({"bootstrap.servers": "localhost:9092"})

order = {"order_id": "abc-123", "item": "Pizza", "quantity": 2}

producer.produce(
    topic="orders",
    value=avro_serializer(
        order,
        SerializationContext("orders", MessageField.VALUE)
    )
)
producer.flush()
```

---

## Alternatives to Avro

| Format | Notes |
|---|---|
| **Avro** | Most popular with Kafka, compact binary |
| **Protobuf** | Google's format, great language support |
| **JSON Schema** | JSON with schema validation, human-readable |
| **Plain JSON** | Simple, no schema enforcement, verbose |

Schema Registry supports all three (Avro, Protobuf, JSON Schema).

---

← [[13 - Kafka Streams]] | [[15 - Deploying Kafka]] →
