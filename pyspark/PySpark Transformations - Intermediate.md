## union()

**Purpose:** Combines two DataFrames by stacking rows (concatenates vertically)

**Syntax:**
```python
df1.union(df2)
```

**Examples:**
```python
# Simple union
df1.union(df2)

# Chain multiple unions
df1.union(df2).union(df3)
```

**SQL Equivalent:**
```sql
SELECT * FROM table1
UNION ALL
SELECT * FROM table2;
```

**Key Points:**
- Combines by **position**, NOT by column name
- Column order matters (col1 of df1 → col1 of df2)
- Does NOT remove duplicates (like SQL `UNION ALL`)
- Column count must match

## unionByName()

**Purpose:** Combines DataFrames by matching column **names** (not position)

**Syntax:**
```python
df1.unionByName(df2)
df1.unionByName(df2, allowMissingColumns=True)  # Handle schema differences
```

**Examples:**
```python
# Union by column names
df1.unionByName(df2)

# Allow missing columns (fills with null)
df1.unionByName(df2, allowMissingColumns=True)
```

**Comparison:**
```python
# df1: [id, name, age]
# df2: [name, id, age]  # Different order!

df1.union(df2)          # ❌ Misaligned! name→id, id→name
df1.unionByName(df2)    # ✅ Correctly aligned by name
```

**Key Points:**
- **Safer than union()** - maps by column name
- Column order doesn't matter
- Use `allowMissingColumns=True` for different schemas
- Preferred in production code

**Related:** [[union]], [[join]], [[concat]]

## String Functions

**Purpose:** Transform and manipulate string/text columns

**Common Functions:**

### Case Conversion
```python
from pyspark.sql.functions import *

upper(col("name"))        # UPPERCASE
lower(col("name"))        # lowercase
initcap(col("name"))      # Proper Case (Title Case)
```

### String Manipulation
```python
concat(col("first"), lit(" "), col("last"))  # Concatenate
substring(col("text"), 1, 5)                 # Extract substring (start, length)
length(col("name"))                          # String length
trim(col("text"))                            # Remove leading/trailing spaces
ltrim(col("text"))                           # Remove leading spaces
rtrim(col("text"))                           # Remove trailing spaces
```

### Pattern Matching
```python
regexp_replace(col("text"), "old", "new")    # Replace pattern
regexp_extract(col("text"), r"\d+", 0)       # Extract with regex
split(col("text"), ",")                      # Split into array
```

**Examples:**
```python
# Convert to uppercase
df.withColumn("upper_name", upper(col("name")))

# Replace text
df.withColumn("clean_text", regexp_replace(col("text"), "bad", "good"))

# Split column
df.withColumn("items_array", split(col("items"), ","))
```

**SQL Equivalents:**
```sql
UPPER(name)
LOWER(name)
INITCAP(name)
CONCAT(first, ' ', last)
SUBSTRING(text, 1, 5)
LENGTH(name)
TRIM(text)
REGEXP_REPLACE(text, 'old', 'new')
```

**Related:** [[col]], [[withColumn]], [[regexp_replace]], [[split]]