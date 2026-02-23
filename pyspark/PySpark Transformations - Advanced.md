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

