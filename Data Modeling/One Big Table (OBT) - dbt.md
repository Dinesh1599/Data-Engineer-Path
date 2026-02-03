

#data-modeling #dimensional-modeling #dbt #analytics

## Overview

**One Big Table (OBT)** is a data modeling pattern that **denormalizes** a fact table with all its related dimensions into a single, wide table. Instead of using JOINs to connect facts and dimensions, all data is pre-joined and stored together.

## Related Concepts

- [[Star Schema]]
- [[Dimensional Modeling]]
- [[Data Denormalization]]
- [[dbt Materialization Strategies]]
- [[SCD Type 2]]

---

## Traditional vs OBT

### Star Schema (Normalized)

```
fact_orders (small)
├─ order_id
├─ customer_id  ──→ dim_customers
├─ product_id   ──→ dim_products  
├─ date_id      ──→ dim_dates
└─ amount

Query: Requires JOINs
```

### One Big Table (Denormalized)

```
obt_orders (wide)
├─ order_id
├─ customer_id
├─ customer_name         ← Denormalized
├─ customer_email        ← Denormalized
├─ customer_city         ← Denormalized
├─ product_id
├─ product_name          ← Denormalized
├─ product_category      ← Denormalized
├─ order_date
├─ year, month, quarter  ← Denormalized
└─ amount

Query: No JOINs needed
```

---

## When to Use OBT ✅

### Best Use Cases

1. **BI Tool Performance**
    
    - BI tools (Tableau, Power BI, Looker) struggle with complex JOINs
    - Users need drag-and-drop simplicity
    - Query performance is critical
2. **Self-Service Analytics**
    
    - Non-technical users need data access
    - Don't want users to understand data modeling
    - Reduce SQL complexity
3. **Specific Analytical Domains**
    
    - Customer 360 views
    - Product analytics dashboards
    - Sales reporting
    - Marketing attribution
4. **ML Feature Stores**
    
    - Pre-aggregated features for models
    - All features in one place
    - Fast feature retrieval
5. **Small-Medium Datasets**
    
    - Less than 10-100M rows
    - Limited dimensions (5-10)
    - Storage cost manageable
6. **Embedded Analytics**
    
    - Customer-facing dashboards
    - API endpoints needing fast responses
    - Mobile app data queries

---

## When NOT to Use OBT ❌

### Anti-Patterns

1. **Large Datasets**
    
    - Billions of rows
    - Storage costs explode
    - Query performance degrades
2. **High-Cardinality Dimensions**
    - Millions of unique dimension values
    - Creates massive duplication
3. **Frequently Changing Dimensions**
    
    - Price updates
    - Address changes
    - Requires expensive full table updates
4. **Complex Many-to-Many Relationships**
    
    - Row explosion problem
    - Aggregation errors
5. **Multiple Grain Levels**
    
    - Mixing daily/hourly/monthly in one table
    - Causes confusion and errors
6. **High Update Frequency**
    
    - Real-time updates needed
    - Dimension changes are frequent

---

## Key Considerations 🎯

### 1. Data Duplication & Storage

**Problem**: Dimension values repeated for every fact row

```sql
-- Customer "John" has 100 orders
-- His details are stored 100 times!
order_id | customer_name | customer_email   | amount
---------|---------------|------------------|-------
1        | John Smith    | john@email.com   | 100
2        | John Smith    | john@email.com   | 150
...
100      | John Smith    | john@email.com   | 175
```

**Best Practice**:

- Only denormalize frequently-used attributes
- Monitor storage costs over time
- ✅ Good: customer_city, product_category (low cardinality)
- ❌ Bad: customer_address, product_description (high cardinality)

### 2. Update Anomalies

**Problem**: Dimension changes require updating ALL rows

```sql
-- Customer changes email
UPDATE obt_orders 
SET customer_email = 'new@email.com'
WHERE customer_id = 123;  -- Updates 1,000 rows!

-- vs normalized:
UPDATE dim_customers
SET email = 'new@email.com'
WHERE customer_id = 123;  -- Updates 1 row!
```

**Best Practice**:

- Rebuild OBT periodically instead of updating
- Use incremental materialization
- Schedule full refreshes (nightly/weekly)

### 3. Aggregation Errors (Fan-out)

**Problem**: One-to-many relationships cause row explosion

