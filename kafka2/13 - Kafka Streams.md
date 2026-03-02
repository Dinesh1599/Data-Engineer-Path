# 13 - Kafka Streams

#kafka #kafka-streams #stream-processing #real-time

← [[kafka2/Kafka]]

---

## What is Kafka Streams?

Kafka Streams is a **Java/Scala library** (built into Kafka) for writing stream processing applications. It reads events from topics, transforms them, aggregates them, and writes results back to topics — in real time.

Think of it as a way to do data pipeline logic **inside Kafka**, without a separate processing engine.

```
[orders topic] → [Kafka Streams App] → [order-totals topic]
                     (filter, enrich,
                      aggregate, join)
```

---

## Kafka Streams vs Regular Consumer

| Regular Consumer | Kafka Streams |
|---|---|
| Read events, do custom logic | Read, transform, and write in pipelines |
| No built-in windowing | Time-based windowing built in |
| Manual state management | Built-in stateful stores |
| No topology abstraction | Declarative processing topology |
| Simple use cases | Complex streaming pipelines |

---

## Core Operations

### Filter
```java
stream.filter((key, value) -> value.getAmount() > 100)
```

### Map (transform)
```java
stream.mapValues(order -> new EnrichedOrder(order, lookupUser(order.userId)))
```

### GroupBy and Count
```java
stream.groupBy((key, value) -> value.getCategory())
      .count()
      .toStream()
      .to("category-counts");
```

### Join Two Streams
```java
orders.join(payments,
    (order, payment) -> new CompletedOrder(order, payment),
    JoinWindows.of(Duration.ofMinutes(5))
)
```

---

## Windowing

Windowing lets you aggregate events over a **time window**.

### Tumbling Window (non-overlapping)
```java
// Count orders per minute, in 1-minute buckets
stream.groupByKey()
      .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(1)))
      .count()
```

```
|--window1--|--window2--|--window3--|
   0-60s      60-120s    120-180s
```

### Hopping Window (overlapping)
```java
// 5-minute window, advancing every 1 minute
TimeWindows.ofSizeAndGrace(Duration.ofMinutes(5), Duration.ofSeconds(30))
           .advanceBy(Duration.ofMinutes(1))
```

### Session Window (activity-based)
Groups events that are close together, with a gap indicating a new session.
```java
SessionWindows.ofInactivityGapWithNoGrace(Duration.ofMinutes(30))
```

---

## State Stores

Kafka Streams can maintain local state (key-value stores backed by RocksDB).

```java
// Materialise a count into a queryable store
KTable<String, Long> orderCounts =
    stream.groupByKey().count(Materialized.as("order-counts-store"));

// Query the store interactively
ReadOnlyKeyValueStore<String, Long> store =
    streams.store(StoreQueryParameters.fromNameAndType(
        "order-counts-store",
        QueryableStoreTypes.keyValueStore()
    ));

Long count = store.get("user-123");
```

The state store is **backed by a changelog topic** in Kafka, so state survives application restarts.

---

## Kafka Streams vs Apache Flink / Spark Streaming

| Feature | Kafka Streams | Flink / Spark |
|---|---|---|
| Language | Java/Scala only | Java, Python, SQL, Scala |
| Deployment | Embedded in your app | Separate cluster |
| Complexity | Lower | Higher |
| Throughput | High | Very high |
| Use case | Kafka-native pipelines | Large-scale complex analytics |

Use Kafka Streams when your data is already in Kafka and you want simple-to-moderate processing. Use Flink/Spark for complex, large-scale analytics.

---

## KSQL / ksqlDB

ksqlDB is a SQL layer on top of Kafka Streams. If you prefer SQL over Java code:

```sql
-- Create a stream from a topic
CREATE STREAM orders (order_id VARCHAR, user_id VARCHAR, amount DOUBLE)
  WITH (KAFKA_TOPIC='orders', VALUE_FORMAT='JSON');

-- Filter orders over $100
CREATE STREAM high_value_orders AS
  SELECT * FROM orders WHERE amount > 100;

-- Count orders per user per hour
CREATE TABLE orders_per_user AS
  SELECT user_id, COUNT(*) as order_count
  FROM orders
  WINDOW TUMBLING (SIZE 1 HOUR)
  GROUP BY user_id;
```

---

← [[12 - Kafka Connect]] | [[14 - Schema Registry and Avro]] →
