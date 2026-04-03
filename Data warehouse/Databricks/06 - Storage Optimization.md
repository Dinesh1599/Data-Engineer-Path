---
tags: [databricks, delta-lake, optimize, zorder, vacuum, storage]
aliases: [Storage Optimization, OPTIMIZE, Z-ORDER, VACUUM]
---

# Storage Optimization

> [!info] Related notes
> [[02 - Delta Lake]] | [[05 - Spark Internals]] | [[14 - Cost Optimization]] | [[15 - Debugging Slow Queries]]

## The Small File Problem

The **#1 performance issue** in Delta Lake:

```
200,000 files × 500KB each =
  → 200K Spark tasks (100ms overhead each = 20 sec just to start)
  → 200K ADLS API calls
  → Massive metadata scan
  → 2 sec query takes 45 sec
```

**Root causes:**
1. Too many [[05 - Spark Internals#Partitions|Spark partitions]] for the data (200 shuffle partitions for 5MB)
2. High-frequency streaming micro-batches (10 sec intervals = 720 files/hour)
3. Over-partitioned tables (partition by date with 50 rows/day)
4. UPDATE/MERGE rewrites creating new files without cleanup

## OPTIMIZE (File Compaction)

Reads small files, combines in memory, rewrites as fewer large files (~1GB each):

```sql
OPTIMIZE silver.claims;
```

| | Before | After |
|---|---|---|
| Files | 200,000 × 500KB | ~100 × 1GB |
| Spark tasks | 200,000 | 100 |
| Query planning | 45 seconds | < 1 second |
| Data volume | 100 GB | 100 GB (unchanged) |

> [!info] Analogy
> Repacking 200 half-empty shoeboxes into 2 full boxes. Same stuff, fewer containers.

## Z-ORDER (Data Skipping)

OPTIMIZE compacts files but data inside is randomly mixed. Z-ORDER **sorts the data** so similar values cluster together:

```sql
OPTIMIZE silver.claims ZORDER BY (claim_date, policy_id);
```

```
Without Z-ORDER:
  File 1: NY + CA + TX, all dates mixed → every file's range includes everything
  Query WHERE state = 'NY': must read ALL files (nothing skippable)

With Z-ORDER BY (state):
  File 1: CA claims only (state min=CA, max=CA)
  File 2: NY claims only (state min=NY, max=NY)
  File 3: TX claims only
  Query WHERE state = 'NY': reads ONLY File 2, skips 1 and 3
```

**How it works:** Delta stores min/max stats per column per file. Without Z-ORDER, every file's range spans the full dataset → nothing skipped. With Z-ORDER, ranges are tight → massive skipping.

> [!tip] Z-ORDER rules
> - Z-ORDER on columns you **filter by** (WHERE clause), not columns you SELECT
> - Limit to **2-3 columns max**
> - Z-ORDER runs **during** OPTIMIZE, not separately
> - Z-ORDER does NOT create an index. It reorganizes data inside files.

## Liquid Clustering

Next-gen replacement for Z-ORDER + manual OPTIMIZE. Automatically reorganizes data during writes:

```sql
-- Create with clustering
CREATE TABLE silver.claims (...) CLUSTER BY (state, claim_date);

-- Change clustering columns on existing table (Z-ORDER can't do this)
ALTER TABLE silver.claims CLUSTER BY (claim_date, policy_id);
```

| Feature | Z-ORDER | Liquid Clustering |
|---------|---------|-------------------|
| Manual OPTIMIZE needed? | Yes (scheduled) | No (automatic) |
| Change columns? | No (must rewrite table) | Yes (`ALTER TABLE CLUSTER BY`) |
| Incremental? | No (re-sorts entire table) | Yes (only new data) |
| Works on external tables? | Yes | Yes |

> [!tip] Recommendation
> New tables → Liquid Clustering. Existing Z-ORDER tables → migrate when convenient.
> 
> **Liquid Clustering = optimizeWrite + autoCompact + Z-ORDER all in one.**

## VACUUM (Dead File Cleanup)

The **only** operation that physically deletes old files from ADLS:

```sql
VACUUM silver.claims RETAIN 168 HOURS;  -- 7 days
```

**Beyond storage cleanup, VACUUM also:**
1. **GDPR/CCPA compliance** — permanently removes PII from deleted records
2. **Faster query planning** — fewer dead file references in the transaction log
3. **Reduces ADLS API costs** — fewer files to LIST
4. **Prevents stale cache errors** — removes files that cached queries might reference

> [!warning] VACUUM limits time travel
> After `VACUUM RETAIN 168 HOURS`, you can only time-travel back 7 days. Older versions fail with "file not found."

## autoOptimize (Prevention)

Prevent small files instead of cleaning up after:

```sql
ALTER TABLE silver.claims SET TBLPROPERTIES (
  'delta.autoOptimize.optimizeWrite' = 'true',
  'delta.autoOptimize.autoCompact'  = 'true'
);
```

| Property | What it does | When | Overhead |
|----------|-------------|------|----------|
| `optimizeWrite` | Repartitions before writing → fewer, larger files | During the write | 3-5 sec |
| `autoCompact` | Mini-OPTIMIZE after write if small files accumulated | After the write | 10-30 sec background |

**Why you need both:** `optimizeWrite` prevents ONE write from creating 200 tiny files. `autoCompact` prevents 200 SEPARATE writes from accumulating 200 small files. One prevents bad writes, the other cleans up accumulation.

> [!warning] autoOptimize does NOT sort data
> You still need Z-ORDER or Liquid Clustering for [[07 - Query Optimization#Predicate Pushdown|data skipping]]. autoOptimize only handles file sizes.

## Complete strategy for production

```sql
-- Prevention (enable once per table)
ALTER TABLE silver.claims SET TBLPROPERTIES (
  'delta.autoOptimize.optimizeWrite' = 'true',
  'delta.autoOptimize.autoCompact' = 'true'
);

-- Sorting (still needed on top of autoOptimize)
-- Option A: Scheduled Z-ORDER
OPTIMIZE silver.claims ZORDER BY (claim_date, state);  -- weekly

-- Option B: Liquid Clustering (replaces EVERYTHING above)
ALTER TABLE silver.claims CLUSTER BY (claim_date, state);

-- Cleanup (schedule monthly)
VACUUM silver.claims RETAIN 168 HOURS;
```

---

**Next:** [[07 - Query Optimization]] →
