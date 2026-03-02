# 18 - Performance Tuning

#kafka #performance #throughput #latency #tuning

← [[Kafka MOC]]

---

## The Core Trade-off: Throughput vs Latency

Almost every Kafka tuning decision is a trade-off between:
- **Throughput** — how many events per second you can process
- **Latency** — how quickly a single event gets from producer to consumer

Optimise one, and you typically sacrifice the other.

---

## Producer Tuning

### For Maximum Throughput
```python
config = {
    "linger.ms": 20,           # wait 20ms to fill batches
    "batch.size": 65536,       # 64KB batches (default 16KB)
    "compression.type": "lz4", # compress batches
    "acks": "1",               # only leader confirmation
}
```

### For Minimum Latency
```python
config = {
    "linger.ms": 0,     # send immediately
    "batch.size": 1,    # don't batch
    "acks": "1",        # don't wait for replicas
}
```

### For Safety (Production Recommended)
```python
config = {
    "linger.ms": 5,
    "batch.size": 32768,        # 32KB
    "compression.type": "zstd", # best compression ratio
    "acks": "all",              # all ISR confirm
    "enable.idempotence": True,
}
```

---

## Consumer Tuning

### Fetch Settings
```python
config = {
    "fetch.min.bytes": 1024,      # wait for at least 1KB before returning
    "fetch.max.wait.ms": 500,     # but don't wait more than 500ms
    "max.poll.records": 500,      # max events per poll() call
}
```

- `fetch.min.bytes` high → fewer fetch requests, higher throughput, slightly higher latency
- `max.poll.records` — tune this based on how long processing takes. If each event takes 10ms, 500 events = 5 seconds per poll loop

### Parallelism
More consumers in a group = more parallelism. But max useful consumers = number of partitions.

```bash
# Scale up to 5 consumer instances (Docker/Kubernetes)
docker compose up --scale order-tracker=5
```

---

## Broker Tuning

### OS and Disk
- Kafka is extremely I/O intensive. Use **SSDs** in production.
- Use **XFS or ext4** filesystem (not NFS, not ZFS without tuning)
- Give Kafka **dedicated disks** — don't share with OS
- Increase open file descriptor limit: `ulimit -n 100000`

### JVM Heap
```bash
# Set in kafka-server-start.sh or KAFKA_HEAP_OPTS env var
export KAFKA_HEAP_OPTS="-Xmx6g -Xms6g"
```
Recommended: 4-8 GB for most workloads. Don't go above 8 GB or GC pauses become a problem (Kafka uses OS page cache heavily, more RAM for OS = better).

### Key Broker Configs
```properties
# Number of threads for network requests
num.network.threads=8

# Number of threads for disk I/O
num.io.threads=16

# Socket send/receive buffer sizes
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400

# Log flush (usually let OS handle it)
log.flush.interval.messages=10000
log.flush.interval.ms=1000
```

---

## Partition Count and Throughput

More partitions = more parallelism = higher throughput. But too many partitions has costs:

| More Partitions | Trade-offs |
|---|---|
| Higher throughput | More open file handles per broker |
| More consumer parallelism | Longer leader election on broker failure |
| Better load distribution | More memory for replication |

**Guideline:** A single partition can handle ~10–100 MB/s. If you need 1 GB/s, you need 10-100 partitions.

---

## Page Cache

Kafka's secret weapon for performance: it relies heavily on the **OS page cache** rather than heap memory.

When a consumer reads recent events, the OS often serves them from RAM (page cache) rather than disk — making reads near-instant.

**Implication:** For best performance, provision enough RAM so recently written data stays in page cache. If page cache is too small, every read becomes a disk seek.

---

## Compression Comparison

| Codec | Compression Ratio | CPU Cost | Best For |
|---|---|---|---|
| none | 1x | Lowest | Already compressed data |
| gzip | 3-4x | High | Archival, low throughput |
| snappy | 2-3x | Low | Balanced (legacy) |
| lz4 | 2-3x | Very Low | High throughput |
| zstd | 3-4x | Medium | Best overall (modern) |

**Recommendation:** Use `zstd` for new systems. It achieves gzip-level compression at near-lz4 speed.

---

## Benchmarking

Kafka ships with built-in benchmark tools:

```bash
# Producer throughput test
kafka-producer-perf-test.sh \
  --topic test \
  --num-records 1000000 \
  --record-size 1000 \
  --throughput -1 \  # -1 = no throttling
  --producer-props bootstrap.servers=localhost:9092

# Consumer throughput test
kafka-consumer-perf-test.sh \
  --bootstrap-server localhost:9092 \
  --topic test \
  --messages 1000000
```

---

← [[17 - Monitoring and Observability]] | [[19 - Topic Design]] →
