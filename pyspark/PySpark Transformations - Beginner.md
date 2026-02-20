Note: Ensure `from pyspark.sql.functions import *` is executed.
## 1. Select Transformation
- Does the same function as [[DQL | SELECT]] in SQL
- gets `col` from `pyspark.sql.functions`

Note: df.select and df.select(col()) both does the same thing but
-  Strings = "just column names" , cannot build expressions in it 
- `col()` = "column as a programmable object" - **Good practice**

`col()` is used to build expression on a column and not to s

Syntax:
1. Normal Select
```python
df.select('Item Identifier','Item_weight')
```

2. Using col() - Suggested
```python

df.select(col('Item Identifier'),col('Item_Weight'),col('Item_Size'))\
.display()
```

## 2. Alias
Similar to SELECT d_name as Name from xyz; (the AS command)

```python
df.select(col('Item_Identifier').alias('Item ID')).display()
```
## 3.  Filter
### Secnario 1 - WHERE 
``` python
df.filter(col('Item_Fat_Content')=='Regular').display()
```

-  This is similar to using [[DQL | WHERE]] in SQL
- df.filter() is used
### Scenario 2 - WHERE, AND

```python
df.filter((col('Item_Type') == 'Soft Drinks') & (col('Item_Weight' <10)).display()
```
- AND is replaced with &
- SQL EQUIVALENT
	```sql
	SELECT * from XYZ
	WHERE Item_Type = 'Soft Drinks'
	AND Item_Weight <10;
	```
### Scenario 3 - Where, AND, In

```python
df.filter((col('Outlet_Size').isNull()) & (col('Outlet_Location_Type').isin('Tier 1','Tier 2'))).display()
```

- Similar to the previous 2 scenarios
- IN is used as isin()
- SQL Equivalen
	```sql
	SELECT * FROM df
WHERE Outlet_Size IS NULL
  AND Outlet_Location_Type IN ('Tier 1', 'Tier 2');
	```

## 4. withColumnRenamed
- Renames a column at a DataFrame Level
- Different from `.alias()` which only renames in that specific transformation 
- Returns a new DataFrame (original unchanged)
**Syntax:** ```python df.withColumnRenamed("old_name", "new_name") ```

## 5. withColumn()
- If column name is **NEW** → creates new column 
- If column name **EXISTS** → modifies that column 
- Returns a new DataFrame (original unchanged)
```python 
# Add a constant value 
df = df.withColumn("flag", lit("new")) 

# Calculate from existing columns 
df = df.withColumn("total", col("price") * col("quantity")) 

# Create from function 
df = df.withColumn("upper_name", upper(col("name"))) 
```

## 6. cast()

**Purpose:** Converts a column from one data type to another (type casting)

**Syntax:**
```python
col("column_name").cast("new_type")
# OR
col("column_name").cast(IntegerType())
```

**Common Data Types:**
- `"string"` or `StringType()`
- `"integer"` or `IntegerType()`
- `"double"` or `DoubleType()`
- `"boolean"` or `BooleanType()`
- `"date"` or `DateType()`
- `"timestamp"` or `TimestampType()`

**Examples:**
```python
# Convert double to string
df = df.withColumn("item_weight", col("item_weight").cast("string"))

# Convert string to integer
df = df.withColumn("age", col("age").cast("integer"))

# Convert string to date
df = df.withColumn("date", col("date_str").cast("date"))
```

**SQL Equivalent:**
```sql
-- PySpark
col("price").cast("integer")

-- SQL
CAST(price AS INTEGER)
-- OR
price::INTEGER  -- PostgreSQL shorthand
```

**Common Use Cases:**
- Prepare columns for joins (ensure matching types)
- Fix schema issues after reading data
- Convert strings to numeric for calculations
- Prepare data for aggregations

**Important:**
- Invalid casts return `null` (e.g., "abc" → integer = null)
- Always validate data before casting
- Use before joins to avoid type mismatch errors

**Related:**
- [[PySpark Transformations - Beginner#withColumn() | withColumn()]] - often used together to modify columns

## 7. sort() or orderBy()

**Purpose:** Sorts/orders a DataFrame by one or more columns

**Syntax:**
```python
df.sort("column_name")  # ascending by default
df.sort(col("column_name").desc())  # descending
df.sort("col1", "col2")  # multiple columns
```

**Alias:** `orderBy()` - exact same function, interchangeable

**Examples:**
```python
# Sort ascending (default)
df.sort("item_weight")

# Sort descending
df.sort(col("item_weight").desc())
# OR
df.sort("item_weight", ascending=False)

# Multiple columns
df.sort("category", col("price").desc())

# Using asc/desc explicitly
df.sort(col("item_weight").asc())

# Multiple columns with mixed order
df.sort(
    col("item_type").desc(),
    col("item_visibility").asc()
)
```

**SQL Equivalent:**
```sql
-- PySpark
df.sort("price", ascending=False)

-- SQL
SELECT * FROM table
ORDER BY price DESC;
```

**Sort vs OrderBy:**
- `sort()` and `orderBy()` are **identical**
- Use whichever feels natural (SQL users prefer `orderBy`)

**Important Notes:**
- Sorting is **expensive** on large datasets (shuffles data)
- Returns a new DataFrame
- Default is ascending order
- Use `.asc()` or `.desc()` for explicit control

**Common Use Cases:**
- Find top/bottom N records
- Prepare data for display
- Rank analysis
- Time-series ordering
## 8. limit()

**Purpose:** Returns first N rows from DataFrame

**Syntax:**
```python
df.limit(n)
```

**Examples:**
```python
df.limit(10)  # First 10 rows
df.sort(col("sales").desc()).limit(5)  # Top 5 by sales
```

**SQL Equivalent:**
```sql
SELECT * FROM table LIMIT 10;
```

**Key Points:**
- No guaranteed order without `.sort()`
- Very fast operation
- Like Pandas `.head()`

**Related:** [[PySpark Transformations - Beginner#7. sort() or orderBy() | sort]]
## drop()

**Purpose:** Removes one or more columns from DataFrame

**Syntax:**
```python
df.drop("column_name")
df.drop("col1", "col2", "col3")  # Multiple columns
```

**Examples:**
```python
# Drop single column
df.drop("item_visibility")

# Drop multiple columns
df.drop("item_visibility", "item_type")
```

**SQL Equivalent:**
```sql
-- No direct DROP in SELECT, use exclusion
SELECT col1, col2  -- (exclude unwanted columns)
FROM table;
```

**Key Points:**
- Returns new DataFrame (original unchanged)
- Can drop multiple columns in one call
- Opposite of `.select()`
