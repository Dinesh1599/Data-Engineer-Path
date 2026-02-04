# Topics
[[OLTP vs OLAP]]
[[Data modeling Techniques]]
[[Questions to think about when creating a Data model]]

**Data modeling** is the process of organizing raw data into a structured format (blueprint) so it can be stored, understood, and queried efficiently for analytics and reporting.

**For Data Engineers:**
	its not just about pipelines and tools, its needed to design data structures that support reporting, scalability and performance!!

More Explaination:

**Data modeling** defines **how data is structured, related, and stored** in a database or data warehouse. It translates real-world business concepts (customers, orders, events) into tables, columns, and relationships that machines can process efficiently!

Instead of keeping data as scattered files or logs, data modeling gives it **shape, meaning, and rules**.

**Why Data Modeling?**

1. Without data modeling:
	- Data is hard to understand
	- Queries become slow and complex
	- Reports give inconsistent results
	- Scaling analytics becomes painful

2. With good data modeling:
		- Data is **consistent and trustworthy**
		- Queries are **simpler and faster**
		- Analytics and dashboards are **easy to build**
		- New data sources integrate cleanly
		- Scalable!

> **In short:** Data modeling turns data into a **reliable product**, not just stored information.

**Where does it fit in the life cycle?**
	1. Before Pipeline Build
		1. Purpose
			1. To define fact/dim table
			2. Table Structure Decision
			3. Prevent rework
		2. Used for:
			1. Data Warehouse design
			2. dbt model planning
			3. Analytics layer definition
	2. During Transformation
		1. Purpose
			1. To refine raw data to analyics-ready data
			2. Normalize and denormalize
			3. Business logic
		2. Used for:
			1. [[Data modeling Techniques#**Star Schema** | Star Schema]]
			2. Aggregated marts
			3. Feature Table
	3. Before BI / Reporting - Critical
		1. Purpose
			1. Ensure consistent metrics
			2. Enable simple queries
			3. Avoid duplicated logic in dashboards
		2. Used for
			1. Fact tables
			2. Dim tables
			3. Semantic layers


**Types of Data Modeling:**
	1. Conceptual Model (What)
		 High-level business view
		 No technical details
		 Used to align with stakeholders

> 	_Example:_ Customer → Order → Product

![[Screenshot 2026-01-17 at 4.17.32 AM.png]]


---

#### 2. Logical Model (How logically)

- Defines entities, attributes, and relationships
    
- Still database-agnostic
    

> _Example:_ Customer has many Orders; Order has order_date, total_amount
![[Screenshot 2026-01-17 at 4.18.16 AM.png]]
---

#### 3. Physical Model (How technically)

- Database-specific implementation
    
- Tables, columns, data types, indexes
    

> _Example:_ PostgreSQL tables with primary keys, foreign keys, and indexes

![[Screenshot 2026-01-17 at 4.18.29 AM.png]]




