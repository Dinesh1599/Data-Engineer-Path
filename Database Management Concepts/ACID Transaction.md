 
ACID is a set of rules that make sure a database transaction is _safe, correct, and reliable_.  
Every time a bank transfers money, a shopping cart updates, or a record changes — ACID makes sure nothing breaks.

A - Atomicity
C - Consistency
I - Isolation
D - Durability

# **A → Atomicity**

**All or nothing.**  
A transaction either completes fully or not at all.  
No half-done updates.

**Example:**  
Move $100 from Account A → B
- If debit succeeds but credit fails → rollback → nothing changes.

# **C → Consistency**

**Data must always follow rules.**  
The database moves from one valid state to another valid state.

**Example:**  
Bank rule: Account balance cannot be negative.  
If a transaction violates this → database rejects it.

# **I → Isolation**

**Transactions don’t interfere with each other.**  
Even if 100 people update the system at the same time, each transaction behaves as if it's running alone.

**Example:**  
Two users buy the last item at the same time — database ensures only one succeeds.

# **D → Durability**

**Once a transaction is committed, it stays forever.**  
Even if the server crashes, power goes out, or the database restarts — the changes remain.

**Example:**  
You hit “Submit Order”.  
Power outage 1 second later — order is still saved.