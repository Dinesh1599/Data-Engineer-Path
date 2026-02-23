## collect_list()

**Purpose:** Aggregates values into an array (like SQL's GROUP_CONCAT but returns array)

**Syntax:**
```python
collect_list(col("column"))
```

**Examples:**
```python
# Collect all books read by each user
df.groupBy("user_id").agg(
    collect_list("book_name").alias("books_read")
)

# Multiple aggregations
df.groupBy("category").agg(
    collect_list("product_name").alias("products"),
    sum("sales").alias("total_sales")
)
```

**Before vs After:**
```python
# Before:
[user_id, book]
[1, "book1"]
[1, "book2"]
[1, "book3"]
[2, "book4"]

# After collect_list:
[user_id, books_read]
[1, ["book1", "book2", "book3"]]
[2, ["book4"]]
```

**SQL Equivalent:**
```sql
-- MySQL
GROUP_CONCAT(book_name)

-- PostgreSQL
ARRAY_AGG(book_name)

-- Standard SQL (some dialects)
COLLECT_LIST(book_name)
```

**Key Points:**
- Must use with `groupBy()`
- Returns array column
- Keeps duplicate values
- Order not guaranteed (use `collect_set()` for distinct values)

**Related Function:**
```python
collect_set(col("column"))  # Same but returns distinct values only
```

**Common Use Case:**
```python
# Get all transactions per customer
orders.groupBy("customer_id").agg(
    collect_list("order_id").alias("all_orders"),
    count("*").alias("order_count")
)
```

## pivot()

**Purpose:** Rotates rows into columns (creates pivot table)

**Syntax:**
```python
df.groupBy("row_column").pivot("pivot_column").agg(aggregation)
```

**Examples:**
```python
# Basic pivot
df.groupBy("item_type") \
  .pivot("outlet_size") \
  .sum("sales")

# With multiple aggregations
df.groupBy("region") \
  .pivot("quarter") \
  .agg(
      sum("sales").alias("total_sales"),
      avg("price").alias("avg_price")
  )
```

**Before vs After:**
```python
# Before (long format):
[item_type, outlet_size, sales]
["Dairy", "Small", 100]
["Dairy", "Medium", 200]
["Fruits", "Small", 150]

# After pivot (wide format):
[item_type, Small, Medium]
["Dairy", 100, 200]
["Fruits", 150, null]
```

**SQL Equivalent:**
```sql
SELECT item_type,
  SUM(CASE WHEN outlet_size = 'Small' THEN sales END) AS Small,
  SUM(CASE WHEN outlet_size = 'Medium' THEN sales END) AS Medium
FROM table
GROUP BY item_type;
```

### When to Use Pivot - Tips

### ✅ Use Pivot When:
1. **Reporting/Dashboards** - Easier to read wide format
2. **Few unique values** in pivot column (< 100)
3. **Comparison across categories** - Sales by month, region by product
4. **Excel-like output** needed
5. **Matrix views** - Products vs Regions, Time vs Metrics

### ❌ Avoid Pivot When:
1. **Many unique values** in pivot column (> 1000) - creates too many columns
2. **Dynamic/unknown values** - Better to keep long format
3. **Further aggregations** needed - Harder with pivoted data
4. **Machine learning** - Models prefer long format
5. **Large datasets** - Pivot is expensive (use for final reporting only)

**Performance Tips:**
```python
# Specify pivot values explicitly (faster!)
df.groupBy("item_type") \
  .pivot("outlet_size", ["Small", "Medium", "Large"]) \
  .sum("sales")
```

**Common Use Cases:**
```python
# Sales by month (columns = months)
sales.groupBy("product").pivot("month").sum("revenue")

# Metrics dashboard (columns = metric types)
data.groupBy("date").pivot("metric_name").avg("value")

# A/B test results (columns = test groups)
experiments.groupBy("feature").pivot("test_group").avg("conversion_rate")
```



