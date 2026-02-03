# One Big Table (OBT)

#data-modeling #dimensional-modeling #analytics #denormalization

## Overview

**One Big Table (OBT)** is a data modeling pattern that **denormalizes** a fact table with all its related dimensions into a single, wide table. Instead of using JOINs to connect facts and dimensions, all data is pre-joined and stored together.

## Related Concepts

- [[Star Schema]]
- [[Dimensional Modeling]]
- [[Data Denormalization]]
- [[Materialized Views]]
- [[SCD Type 2]]
- [[Data Warehouse Performance]]

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
- Use incremental refresh patterns (delete + insert)
- Schedule full refreshes (nightly/weekly)
- Consider the update frequency before denormalizing

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

```sql
-- Document grain in table comments
COMMENT ON TABLE obt_order_lines IS 
'GRAIN: One row per product per order

Each order can have multiple rows.
When aggregating:
- Use COUNT(DISTINCT order_id) for order counts
- Use SUM(line_total) for revenue

Refresh Schedule:
- Incremental: Hourly
- Full refresh: Weekly (Sunday 2am)

Created: 2024-01-15
Owner: Analytics Team';
```

Or maintain documentation separately:

- Wiki/Confluence page
- Data catalog (Alation, Collibra, etc.)
- README file in version control

### 6. Historical Accuracy (SCD Issues)

**Problem**: Point-in-time accuracy lost when dimensions update

```sql
-- Product category changed from "Electronics" to "Smart Devices" in 2024
-- Historical orders from 2020 now show new category!

Order from 2020:
order_id | product_category | order_date
---------|------------------|------------
1        | Smart Devices    | 2020-01-15  ← Should be "Electronics"
```

**Best Practice**:

**Option 1: SCD Type 2 Snapshots**

```sql
-- Join to historical dimension values
SELECT 
    o.order_id,
    o.order_date,
    p_hist.product_category,  -- Category as it was at order time
    o.amount
FROM fct_orders o
LEFT JOIN dim_products_history p_hist
    ON o.product_id = p_hist.product_id
    AND o.order_date BETWEEN p_hist.valid_from 
                         AND COALESCE(p_hist.valid_to, '9999-12-31');
```

**Option 2: Capture at Transaction Time**

```sql
-- Store dimension values when transaction occurs
-- Never update these values in the OBT
INSERT INTO obt_orders (
    order_id,
    product_category,  -- Captured at order time
    order_date
)
SELECT 
    o.order_id,
    p.product_category,  -- Current value at order time
    o.order_date
FROM new_orders o
JOIN dim_products p ON o.product_id = p.product_id;
```

### 7. Maintenance Complexity

**Challenges**:

- Multiple sources of truth (normalized + OBT)
- Sync issues between OBT and source tables
- Long rebuild times for large tables
- Schema changes require full rebuild
- Testing complexity

**Best Practice**:

- Use incremental refresh patterns (delete + insert)
- Schedule regular full refreshes (weekly/monthly)
- Keep normalized tables as the source of truth
- Monitor rebuild duration and costs
- Document refresh schedules clearly
- Automate with orchestration tools (Airflow, Prefect, etc.)
- Version control all OBT creation scripts

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

## Implementation Patterns

### SQL Implementation (Standard)

```sql
-- Create OBT as a materialized table
CREATE TABLE obt_sales AS
WITH 
orders AS (
    SELECT * FROM fct_orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '90 days'  -- Incremental window
),

customers AS (
    SELECT 
        customer_id,
        customer_name,
        customer_city,
        customer_segment
    FROM dim_customers
),

products AS (
    SELECT 
        product_id,
        product_name,
        product_category
    FROM dim_products
)

SELECT 
    -- Surrogate key
    MD5(CONCAT(o.order_id, '-', o.product_id)) as surrogate_key,
    
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
LEFT JOIN products p ON o.product_id = p.product_id;
```

### Incremental Refresh Pattern

