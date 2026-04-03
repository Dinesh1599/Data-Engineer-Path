---
tags: [databricks, cost, storage-tiering, spot-instances]
aliases: [Cost Optimization, Storage Tiering]
---

# Cost Optimization

> [!info] Related notes
> [[06 - Storage Optimization]] | [[09 - Compute and Clusters]]

## Storage Tiering (ADLS)

| Tier | Cost/GB/month | Savings | Access | Use for |
|------|--------------|---------|--------|---------|
| **Hot** | $0.0184 | Baseline | Instant | Active Delta tables (Silver, Gold) |
| **Cool** | $0.0100 | ~45% | Instant | Bronze raw files after 30 days |
| **Cold** | $0.0036 | ~83% | Instant | Historical exports after 90 days |
| **Archive** | $0.0020 | ~90% | Hours (rehydrate) | 7-year regulatory retention |

> [!danger] Never tier Delta tables to Archive
> Spark needs instant file access. Only tier raw landing zone files and historical exports. Configure via Storage Account → Data Management → Lifecycle Management (or Terraform).

## Compute Cost Control

- **[[09 - Compute and Clusters#Job Clusters|Job clusters]]** for pipelines (auto-terminate, zero idle cost)
- **[[09 - Compute and Clusters#Spot Instances|Spot instances]]** for workers (60-80% savings)
- **Auto-terminate** on all interactive clusters (30 min)
- **Right-size** clusters (don't use 20 nodes for 500MB)
- **Photon engine** for SQL-heavy workloads (2-8x faster)

## ELT vs Storage Optimization

| | ELT Optimization | Storage Optimization |
|---|---|---|
| **What** | Making pipeline code faster | Keeping files healthy |
| **Examples** | Incremental loads, broadcast joins, predicate pushdown | OPTIMIZE, Z-ORDER, VACUUM, tiering |
| **When** | During pipeline development | Scheduled maintenance |
| **Effect** | Pipeline runs faster | Queries read faster, storage costs drop |

Both matter. They're different jobs that help each other.

---

**Next:** [[15 - Debugging Slow Queries]] →
