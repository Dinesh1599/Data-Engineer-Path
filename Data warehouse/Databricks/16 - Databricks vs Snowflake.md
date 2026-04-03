
# Databricks vs Snowflake

> [!info] Related notes
> [[01 - What is Databricks]] | [[06 - Storage Optimization]]

## Head-to-Head

| Dimension | Databricks | Snowflake |
|-----------|-----------|-----------|
| Architecture | Lakehouse (open files on your storage) | Cloud warehouse (managed storage) |
| Storage format | Open Parquet + Delta log | Proprietary (FDN) |
| File management | Manual (OPTIMIZE, VACUUM) | Fully automatic |
| Compute | Spark clusters (you configure) | Warehouses (T-shirt sizes) |
| Streaming | Strong (Structured Streaming) | Limited (Snowpipe) |
| ML support | Native (MLlib, GPU) | Snowpark ML (newer) |
| Cost transparency | Complex (DBU + VM + storage) | Simple (credits) |
| Vendor lock-in | Low (open formats) | High (proprietary) |

## Optimization Concepts Mapped

| Concept | Databricks | Snowflake |
|---------|-----------|-----------|
| File compaction | [[06 - Storage Optimization#OPTIMIZE|OPTIMIZE]] (manual) | Automatic |
| Data sorting | [[06 - Storage Optimization#Z-ORDER|Z-ORDER]] / Liquid Clustering | Clustering Keys + Auto Clustering |
| Dead file cleanup | [[06 - Storage Optimization#VACUUM|VACUUM]] (manual) | Automatic |
| Query inspection | Spark UI + `.explain()` | Query Profile |
| Caching | Manual (`.cache()`) | Automatic (3-layer) |
| Join optimization | Broadcast hint, AQE | Automatic |

## When to Choose Which

**Choose Databricks when:**
- ETL/ELT-heavy with complex transformations (PySpark)
- Machine learning is a core use case
- Open formats and no vendor lock-in matter
- Streaming with complex event processing
- Your team has strong Spark/Python skills

**Choose Snowflake when:**
- SQL analytics and BI is the primary use case
- Zero infrastructure management is priority
- SQL-first team with limited Spark experience
- Cost predictability matters most
- Easy cross-organization data sharing needed

> [!tip] Interview one-liner
> "Snowflake is the best data warehouse. Databricks is the best data platform. For ETL-heavy workloads with open formats → Databricks. For SQL-first analytics with zero maintenance → Snowflake."

---

← Back to [[00 - Index]]