```sql
-- Merge new/updated records
MERGE INTO obt_sales AS target
USING (
    SELECT 
        MD5(CONCAT(o.order_id, '-', o.product_id)) as surrogate_key,
        o.order_id,
        o.order_date,
        o.amount,
        c.customer_id,
        c.customer_name,
        c.customer_city,
        c.customer_segment,
        p.product_id,
        p.product_name,
        p.product_category,
        DATE_TRUNC('month', o.order_date) as order_month,
        EXTRACT(year FROM o.order_date) as order_year
    FROM fct_orders o
    LEFT JOIN dim_customers c ON o.customer_id = c.customer_id
    LEFT JOIN dim_products p ON o.product_id = p.product_id
    WHERE o.order_date >= CURRENT_DATE - INTERVAL '7 days'
) AS source
ON target.surrogate_key = source.surrogate_key
WHEN MATCHED THEN
    UPDATE SET
        target.amount = source.amount,
        target.customer_name = source.customer_name,
        target.customer_city = source.customer_city,
        -- ... other fields
WHEN NOT MATCHED THEN
    INSERT VALUES (
        source.surrogate_key,
        source.order_id,
        source.order_date,
        -- ... all fields
    );
```

### Platform-Specific Optimizations

#### BigQuery

```sql
-- Partitioned and clustered OBT
CREATE TABLE obt_sales
PARTITION BY DATE(order_date)
CLUSTER BY customer_city, product_category
AS
SELECT ...
```

#### Snowflake

```sql
-- Clustered OBT
CREATE TABLE obt_sales
CLUSTER BY (order_date, customer_city, product_category)
AS
SELECT ...
```

#### Redshift

```sql
-- Distribution and sort keys
CREATE TABLE obt_sales
DISTKEY(customer_id)
SORTKEY(order_date, customer_city)
AS
SELECT ...
```

#### Databricks

```sql
-- Delta table with partitioning
CREATE TABLE obt_sales
USING DELTA
PARTITIONED BY (order_year, order_month)
AS
SELECT ...
```

---

## Maintenance Strategies

### Full Refresh

```sql
-- Drop and recreate (simple but expensive)
DROP TABLE IF EXISTS obt_sales;
CREATE TABLE obt_sales AS
SELECT ... FROM fct_orders o
LEFT JOIN dim_customers c ON ...
LEFT JOIN dim_products p ON ...;
```

**Schedule**: Weekly or monthly for large tables

### Incremental Update

```sql
-- Delete and insert changed rows
DELETE FROM obt_sales
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days';

INSERT INTO obt_sales
SELECT ... FROM fct_orders o
LEFT JOIN dim_customers c ON ...
LEFT JOIN dim_products p ON ...
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days';
```

**Schedule**: Daily or hourly

### Materialized View (if supported)

```sql
-- Database automatically maintains it
CREATE MATERIALIZED VIEW obt_sales AS
SELECT ... FROM fct_orders o
LEFT JOIN dim_customers c ON ...
LEFT JOIN dim_products p ON ...;

-- Refresh periodically
REFRESH MATERIALIZED VIEW obt_sales;
```

**Schedule**: Based on data freshness requirements

---

## Data Quality & Testing

### Critical Validations for OBTs

#### 1. Grain Validation

```sql
-- Ensure no duplicate keys
SELECT 
    surrogate_key,
    COUNT(*) as count
FROM obt_sales
GROUP BY surrogate_key
HAVING COUNT(*) > 1;
-- Should return 0 rows

-- Or check combination of columns
SELECT 
    order_id,
    product_id,
    COUNT(*) as count
FROM obt_sales
GROUP BY order_id, product_id
HAVING COUNT(*) > 1;
-- Should return 0 rows
```

#### 2. Row Count Validation

```sql
-- OBT should match source fact table
SELECT 
    'Source' as source,
    COUNT(*) as row_count
FROM fct_order_lines
UNION ALL
SELECT 
    'OBT' as source,
    COUNT(*) as row_count
FROM obt_sales;
-- Counts should match
```

#### 3. Data Freshness Check

```sql
-- Check latest data available
SELECT 
    MAX(order_date) as latest_order,
    CURRENT_DATE - MAX(order_date) as days_old
FROM obt_sales;
-- Should be recent (e.g., < 1 day old)
```

#### 4. Denormalized Values Match Source

```sql
-- Verify customer names match dimension
SELECT 
    o.customer_id,
    o.customer_name as obt_name,
    c.customer_name as dim_name
FROM obt_sales o
JOIN dim_customers c ON o.customer_id = c.customer_id
WHERE o.customer_name != c.customer_name;
-- Should return 0 rows
```

#### 5. NULL Check on Key Fields

```sql
-- Critical fields should not be NULL
SELECT COUNT(*) as null_count
FROM obt_sales
WHERE order_id IS NULL
   OR customer_id IS NULL
   OR product_id IS NULL;
-- Should return 0
```

