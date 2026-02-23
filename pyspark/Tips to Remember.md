## col() vs Column Name

**Use `col()` → when computing on column values**  
(math, comparisons, conditions)  
Example:
```python
df.filter(col("age")>18)
```

**Use column name ("col") → when referring to the column itself**  
(select, drop, subset, groupBy, fillna, dropna)  
Example:
```python
df.fillna(
	"This is a null",
	subset = ["col1","col2"])
```

Since:
- col() - Returns a Spark Object
- Column Name - pointing to the column itself
> [!tip] `when/otherwise` is a Column Expression, Not a `withColumn()` Feature
> `when/otherwise` can be used **anywhere a column expression is valid** — not just inside `withColumn()`.
>
> ```python
> # withColumn()
> df.withColumn("flag", when(col("price") > 100, "High").otherwise("Low"))
>
> # select()
> df.select(when(col("price") > 100, "High").otherwise("Low").alias("flag"))
>
> # agg() — most powerful use case
> df.groupBy("category") \
>   .agg(sum(when(col("status") == "sold", col("price")).otherwise(0)).alias("revenue"))
> ```
>
> **Rule of thumb:** Wherever you can write `col("x")`, you can write `when(...).otherwise(...)` instead.