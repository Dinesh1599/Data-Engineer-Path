---
tags: [sql, window-functions, row-number, rank, lag, lead]
aliases: [Window Functions, ROW_NUMBER, LAG, LEAD]
---

# Window Functions

> [!info] Related notes
> [[07 - Query Optimization]] | [[02 - Delta Lake]]

Window functions perform calculations across rows **without collapsing them** (unlike GROUP BY).

```sql
FUNCTION() OVER (PARTITION BY column ORDER BY column)
```

- **PARTITION BY** = split into groups (like GROUP BY but keeps all rows)
- **ORDER BY** = sort within each group
- The function runs across each group independently

## The 5 patterns that cover 95% of interviews

### 1. Deduplication (ROW_NUMBER)

```sql
WITH ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY policy_id ORDER BY last_modified DESC
    ) AS rn
  FROM claims
)
SELECT * FROM ranked WHERE rn = 1;
```

> [!tip] This is THE dedup pattern
> Partition by business key, order by timestamp descending, keep `rn = 1`.

### 2. Nth Highest (DENSE_RANK)

```sql
WITH ranked AS (
  SELECT *,
    DENSE_RANK() OVER (
      PARTITION BY department ORDER BY salary DESC
    ) AS dr
  FROM employees
)
SELECT * FROM ranked WHERE dr = 4;
```

> [!warning] Use DENSE_RANK, not RANK
> RANK skips numbers after ties (1,1,3 — no rank 2). DENSE_RANK doesn't (1,1,2). For "Nth highest," RANK might not have a rank N.

### 3. Change from Previous (LAG)

```sql
SELECT claim_id, amount, claim_date,
  LAG(amount) OVER (PARTITION BY state ORDER BY claim_date) AS prev_amount,
  amount - LAG(amount) OVER (PARTITION BY state ORDER BY claim_date) AS change
FROM claims;
```

### 4. Running Total (SUM with ORDER BY)

```sql
SELECT claim_id, state, amount,
  SUM(amount) OVER (
    PARTITION BY state ORDER BY claim_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM claims;
```

### 5. Percentage of Total (SUM without ORDER BY)

```sql
SELECT claim_id, state, amount,
  ROUND(amount * 100.0 / SUM(amount) OVER (PARTITION BY state), 1) AS pct
FROM claims;
```

No ORDER BY on SUM → covers entire partition → same total on every row.

## Quick Reference

| Function | Output | Use for |
|----------|--------|---------|
| `ROW_NUMBER()` | 1, 2, 3, 4... | Deduplication |
| `RANK()` | 1, 1, 3, 4... | Ranking (skips after tie) |
| `DENSE_RANK()` | 1, 1, 2, 3... | Nth highest (no skip) |
| `LAG(col, n)` | Previous row's value | Change detection |
| `LEAD(col, n)` | Next row's value | Forward-looking |
| `SUM() OVER` | Running or partition total | Cumulative amounts |
| `NTILE(n)` | Split into n buckets | Percentiles |
| `FIRST_VALUE()` | First in window | Compare to best |
| `LAST_VALUE()` | Last in window | ⚠️ Needs frame clause |

## The LAST_VALUE Trap

> [!danger] LAST_VALUE is broken by default
> The default window frame ends at the **current row**. So `LAST_VALUE` returns the current row's own value (useless!).
>
> **Fix:** Add `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`
>
> **Better fix:** Use `FIRST_VALUE` with flipped `ORDER BY`. Same result, no trap.

```sql
-- ❌ BROKEN — returns current row's value
LAST_VALUE(amount) OVER (ORDER BY amount DESC)

-- ✅ FIXED — explicit full frame
LAST_VALUE(amount) OVER (ORDER BY amount DESC
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)

-- ✅ BETTER — avoid the trap entirely
FIRST_VALUE(amount) OVER (ORDER BY amount ASC)
```

---

**Next:** [[09 - Compute and Clusters]] →
