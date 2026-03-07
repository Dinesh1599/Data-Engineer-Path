# 11 — SQL Database Agent

## What We're Building

An agent that takes natural language questions and automatically queries your SQL database to get answers.

```
User: "How much total sales we made for tablet?"
Agent: → lists tables → reads schema → writes SQL → runs query → "600"
```

## Step 1: Connect to the Database

```python
from langchain_community.utilities.sql_database import SQLDatabase

sql_db = SQLDatabase.from_uri("sqlite:///SalesDB/sales.db")
```

LangChain reads the database schema (tables, columns, types) so the LLM knows what's available.

## Step 2: Create the SQL Toolkit

```python
from langchain_community.agent_toolkits.sql.toolkit import SQLDatabaseToolkit

toolkit = SQLDatabaseToolkit(db=sql_db, llm=llm)
toolkit.get_tools()
```

This automatically creates 4 SQL-specific tools:

| Tool | What it does |
|---|---|
| `sql_db_list_tables` | Lists all tables in the database |
| `info_sql_database` | Gets schema (columns, types) for a table |
| `sql_db_query` | Runs a SQL query and returns results |
| `sql_db_query_checker` | Validates a SQL query before running it |

You don't write these tools yourself — LangChain provides them.

## Step 3: Create the Agent

```python
from langchain.agents import create_agent

agent = create_agent(llm, toolkit.get_tools())
```

No system prompt needed. No `hub.pull()` needed. The agent has sensible defaults.

## Step 4: Ask Questions

```python
# With streaming (see the steps)
events = agent.stream(
    {"messages": [("user", "How much total sales we made for tablet?")]},
    stream_mode="values"
)
for event in events:
    event["messages"][-1].pretty_print()
```

## What the Agent Does Behind the Scenes

1. **Lists tables** → finds `orders` table
2. **Gets schema** → learns columns: `id, customer_name, product_name, quantity, price, total`
3. **Writes SQL** → `SELECT SUM(total) FROM orders WHERE product_name = 'Tablet'`
4. **Runs query** → gets result: `600`
5. **Formats answer** → "Total sales for tablet: $600 (3 units × $200 each)"

All automatic. You just asked a question in English.

## Why Schema Descriptions Matter

The LLM picks columns based on their names. If your column is named `tot_amt` instead of `total`, the LLM might pick the wrong column. Adding descriptions to your database columns helps the LLM make better decisions. This is a data governance best practice for AI-powered databases.

## Setting Up a SQLite Database (For Practice)

```python
import sqlite3

con = sqlite3.connect("SalesDB/sales.db")
cursor = con.cursor()

cursor.execute("""
    CREATE TABLE orders (
        id INTEGER, customer_name TEXT, product_name TEXT,
        quantity INTEGER, price REAL, total REAL
    )
""")

cursor.execute("INSERT INTO orders VALUES (1, 'Alice', 'Laptop', 1, 1000, 1000)")
cursor.execute("INSERT INTO orders VALUES (2, 'Bob', 'Tablet', 3, 200, 600)")
cursor.execute("INSERT INTO orders VALUES (3, 'Charlie', 'Smartphone', 2, 500, 1000)")

con.commit()
con.close()
```

## Security Note

Be cautious in production. An LLM can generate any SQL — including `DROP TABLE`. Always scope database permissions as narrowly as possible.

---

Previous: [[10-ReAct-Agents]] | Next: [[12-Memory-and-Conversations]]