```sql
-- Order with 3 products creates 3 rows
order_id | order_total | product_name
---------|-------------|-------------
1        | 1000        | Product A
1        | 1000        | Product B
1        | 1000        | Product C

-- ❌ WRONG:
SELECT SUM(order_total) FROM obt_orders
-- Returns: 3000 (should be 1000!)

-- ✅ CORRECT:
SELECT SUM(DISTINCT order_total) FROM obt_orders
-- Or: COUNT(DISTINCT order_id)
```

**Best Practice**:

- Document grain clearly
- Educate users on correct aggregation
- Add data tests for grain validation

### 4. Query Performance Paradox

**Problem**: Wide tables scan unnecessary columns

```sql
-- OBT has 100 columns, user needs 3
SELECT customer_city, product_category, amount
FROM obt_sales  -- Still scans all 100 columns!
```

**Best Practice**:

- Always SELECT specific columns (never `SELECT *`)
- Use partitioning and clustering
- Consider column-level compression

### 5. Grain Confusion

**Problem**: Unclear what each row represents

**Best Practice**:

```yaml
# Document grain prominently
models:
  - name: obt_order_lines
    description: |
      **GRAIN**: One row per product per order
      
      Each order can have multiple rows.
      When aggregating:
      - Use COUNT(DISTINCT order_id) for order counts
      - Use SUM(line_total) for revenue
```

### 6. Historical Accuracy (SCD)

**Problem**: Point-in-time accuracy lost when dimensions update

```sql
-- Product category changed in 2024
-- But historical orders from 2020 now show new category!
order_id | product_category | order_date
---------|------------------|------------
1        | New Category     | 2020-01-15  ← Wrong!
```

**Best Practice**:

- Use dbt snapshots for slowly changing dimensions
- Join to dimension history at transaction time
- OR capture values at transaction time (never update)

### 7. Maintenance Complexity

**Challenges**:

- Multiple sources of truth (normalized + OBT)
- Sync issues
- Long rebuild times for large tables
- Schema changes require full rebuild

**Best Practice**:

- Use dbt incremental models
- Schedule regular full refreshes
- Keep normalized tables as source of truth
- Monitor rebuild duration

### 8. Cost Management

**Problem**: Small dimension changes trigger expensive rebuilds

**Best Practice**:

```sql
-- Identify change frequency BEFORE denormalizing
-- ✅ Denormalize: country codes, product categories
-- ❌ Don't denormalize: inventory levels, real-time pricing

-- Use views for high-change attributes
CREATE VIEW obt_with_live_data AS
SELECT 
    obt.*,
    inv.current_stock  -- Live join, not denormalized
FROM obt_sales obt
LEFT JOIN inventory inv ON obt.product_id = inv.product_id
```

---

## Implementation Pattern (dbt)

### Basic OBT Structure

```sql
-- models/marts/obt_sales.sql
{{
    config(
        materialized='incremental',
        unique_key='surrogate_key',
        partition_by={'field': 'order_date', 'data_type': 'date'},
        cluster_by=['customer_city', 'product_category']
    )
}}

WITH 
orders AS (
    SELECT * FROM {{ ref('fct_orders') }}
    {% if is_incremental() %}
    WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})
    {% endif %}
),

customers AS (
    SELECT 
        customer_id,
        customer_name,
        customer_city,
        customer_segment
    FROM {{ ref('dim_customers') }}
),

products AS (
    SELECT 
        product_id,
        product_name,
        product_category
    FROM {{ ref('dim_products') }}
)

SELECT 
    -- Surrogate key
    {{ dbt_utils.generate_surrogate_key([
        'o.order_id', 
        'o.product_id'
    ]) }} as surrogate_key,
    
    -- Fact columns
    o.order_id,
    o.order_date,
    o.amount,
    
    -- Customer dimension (selective)
    c.customer_id,
    c.customer_name,
    c.customer_city,
    c.customer_segment,
    
    -- Product dimension (selective)
    p.product_id,
    p.product_name,
    p.product_category,
    
    -- Pre-calculated fields
    DATE_TRUNC('month', o.order_date) as order_month,
    EXTRACT(year FROM o.order_date) as order_year
    
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
LEFT JOIN products p ON o.product_id = p.product_id
```

### YAML Configuration

