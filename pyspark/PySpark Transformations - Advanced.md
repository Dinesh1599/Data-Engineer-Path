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

## when() / otherwise()

**Purpose:** SQL CASE WHEN equivalent - conditional logic for creating/transforming columns

**Syntax:**
```python
when(condition, value).otherwise(default_value)

# Multiple conditions (chained)
when(condition1, value1) \
  .when(condition2, value2) \
  .otherwise(default_value)
```

**Examples:**

### Simple If-Else
```python
# Create flag column
df.withColumn("category",
    when(col("age") < 18, "Minor")
    .otherwise("Adult")
)
```

### Multiple Conditions (If-ElseIf-Else)
```python
df.withColumn("grade",
    when(col("score") >= 90, "A")
    .when(col("score") >= 80, "B")
    .when(col("score") >= 70, "C")
    .otherwise("F")
)
```

### Complex Conditions (AND/OR)
```python
# Using & (AND) and | (OR)
df.withColumn("status",
    when((col("age") >= 18) & (col("country") == "US"), "Eligible")
    .when((col("age") >= 16) & (col("country") == "UK"), "Eligible")
    .otherwise("Not Eligible")
)
```

### Without otherwise()
```python
# Returns null if no condition matches
df.withColumn("flag",
    when(col("price") > 100, "Expensive")
    # No otherwise - returns null for price <= 100
)
```

**SQL Equivalent:**
```sql
CASE 
  WHEN age < 18 THEN 'Minor'
  ELSE 'Adult'
END

-- Multiple conditions
CASE
  WHEN score >= 90 THEN 'A'
  WHEN score >= 80 THEN 'B'
  WHEN score >= 70 THEN 'C'
  ELSE 'F'
END
```

**Key Points:**
- Always import: `from pyspark.sql.functions import when`
- Conditions evaluated **top to bottom** (first match wins)
- Use parentheses `()` for complex conditions with `&` and `|`
- `&` = AND, `|` = OR, `~` = NOT
- `.otherwise()` is optional (defaults to null)

**Common Patterns:**
```python
# Categorize numeric values
when(col("sales") > 1000, "High")
.when(col("sales") > 500, "Medium")
.otherwise("Low")

# Flag based on multiple columns
when((col("type") == "Veg") & (col("price") > 100), "Expensive Veg")
.otherwise("Other")

# Nested conditions
when(col("status") == "active",
    when(col("premium") == True, "Premium Active")
    .otherwise("Regular Active")
)
.otherwise("Inactive")
```

## Joins 

**Purpose:** Combine two DataFrames based on a condition

**Syntax:**
```python
df1.join(df2, condition, how="join_type")
df1.join(df2, df1["col"] == df2["col"], how="join_type")
```

---

### Join Types

### 1. Inner Join (default)
**Returns:** Only matching rows from both DataFrames
```python
df1.join(df2, df1["id"] == df2["id"], how="inner")
# OR
df1.join(df2, "id")  # If column name is same
```

**SQL:** `INNER JOIN`

---

### 2. Left Join (Left Outer)
**Returns:** All rows from left + matching from right (nulls for non-matches)
```python
df1.join(df2, df1["id"] == df2["id"], how="left")
# OR
df1.join(df2, "id", how="left")
```

**SQL:** `LEFT JOIN` or `LEFT OUTER JOIN`

**Use when:** Don't want to lose left table data

---

### 3. Right Join (Right Outer)
**Returns:** All rows from right + matching from left (nulls for non-matches)
```python
df1.join(df2, df1["id"] == df2["id"], how="right")
```

**SQL:** `RIGHT JOIN` or `RIGHT OUTER JOIN`

**Use when:** Don't want to lose right table data

---

### 4. Full Outer Join
**Returns:** All rows from both (nulls where no match)
```python
df1.join(df2, df1["id"] == df2["id"], how="outer")
# OR
df1.join(df2, "id", how="full")
```

**SQL:** `FULL OUTER JOIN`

**Use when:** Need complete data from both sides

---

### 5. Left Semi Join
**Returns:** Rows from left that have match in right (like IN/EXISTS)
```python
df1.join(df2, df1["id"] == df2["id"], how="left_semi")
```

**SQL Equivalent:**
```sql
SELECT * FROM df1
WHERE id IN (SELECT id FROM df2);
```

**Use when:** Filter left table based on right table existence

---

### 6. Left Anti Join ⭐
**Returns:** Rows from left that DON'T have match in right (like NOT IN)
```python
df1.join(df2, df1["id"] == df2["id"], how="left_anti")
```

**SQL Equivalent:**
```sql
SELECT * FROM df1
WHERE id NOT IN (SELECT id FROM df2);
```

