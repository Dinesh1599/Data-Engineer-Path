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

### withColumnRenamed
- Renames a column at a DataFrame Level
- Different from `.alias()` which only renames in that specific transformation 
- Returns a new DataFrame (original unchanged)
**Syntax:** ```python df.withColumnRenamed("old_name", "new_name") ```

### withColumn()
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



