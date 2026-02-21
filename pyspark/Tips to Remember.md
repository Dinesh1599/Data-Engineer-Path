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