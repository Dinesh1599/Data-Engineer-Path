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

## Date Functions

**Purpose:** Work with date and timestamp columns

**Common Functions:**

### Current Date/Time
```python
from pyspark.sql.functions import *

current_date()           # Today's date
current_timestamp()      # Current timestamp
```

### Date Arithmetic
```python
date_add(col("date"), 7)        # Add 7 days
date_sub(col("date"), 7)        # Subtract 7 days
datediff(col("end"), col("start"))  # Days between dates
months_between(col("end"), col("start"))  # Months between
add_months(col("date"), 3)      # Add 3 months
```

### Date Extraction
```python
year(col("date"))        # Extract year
month(col("date"))       # Extract month
dayofmonth(col("date"))  # Extract day
dayofweek(col("date"))   # Day of week (1=Sunday)
hour(col("timestamp"))   # Extract hour
minute(col("timestamp")) # Extract minute
```

### Date Formatting
```python
date_format(col("date"), "yyyy-MM-dd")  # Format date as string
to_date(col("string"), "yyyy-MM-dd")    # String to date
to_timestamp(col("string"), "yyyy-MM-dd HH:mm:ss")  # String to timestamp
```

**Examples:**
```python
# Add week to current date
df.withColumn("next_week", date_add(current_date(), 7))

# Calculate age in days
df.withColumn("days_old", datediff(current_date(), col("birth_date")))

# Format date
df.withColumn("formatted", date_format(col("date"), "MM/dd/yyyy"))

# Extract year
df.withColumn("year", year(col("order_date")))
```

**SQL Equivalents:**
```sql
CURRENT_DATE
DATE_ADD(date, INTERVAL 7 DAY)
DATEDIFF(end_date, start_date)
DATE_FORMAT(date, '%Y-%m-%d')
YEAR(date)
MONTH(date)
```

**Key Points:**
- Date format patterns: `yyyy` (year), `MM` (month), `dd` (day), `HH` (hour)
- `datediff()` returns integer (number of days)
- Use `to_date()` to convert strings to proper date type
## dropna()

**Purpose:** Remove rows with null/missing values

**Syntax:**
```python
df.dropna()  # Drop rows with ANY null
df.dropna(how="all")  # Drop rows where ALL columns are null
df.dropna(subset=["col1", "col2"])  # Drop based on specific columns
```

**Examples:**
```python
# Drop rows with any null
df.dropna()

# Drop only if ALL columns are null
df.dropna(how="all")

# Drop if specific column has null
df.dropna(subset=["outlet_size"])

# Drop if any of multiple columns has null
df.dropna(subset=["col1", "col2"])
```

**SQL Equivalent:**
```sql
-- Drop rows with null in specific column
SELECT * FROM table
WHERE outlet_size IS NOT NULL;
```

**Key Points:**
- `how="any"` (default) - drop if ANY column is null
- `how="all"` - drop only if ALL columns are null
- `subset` - check only specified columns
- `thresh` – minimum number of non-null values required to keep a row
- `thresh` with `subset` – counts non-null values only in the specified columns
- Returns new DataFrame

## fillna()

**Purpose:** Replace null values with specified values

**Syntax:**
```python
df.fillna(value)  # Fill all nulls
df.fillna(value, subset=["col1"])  # Fill specific columns
df.fillna({"col1": val1, "col2": val2})  # Different values per column
```

**Examples:**
```python
# Fill all nulls with "Unknown"
df.fillna("Unknown")

# Fill specific column
df.fillna("Not Available", subset=["outlet_size"])

# Fill different values for different columns
df.fillna({
    "outlet_size": "Unknown",
    "item_weight": 0,
    "sales": 0.0
})
```

**SQL Equivalent:**
```sql
-- COALESCE or IFNULL
SELECT COALESCE(outlet_size, 'Unknown') AS outlet_size
FROM table;
```

**Key Points:**
- Without `subset` - fills ALL columns
- With `subset` - fills only specified columns
- Can use dictionary for column-specific values
- Type must match column type

**Common Pattern:**
```python
# Drop critical nulls, fill others
df = df.dropna(subset=["user_id"])  # Must have user_id
df = df.fillna({"age": 0, "city": "Unknown"})  # Fill optional fields
```

## split()

**Purpose:** Splits a string column into an array based on delimiter

**Syntax:**
```python
split(col("column"), "delimiter")
```

**Examples:**
```python
# Split by space
df.withColumn("words", split(col("sentence"), " "))

# Split CSV values
df.withColumn("items_array", split(col("items"), ","))

# Example data transformation:
# "apple,banana,orange" → ["apple", "banana", "orange"]
```

---

## Indexing (Array Access)

**Purpose:** Access specific element from array column

**Syntax:**
```python
col("array_column")[index]  # 0-based indexing
col("array_column").getItem(index)  # Alternative syntax
```

**Examples:**
```python
# Split and get first element
df.withColumn("items_array", split(col("outlet_type"), " "))
df.withColumn("first_word", col("items_array")[0])

# "Supermarket Type1" → ["Supermarket", "Type1"] → "Supermarket"

# Get second element
df.withColumn("second_word", col("items_array")[1])  # "Type1"
```

---

## explode()

**Purpose:** Converts array column into multiple rows (one row per array element)

**Syntax:**
```python
explode(col("array_column"))
```

**Examples:**
```python
# Before explode:
# [user_id, books]
# [1, ["book1", "book2", "book3"]]

df.withColumn("book", explode(col("books")))

# After explode:
# [user_id, books, book]
# [1, ["book1", "book2", "book3"], "book1"]
# [1, ["book1", "book2", "book3"], "book2"]
# [1, ["book1", "book2", "book3"], "book3"]
```

**Complete Example:**
```python
# Split, then explode
df = df.withColumn("items_array", split(col("items"), ","))
df = df.withColumn("item", explode(col("items_array")))

# "apple,banana,orange" → 3 separate rows with apple, banana, orange
```

**SQL Equivalent:**
```sql
-- Split (varies by SQL dialect)
SPLIT(text, ',')

-- Explode (not standard SQL, but exists in some dialects)
SELECT user_id, book
FROM table
LATERAL VIEW EXPLODE(books) AS book;
```

**Key Points:**
- `split()` creates array column
- Indexing `[0]`, `[1]` accesses array elements
- `explode()` creates new rows from array
- Use `explode()` to normalize nested data

**Related:** [[split]], [[array_contains]], [[collect_list]]