```yaml
# models/marts/schema.yml
version: 2

models:
  - name: obt_sales
    description: |
      One Big Table for sales analytics dashboard
      
      **GRAIN**: One row per product per order (order line level)
      
      **USAGE NOTES**:
      - Each order can have multiple rows (one per product)
      - Use COUNT(DISTINCT order_id) for order counts
      - Use SUM(amount) for revenue (already at line level)
      
      **REFRESH SCHEDULE**:
      - Incremental: Hourly
      - Full refresh: Weekly (Sunday 2am)
    
    config:
      tags: ["obt", "hourly", "dashboard"]
    
    tests:
      # Test row count
      - dbt_utils.equal_rowcount:
          compare_model: ref('fct_order_lines')
      
      # Test for duplicates
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - order_id
            - product_id
    
    columns:
      - name: surrogate_key
        description: "Unique identifier for each row"
        tests:
          - unique
          - not_null
      
      - name: order_id
        description: "Order identifier"
        tests:
          - not_null
      
      - name: customer_name
        description: "Customer full name (denormalized from dim_customers)"
      
      - name: product_category
        description: "Product category (denormalized from dim_products)"
```

---

## Testing Strategy

### Critical Tests for OBTs

```yaml
# Grain validation
- dbt_utils.unique_combination_of_columns:
    combination_of_columns:
      - order_id
      - product_id

# Row count matches source
- dbt_utils.equal_rowcount:
    compare_model: ref('fct_orders')

# Data freshness
- dbt_utils.recency:
    datepart: day
    field: order_date
    interval: 1

# Denormalized values match source
- dbt_utils.relationships_where:
    to: ref('dim_customers')
    field: customer_name
    from_condition: "customer_id IS NOT NULL"
```

---

## Decision Checklist

Before creating an OBT, verify:

- [ ] **Use case is specific** (Dashboard X, ML model Y)
- [ ] **Grain is clear** and documented
- [ ] **Row count is manageable** (<10M safe, >100M careful)
- [ ] **Dimensions change infrequently** (low update frequency)
- [ ] **No many-to-many relationships** (or handled properly)
- [ ] **Rebuild frequency defined** (hourly/daily/weekly)
- [ ] **Storage cost acceptable** and monitored
- [ ] **Users trained** on grain and aggregation
- [ ] **Data quality tests** in place
- [ ] **Normalized tables maintained** as fallback

---

## Common OBT Examples

### Customer 360

```sql
obt_customer_360:
- customer_id
- customer_name
- customer_email
- customer_city
- total_orders
- total_revenue
- avg_order_value
- first_order_date
- last_order_date
- days_since_last_order
- lifetime_value
- favorite_category
- customer_segment
```

### Product Analytics

```sql
obt_product_analytics:
- product_id
- product_name
- product_category
- brand
- supplier_name
- total_units_sold
- total_revenue
- avg_price
- inventory_level
- days_since_last_sale
```

### Sales Dashboard

```sql
obt_sales_dashboard:
- order_id
- order_date
- customer_id, name, city, segment
- product_id, name, category
- amount
- order_month, order_year
```

---

## Performance Optimization

### Partitioning

```sql
{{
    config(
        partition_by={
            'field': 'order_date',
            'data_type': 'date',
            'granularity': 'day'
        }
    )
}}
```

### Clustering

```sql
{{
    config(
        cluster_by=['customer_city', 'product_category', 'order_date']
    )
}}
```

### Incremental Logic

```sql
{% if is_incremental() %}
    WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})
{% endif %}
```

---

## Hybrid Approach (Recommended)

Many modern teams use:

1. **Normalized star schema** (flexibility, source of truth)
2. **Focused OBTs** (specific dashboards, performance)
3. **Semantic layer** (dbt metrics, Looker) to abstract complexity

```
Data Warehouse Layer:
├─ fct_orders          (normalized)
├─ dim_customers       (normalized)
├─ dim_products        (normalized)

Marts Layer (OBTs):
├─ obt_sales_dashboard (denormalized, specific use case)
├─ obt_customer_360    (denormalized, specific use case)
└─ obt_product_analytics (denormalized, specific use case)
```

---

## Golden Rules 🏆

1. **Start narrow, expand carefully** - Don't denormalize everything
2. **Document the grain** - Make it crystal clear
3. **Test aggregations** - Watch for fan-out errors
4. **Monitor storage** - Track size growth over time
5. **Schedule rebuilds** - Incremental + periodic full refresh
6. **Keep normalized tables** - OBT is derivative, not source
7. **Train users** - Explain how to query correctly
8. **Version control** - Track all schema changes

---

## Key Takeaway

> **OBT is a performance optimization for specific use cases, not a replacement for good dimensional modeling.**

Use OBT when:

- Performance matters more than flexibility
- Users need simplicity over power
- Dataset size is manageable
- Use case is well-defined

Avoid OBT when:

- Data is huge and complex
- Flexibility is critical
- Updates are frequent
- Users are SQL-proficient