**Use when:** Find unmatched records (e.g., users who haven't ordered)

---

### 7. Cross Join (Cartesian Product)
**Returns:** Every row from left × every row from right
```python
df1.crossJoin(df2)
# OR
df1.join(df2, how="cross")
```

**SQL:** `CROSS JOIN`

**Warning:** ⚠️ Expensive! Use with small DataFrames only

---

### Join Condition Patterns

### Same column name
```python
df1.join(df2, "id")  # Simplest
```

### Different column names
```python
df1.join(df2, df1["user_id"] == df2["customer_id"])
```

### Multiple columns
```python
df1.join(df2, 
    (df1["id"] == df2["id"]) & (df1["date"] == df2["date"])
)
```

### List of columns (same names)
```python
df1.join(df2, ["id", "date"])
```

---

### Handling Ambiguous Columns

**Problem:** Both DataFrames have same column names
```python
# Both have "id" column
df1.join(df2, "id")  # ❌ Ambiguous column error!
```

**Solutions:**

### 1. Specify DataFrame
```python
result = df1.join(df2, df1["id"] == df2["id"])
result.select(df1["id"], df1["name"], df2["sales"])
```

### 2. Rename before join
```python
df2 = df2.withColumnRenamed("id", "customer_id")
df1.join(df2, df1["id"] == df2["customer_id"])
```

### 3. Drop duplicate column after join
```python
result = df1.join(df2, "id").drop(df2["id"])
```

### 4. Use Alias
```python
result = df1.alias('a').join(df2.alias('b'),df1["id"]==df2["id"],"inner")\
	.select(col("a.id"))
```


---

### Performance Tips

### ✅ Best Practices:
```python
# 1. Broadcast small tables (< 10MB)
from pyspark.sql.functions import broadcast
large_df.join(broadcast(small_df), "id")

# 2. Filter before join
df1.filter(col("status") == "active").join(df2, "id")

# 3. Select only needed columns before join
df1.select("id", "name").join(df2.select("id", "sales"), "id")
```

### ⚠️ Avoid:
- Cross joins on large DataFrames
- Joins without filters on huge tables
- Multiple sequential joins without caching

---

### Quick Reference Table

| Join Type | Returns | Use Case |
|-----------|---------|----------|
| `inner` | Matches only | Standard join |
| `left` | All left + matches | Keep all left records |
| `right` | All right + matches | Keep all right records |
| `outer`/`full` | All from both | Combine everything |
| `left_semi` | Left with matches | Filter left by right |
| `left_anti` | Left without matches | Find missing records |
| `cross` | Cartesian product | Combinations |

## Window Functions in PySpark

**Purpose:** Perform calculations across a set of rows related to the current row

**Setup:**
```python
from pyspark.sql.window import Window
from pyspark.sql.functions import *
```

---

### Core Window Functions

#### 1. row_number()
**Returns:** Unique sequential number (1, 2, 3...) for each row in partition
```python
window = Window.partitionBy("category").orderBy("sales")
df.withColumn("row_num", row_number().over(window))
```

**Use case:** Remove duplicates, assign unique IDs

---

#### 2. rank()
**Returns:** Rank with gaps (1, 1, 3, 4...)
```python
window = Window.orderBy(col("score").desc())
df.withColumn("rank", rank().over(window))
```

**Behavior:** Same values = same rank, next rank skips
- Scores: [100, 95, 95, 90] → Ranks: [1, 2, 2, 4]

---

#### 3. dense_rank()
**Returns:** Rank without gaps (1, 1, 2, 3...)
```python
window = Window.orderBy(col("score").desc())
df.withColumn("dense_rank", dense_rank().over(window))
```

**Behavior:** Same values = same rank, next rank continues
- Scores: [100, 95, 95, 90] → Ranks: [1, 2, 2, 3]

---

#### 4. ntile(n)
**Returns:** Divides rows into n buckets (1 to n)
```python
window = Window.orderBy("salary")
df.withColumn("quartile", ntile(4).over(window))
```

**Use case:** Split data into equal groups (quartiles, percentiles)

---

### Aggregate Functions as Window Functions

#### 5. sum() / avg() / min() / max() / count()

**Running Total (Cumulative Sum):**
```python
window = Window.partitionBy("user_id") \
              .orderBy("date") \
              .rowsBetween(Window.unboundedPreceding, Window.currentRow)

df.withColumn("cumulative_sum", sum("amount").over(window))
```

**Moving Average (Last 7 days):**
```python
window = Window.partitionBy("product") \
              .orderBy("date") \
              .rowsBetween(-6, 0)  # Last 7 rows including current

df.withColumn("moving_avg", avg("sales").over(window))
```

---

### Offset Functions

#### 6. lag()
**Returns:** Value from previous row
```python
window = Window.partitionBy("user_id").orderBy("date")
df.withColumn("prev_value", lag("amount", 1).over(window))
# offset=1 means 1 row back, default is 1
```

**Use case:** Compare with previous value, calculate changes

---

#### 7. lead()
**Returns:** Value from next row
```python
window = Window.partitionBy("user_id").orderBy("date")
df.withColumn("next_value", lead("amount", 1).over(window))
```

**Use case:** Look ahead in time series

---

#### 8. first() / last()
**Returns:** First/last value in window frame
```python
window = Window.partitionBy("category").orderBy("date")
df.withColumn("first_sale", first("sales").over(window))
df.withColumn("last_sale", last("sales").over(window))
```

---

### Window Specification Components

#### partitionBy()
**Purpose:** Divide data into groups (like GROUP BY)
```python
Window.partitionBy("department")
Window.partitionBy("dept", "region")  # Multiple columns
```

---

#### orderBy()
**Purpose:** Define order within partition
```python
Window.orderBy("date")
Window.orderBy(col("sales").desc())
Window.orderBy("date", col("sales").desc())  # Multiple columns
```

---

### Frame Specifications

#### rowsBetween()
**Purpose:** Define row range for calculation
```python
# All rows from start to current
Window.rowsBetween(Window.unboundedPreceding, Window.currentRow)

# Last 3 rows including current
Window.rowsBetween(-2, 0)

# Current row + next 2 rows
Window.rowsBetween(0, 2)

# All rows in partition
Window.rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing)
```

#### rangeBetween()
**Purpose:** Define value range (for numeric/date columns)
```python
# All rows within 7 days before current row
Window.orderBy("date").rangeBetween(-7, 0)
```

---

### Common Patterns

#### Remove Duplicates (Keep First)
```python
window = Window.partitionBy("user_id").orderBy("timestamp")
df.withColumn("rn", row_number().over(window)) \
  .filter(col("rn") == 1) \
  .drop("rn")
```

#### Top N per Group
```python
window = Window.partitionBy("category").orderBy(col("sales").desc())
df.withColumn("rank", row_number().over(window)) \
  .filter(col("rank") <= 3)  # Top 3 per category
```

#### Running Total
```python
window = Window.partitionBy("user_id") \
              .orderBy("date") \
              .rowsBetween(Window.unboundedPreceding, Window.currentRow)

df.withColumn("running_total", sum("amount").over(window))
```

#### Calculate Change from Previous
```python
window = Window.partitionBy("product").orderBy("date")
df.withColumn("prev_price", lag("price").over(window)) \
  .withColumn("price_change", col("price") - col("prev_price"))
```

#### Moving Average (7-day)
```python
window = Window.partitionBy("product") \
              .orderBy("date") \
              .rowsBetween(-6, 0)

df.withColumn("moving_avg_7d", avg("sales").over(window))
```

#### Percent of Total
```python
window = Window.partitionBy("category")
df.withColumn("category_total", sum("sales").over(window)) \
  .withColumn("percent_of_total", 
      (col("sales") / col("category_total")) * 100)
```

---

### SQL Equivalent
```sql
-- PySpark
window = Window.partitionBy("dept").orderBy("salary")
df.withColumn("rank", rank().over(window))

-- SQL
SELECT *, 
  RANK() OVER (PARTITION BY dept ORDER BY salary) as rank
FROM table;
```

---

### Performance Tips

#### ✅ Best Practices:
- Always use `orderBy()` with ranking functions
- Partition on columns with reasonable cardinality
- Cache DataFrame if using multiple window functions
- Use specific frame bounds when possible

#### ⚠️ Avoid:
- Window without `partitionBy()` on huge datasets (global sort)
- Multiple different windows - combine when possible
- Unbounded frames with large partitions

---

### Quick Reference

| Function       | Returns        | Common Use            |
| -------------- | -------------- | --------------------- |
| `row_number()` | 1,2,3,4        | Unique ID, dedupe     |
| `rank()`       | 1,2,2,4        | Rankings with gaps    |
| `dense_rank()` | 1,2,2,3        | Rankings without gaps |
| `ntile(n)`     | 1,2,3,4        | Buckets/quartiles     |
| `lag()`        | Previous value | Change calculation    |
| `lead()`       | Next value     | Look ahead            |
| `sum().over()` | Running total  | Cumulative sum        |
| `avg().over()` | Moving average | Smoothing             |