#### 6. Aggregation Test

```sql
-- Compare aggregated metrics
SELECT 
    'Source' as source,
    COUNT(DISTINCT order_id) as order_count,
    SUM(amount) as total_revenue
FROM fct_orders
UNION ALL
SELECT 
    'OBT' as source,
    COUNT(DISTINCT order_id) as order_count,
    SUM(amount) as total_revenue
FROM obt_sales;
-- Metrics should match
```

### Automated Testing Schedule

```
Daily:
- Row count validation
- Data freshness check
- NULL checks on key fields

Weekly:
- Grain validation (no duplicates)
- Denormalized values match source
- Aggregation reconciliation

Monthly:
- Full audit against normalized tables
- Performance benchmarking
- Storage cost review
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

## Performance Optimization Techniques

### 1. Partitioning

Divide table into smaller chunks based on a column (usually date)

**BigQuery:**

```sql
CREATE TABLE obt_sales
PARTITION BY DATE(order_date)
AS SELECT ...
```

**Snowflake:**

```sql
-- Automatic micro-partitioning, but can hint with clustering
CREATE TABLE obt_sales
CLUSTER BY (order_date)
AS SELECT ...
```

**Redshift:**

```sql
CREATE TABLE obt_sales
DISTKEY(customer_id)
SORTKEY(order_date)
AS SELECT ...
```

**Benefits:**

- Query only scans relevant partitions
- Faster queries on date ranges
- Lower costs (in BigQuery)

### 2. Clustering/Sorting

Order data physically by frequently filtered columns

**BigQuery:**

```sql
CREATE TABLE obt_sales
PARTITION BY DATE(order_date)
CLUSTER BY customer_city, product_category
AS SELECT ...
```

**Snowflake:**

```sql
CREATE TABLE obt_sales
CLUSTER BY (customer_city, product_category, order_date)
AS SELECT ...
```

**Benefits:**

- Faster WHERE clause queries
- Better compression
- Reduced scan time

### 3. Incremental Loading

Only process new/changed data instead of full refresh

```sql
-- Delete recent data
DELETE FROM obt_sales
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days';

-- Insert fresh data
INSERT INTO obt_sales
SELECT ... 
FROM fct_orders o
LEFT JOIN dim_customers c ON ...
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days';
```

**Benefits:**

- Faster refresh times
- Lower compute costs
- Less table locking

### 4. Column Selection

Only denormalize frequently-used columns

```sql
-- ✅ Good: Selective denormalization
SELECT 
    o.*,
    c.customer_name,    -- Frequently used
    c.customer_city,    -- Frequently used
    p.product_name,     -- Frequently used
    p.product_category  -- Frequently used
FROM orders o
LEFT JOIN customers c ON ...
LEFT JOIN products p ON ...

-- ❌ Bad: Denormalize everything
SELECT 
    o.*,
    c.*,  -- All 50 customer columns
    p.*   -- All 30 product columns
FROM orders o
LEFT JOIN customers c ON ...
LEFT JOIN products p ON ...
```

### 5. Compression

Enable compression to reduce storage and improve scan speed

**Snowflake:**

```sql
-- Automatic compression, no action needed
```

**Redshift:**

```sql
CREATE TABLE obt_sales (
    order_id INT ENCODE AZ64,
    customer_name VARCHAR(100) ENCODE LZO,
    order_date DATE ENCODE AZ64,
    amount DECIMAL(10,2) ENCODE AZ64
)
SORTKEY(order_date);
```

### 6. Materialized Views vs Tables

**Materialized View:**

- Database manages refresh automatically
- Can't always add indexes/partitions
- Simpler to maintain

**Table:**

- Full control over optimization
- More maintenance overhead
- Better for complex refresh logic

### 7. Query Optimization Tips

```sql
-- ✅ Always select specific columns
SELECT customer_city, product_category, SUM(amount)
FROM obt_sales
WHERE order_date >= '2024-01-01'
GROUP BY 1, 2;

-- ❌ Avoid SELECT *
SELECT * FROM obt_sales;  -- Scans all columns

-- ✅ Use partition filters
WHERE order_date >= '2024-01-01'  -- Leverages partition pruning

-- ✅ Use DISTINCT carefully with aggregations
SELECT 
    COUNT(DISTINCT order_id) as orders,  -- Correct
    SUM(DISTINCT amount) as total        -- May be wrong if duplicate amounts
FROM obt_sales;
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

---
