# Data Modeling Questions Checklist


> A comprehensive guide of questions to ask yourself before and during data model creation

## TL;DR
- **You don't need to ask all questions**
	- Start with these 5:
		1. What's the grain?
		2. What's the purpose?
		3. Who uses it?
		4. How do I test it?
		5. Is it documented?
	- Then ask more based on:
		- Complexity
		- Criticality
		- No: of users
		- Data Volume
		- Experience level


---

## 📋 Quick Navigation

- [[Questions to think about when creating a Data model#🎯 Foundation Questions | Foundation Questions]]
- [[Questions to think about when creating a Data model#🔍 Granularity & Grain |Granularity & Grain]]
- [[Questions to think about when creating a Data model#💼 Business Requirements |Business Requirements]]
- [[Questions to think about when creating a Data model#⚙️ Technical Constraints |Technical Constraints]]
- [[Questions to think about when creating a Data model#🚀 Performance & Scalability|Performance & Scalability]]
- [[Questions to think about when creating a Data model#✅ Data Quality & Integrity|Data Quality & Integrity]]
- [[Questions to think about when creating a Data model#🔧 Maintenance & Operations|Maintenance & Operations]]
- [[Questions to think about when creating a Data model#User Experience|User Experience]]
- [[Questions to think about when creating a Data model#🔒 Security & Compliance|Security & Compliance]]
- [[Questions to think about when creating a Data model#📝 Documentation|Documentation]]

---

## 🎯 Foundation Questions

### What am I building?

- [ ] What is the **purpose** of this model?
- [ ] Is this a **fact table** or **dimension table**?
- [ ] Is this **transactional** or **analytical**?
- [ ] Is this **normalized** or **denormalized**?
- [ ] What **modeling approach** am I using? (Star schema, snowflake, OBT, Data Vault, 3NF)

### Who is this for?

- [ ] Who are the **primary users**? (Data analysts, data scientists, BI tools, applications)
- [ ] What is their **SQL proficiency**? (Beginner, intermediate, advanced)
- [ ] What **tools** will they use? (Tableau, Power BI, Python, R, SQL clients)
- [ ] How **technical** are they?
- [ ] What's their **typical use case**?

### Why this structure?

- [ ] Why this design over alternatives?
- [ ] What **tradeoffs** am I making?
- [ ] What am I **optimizing for**? (Query speed, storage, flexibility, simplicity)
- [ ] What problems does this model **solve**?
- [ ] What problems does this model **create**?

---

## 🔍 Granularity & Grain

### What does one row represent?

- [ ] What is the **grain** of this table?
- [ ] Can I describe the grain in **one sentence**?
- [ ] Is the grain **consistent** throughout the table?
- [ ] Is the grain **documented clearly**?

### Is this the right level of detail?

- [ ] Is this the **finest grain** available from the source?
- [ ] Can users **aggregate up** from this grain to answer their questions?
- [ ] Would a **finer grain** be more useful?
- [ ] Would a **coarser grain** be sufficient?
- [ ] Am I storing **too much detail** (unnecessary granularity)?
- [ ] Am I storing **too little detail** (can't answer key questions)?

### Time dimension questions

- [ ] What **time granularity** do I need? (Second, minute, hour, day, week, month, year)
- [ ] Do I need **point-in-time** data or snapshots?
- [ ] Do I need to track **when** data was created vs. **when** it was loaded?
- [ ] Should I include **timestamp** or just **date**?

### Entity relationships

- [ ] Is this **one-to-one**, **one-to-many**, or **many-to-many**?
- [ ] Could one row explode into **multiple rows** when joined? (Fan-out problem)
- [ ] What happens when I **aggregate** this data?
- [ ] Are there **bridge tables** needed for many-to-many relationships?

---

## 💼 Business Requirements

### What questions must this model answer?

- [ ] What are the **top 5 business questions** this model must support?
- [ ] What **metrics** need to be calculated?
- [ ] What **dimensions** are users slicing by?
- [ ] What **filters** are commonly applied?
- [ ] What **drill-down paths** do users need?

### What's the business context?

- [ ] What **business process** does this model represent?
- [ ] What **department** or **team** owns this data?
- [ ] How does this fit into the **broader business**?
- [ ] What **decisions** will be made from this data?
- [ ] What's the **impact** if this data is wrong?

### Current vs. historical needs

- [ ] Do users need **current state** only?
- [ ] Do users need **historical trends**?
- [ ] How far back should **history** go?
- [ ] Do I need to track **changes over time**? ([[SCD Type 2]])
- [ ] What happens when dimension attributes **change**?

---

## ⚙️ Technical Constraints

### Source data

- [ ] Where is the **source data** coming from?
- [ ] What is the **source system grain**?
- [ ] How **reliable** is the source data?
- [ ] What's the **data quality** of the source?
- [ ] Are there **multiple sources** to reconcile?
- [ ] What happens if sources **conflict**?

### Platform & infrastructure

- [ ] What **data warehouse** am I using? (Snowflake, BigQuery, Redshift, Databricks)
- [ ] What are the **platform limitations**?
- [ ] What **compute resources** are available?
- [ ] What's my **storage budget**?
- [ ] What **network** constraints exist?

### Technology capabilities

- [ ] Does my platform support **partitioning**?
- [ ] Does my platform support **clustering**?
- [ ] Can I use **materialized views**?
- [ ] Are **incremental loads** supported?
- [ ] What **compression** options exist?

---

## 🚀 Performance & Scalability

### Current scale

- [ ] How many **rows** will this table have? (Thousands, millions, billions)
- [ ] How many **columns** will this table have?
- [ ] What's the **average row size**?
- [ ] What's the **total table size**? (MB, GB, TB)
- [ ] How **wide** is this table? (Too wide = performance issues)

### Growth projections

- [ ] How fast will this table **grow**?
- [ ] What will the size be in **6 months**? **1 year**? **3 years**?
- [ ] Will the **grain** change over time?
- [ ] Will **new columns** be added frequently?
- [ ] Is there a **data retention** policy?

### Query patterns

- [ ] What are the **most common queries**?
- [ ] What columns are **filtered most often**?
- [ ] What columns are **joined most often**?
- [ ] What's the **query response time** requirement?
- [ ] Are queries **read-heavy** or **write-heavy**?

### Optimization opportunities

- [ ] Should I **partition** this table? By what column?
- [ ] Should I **cluster/sort** this table? By what columns?
- [ ] Should I create **indexes**? On what columns?
- [ ] Should I **denormalize** any dimensions?
- [ ] Should I create **aggregate tables** or summary tables?
- [ ] Should I use **compression**?

---

## ✅ Data Quality & Integrity

### Uniqueness & keys

- [ ] What is the **primary key**?
- [ ] Is the primary key **truly unique**?
- [ ] Should I create a **surrogate key**?
- [ ] What **foreign keys** exist?
- [ ] How do I handle **duplicates**?

### Nullability

- [ ] Which columns **cannot be NULL**?
- [ ] Which columns **can be NULL**?
- [ ] What does NULL **mean** in each column? (Unknown? Not applicable? Not yet loaded?)
- [ ] How do I **handle missing data**?

### Data integrity

- [ ] What **referential integrity** constraints exist?
- [ ] What **business rules** must be enforced?
- [ ] What **data validation** is needed?
- [ ] How do I handle **orphaned records**?
- [ ] What happens if a dimension **doesn't exist**?

### Testing strategy

- [ ] How do I **test** this model?
- [ ] What **row count** should I expect?
- [ ] What **data tests** should I write?
    - [ ] Unique key test
    - [ ] Not null test
    - [ ] Referential integrity test
    - [ ] Accepted values test
    - [ ] Row count reconciliation
    - [ ] Grain validation
- [ ] How do I detect **data drift**?
- [ ] How do I handle **test failures**?

---

## 🔧 Maintenance & Operations

### Refresh strategy

- [ ] How **often** should this model refresh? (Real-time, hourly, daily, weekly)
- [ ] Is **incremental** loading possible?
- [ ] Or does it need **full refresh**?
- [ ] What's the **refresh window**? (How long can it take?)
- [ ] What happens if a refresh **fails**?
- [ ] How do I handle **late-arriving data**?

### Dependencies

- [ ] What **upstream dependencies** does this model have?
- [ ] What **downstream models** depend on this?
- [ ] What's the **dependency chain**?
- [ ] Can this run in **parallel** with other models?
- [ ] What's the **critical path** in the pipeline?

### Change management

- [ ] How do I handle **schema changes**?
- [ ] What if I need to **add a column**?
- [ ] What if I need to **remove a column**?
- [ ] What if I need to **change the grain**?
- [ ] How do I ensure **backward compatibility**?
- [ ] What's my **rollback strategy**?

### Monitoring & alerts

- [ ] What **metrics** should I monitor?
- [ ] What **alerts** should I set up?
    - [ ] Data freshness
    - [ ] Row count anomalies
    - [ ] Failed refreshes
    - [ ] Query performance degradation
    - [ ] Unexpected nulls
- [ ] Who gets **notified** when something breaks?
- [ ] What's the **SLA** for this model?

---

## 👥 User Experience

### Discoverability

- [ ] How will users **find** this model?
- [ ] Is it in the **data catalog**?
- [ ] Is the **naming** intuitive?
- [ ] Are there **related models** users should know about?

### Usability

- [ ] Are column names **clear and consistent**?
- [ ] Are **abbreviations** documented?
- [ ] Are **units** specified? (dollars vs. cents, meters vs. feet)
- [ ] Are **date formats** consistent?
- [ ] Is the model **easy to join** with other tables?
- [ ] Do I need to create **helper views** for common queries?

### Self-service enablement

- [ ] Can users answer questions **without** help?
- [ ] Is there a **data dictionary**?
- [ ] Are there **example queries**?
- [ ] Are **common pitfalls** documented?
- [ ] Is there **training material** available?

---

## 🔒 Security & Compliance

### Access control

- [ ] Who should have **access** to this model?
- [ ] Should access be **role-based**?
- [ ] Should access be **row-level** filtered?
- [ ] Should access be **column-level** masked?
- [ ] How do I **audit** access?

### Sensitive data

- [ ] Does this model contain **PII** (Personally Identifiable Information)?
- [ ] Does this model contain **PHI** (Protected Health Information)?
- [ ] Does this model contain **financial data**?
- [ ] Should any columns be **encrypted**?
- [ ] Should any columns be **hashed** or **masked**?
- [ ] Do I need **data anonymization** or tokenization?

### Regulatory compliance

- [ ] What **regulations** apply? (GDPR, HIPAA, SOC2, CCPA)
- [ ] What's the **data retention** policy?
- [ ] How do I handle **right to be forgotten** requests?
- [ ] Are there **geographical restrictions**?
- [ ] Do I need an **audit trail**?

---

## 📝 Documentation

### Model documentation

- [ ] Have I documented the **grain**?
- [ ] Have I documented the **purpose**?
- [ ] Have I documented the **refresh schedule**?
- [ ] Have I documented the **data lineage**?
- [ ] Have I documented **known limitations**?

### Column documentation

- [ ] Is **every column** documented?
- [ ] Are **data types** clear?
- [ ] Are **calculation logic** explained?
- [ ] Are **edge cases** documented?
- [ ] Are **example values** provided?

### Usage documentation

- [ ] Have I provided **example queries**?
- [ ] Have I documented **common joins**?
- [ ] Have I documented **common filters**?
- [ ] Have I documented **aggregation gotchas**?
- [ ] Is there a **FAQ** section?

### Maintenance documentation

- [ ] Is the **refresh process** documented?
- [ ] Are **troubleshooting steps** documented?
- [ ] Is the **runbook** up to date?
- [ ] Are **contact owners** listed?
- [ ] Is **version history** tracked?

---

## 🎨 Design Patterns & Anti-Patterns

### Design pattern questions

- [ ] Am I following **established patterns**?
- [ ] Am I consistent with **other models** in the warehouse?
- [ ] Have I considered **reusable components**?
- [ ] Should this be a **fact** or **dimension**?
- [ ] Should I use [[Star Schema]] or [[Snowflake Schema]] or [[OBT]]?

### Anti-patterns to avoid

- [ ] Am I mixing **different grains** in one table? ❌
- [ ] Am I creating **God tables** (too many columns)? ❌
- [ ] Am I **over-normalizing** for analytics? ❌
- [ ] Am I creating **circular dependencies**? ❌
- [ ] Am I using **natural keys** instead of surrogate keys? ❌
- [ ] Am I **duplicating logic** across multiple models? ❌

---

## 🔄 Iteration & Improvement

### Feedback & iteration

- [ ] How will I collect **user feedback**?
- [ ] How often will I **review** this model?
- [ ] What **metrics** indicate success?
- [ ] What would make me **redesign** this model?
- [ ] Am I building for **today** or **tomorrow**?

### Version control

- [ ] Is the model definition in **version control**?
- [ ] Are changes **peer-reviewed**?
- [ ] Is there a **changelog**?
- [ ] Can I **rollback** if needed?

---

## 📊 Specific Model Types

### If building a FACT table:

- [ ] What **business process** does this represent?
- [ ] What are the **measures/metrics**?
- [ ] What are the **foreign keys** to dimensions?
- [ ] Is this **transactional** or **periodic snapshot**?
- [ ] Should I create **aggregate** fact tables?
- [ ] Are there **degenerate dimensions** (facts that act like dimensions)?

### If building a DIMENSION table:

- [ ] What **entity** does this represent?
- [ ] What are the **attributes**?
- [ ] Is this **slowly changing**? What [[SCD Type]]?
- [ ] Should I create a **hierarchy**?
- [ ] Should I create **role-playing dimensions**?
- [ ] Should I include **derived attributes**?

### If building a [[One Big Table (OBT)]]:

- [ ] Is this the **right use case** for denormalization?
- [ ] What's the **storage impact**?
- [ ] How will I handle **dimension updates**?
- [ ] What's the **aggregation strategy**?
- [ ] Have I documented the **grain clearly**?
- [ ] Have I trained users on **proper querying**?

### If building an AGGREGATE table:

- [ ] What **grain** am I aggregating to?
- [ ] What **metrics** am I pre-calculating?
- [ ] How much **faster** will this be than on-the-fly aggregation?
- [ ] How often should this **refresh**?
- [ ] Is this redundant with **platform capabilities** (materialized views)?

---

## ✨ Pre-Flight Checklist

> Use this before finalizing your model

### Design Complete ✓

- [ ] Grain is clearly defined and documented
- [ ] Business requirements are met
- [ ] Primary key is identified
- [ ] Foreign keys are identified
- [ ] All columns are documented
- [ ] Data types are appropriate
- [ ] Nullability is defined

### Performance Optimized ✓

- [ ] Partitioning strategy defined (if needed)
- [ ] Clustering/sorting strategy defined (if needed)
- [ ] Indexes identified (if applicable)
- [ ] Compression enabled
- [ ] Query performance tested

### Quality Assured ✓

- [ ] Data tests written
- [ ] Grain validation test exists
- [ ] Uniqueness tests exist
- [ ] Not-null tests exist
- [ ] Referential integrity tests exist
- [ ] Row count reconciliation test exists

### Operations Ready ✓

- [ ] Refresh schedule defined
- [ ] Dependencies mapped
- [ ] Failure handling defined
- [ ] Monitoring set up
- [ ] Alerts configured
- [ ] Runbook created

### User Ready ✓

- [ ] Model is documented
- [ ] Example queries provided
- [ ] Data catalog updated
- [ ] Training material created (if needed)
- [ ] Access controls implemented

### Compliance Met ✓

- [ ] Security requirements met
- [ ] PII/PHI handled appropriately
- [ ] Regulatory requirements met
- [ ] Data retention policy defined
- [ ] Audit trail enabled (if needed)

---

## 🎯 Decision Tree

```
START: Do I need to create a new model?
│
├─→ Can I use/extend an EXISTING model?
│   ├─→ YES: Extend existing model ✓
│   └─→ NO: Continue
│
├─→ What type of model?
│   ├─→ FACT: Answer fact table questions
│   ├─→ DIMENSION: Answer dimension questions
│   ├─→ AGGREGATE: Create summary/aggregate table
│   └─→ OBT: Evaluate if denormalization is appropriate
│
├─→ What's the GRAIN?
│   ├─→ Can describe in one sentence: ✓
│   └─→ Cannot describe clearly: ❌ STOP - rethink grain
│
├─→ What's the DATA VOLUME?
│   ├─→ <1M rows: Simple design OK
│   ├─→ 1M-100M rows: Need optimization
│   └─→ >100M rows: Critical optimization needed
│
├─→ How will it be REFRESHED?
│   ├─→ Real-time: Consider materialized view
│   ├─→ Incremental: Design for upserts
│   └─→ Full refresh: Simple but expensive
│
└─→ Is it DOCUMENTED?
    ├─→ YES: Ready to build ✓
    └─→ NO: ❌ Document first!
```

---

## 💡 Final Wisdom

> "Weeks of coding can save you hours of planning."

**Remember:**

- **Start with the business question**, not the technical solution
- **Document as you go**, not at the end
- **Test early**, test often
- **Keep it simple** until complexity is justified
- **Think about the end user** - will they understand this?
- **Plan for change** - requirements evolve
- **Measure twice, cut once** - design carefully, build confidently


