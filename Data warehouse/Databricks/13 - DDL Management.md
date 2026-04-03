---
tags: [databricks, ddl, schema, migration, production]
aliases: [DDL Management, Schema Migration]
---

# DDL Management in Production

> [!info] Related notes
> [[12 - CICD for Databricks]] | [[04 - Unity Catalog]]

## Migration Script Pattern

```
migrations/
  V001__create_bronze_claims.sql
  V002__create_silver_claims.sql
  V003__add_column_claim_type.sql
  V004__create_gold_summary.sql
```

**Rules:**
- All DDL is version-controlled in Git
- Every statement uses `IF NOT EXISTS` (idempotent, safe to re-run)
- Sequential numbering ensures dependency order
- A tracking table records applied migrations

```sql
-- V001__create_bronze_claims.sql
CREATE TABLE IF NOT EXISTS bronze.claims (
  claim_id STRING, policy_id STRING, amount DOUBLE
) USING DELTA
LOCATION 'abfss://datalake@exl.dfs.core.windows.net/bronze/claims';

-- V003__add_column_claim_type.sql
ALTER TABLE silver.claims ADD COLUMNS IF NOT EXISTS (
  claim_type STRING COMMENT 'auto, property, liability'
);
```

## Per-Environment Promotion

Same migrations run in same order across dev → staging → prod via [[04 - Unity Catalog|Unity Catalog]] catalogs:

```sql
USE CATALOG ${environment}_catalog;  -- dev_catalog, staging_catalog, prod_catalog
-- Same DDL, different catalog
```

## Execution Order Safety

Three strategies combined:
1. **Sequential execution:** V001, V002, V003... dependent tables come later
2. **Idempotent DDL:** `IF NOT EXISTS` makes re-runs safe
3. **Separate DDL from DML:** Run ALL migrations first, then data processing. All tables exist before any MERGE references them.

---

**Next:** [[14 - Cost Optimization]] →
