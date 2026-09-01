# 06 — Databricks, Spark, Unity Catalog, Delta, Auto Loader, DLT

---

## 1. Why Databricks / PySpark?

- ADF Data Flows = limited UI transforms  
- **Databricks notebooks** = unlimited custom transform logic  

**Languages:** Python, SQL, Scala, R  

| Term | Meaning |
|------|---------|
| **Python** | General language + many libraries |
| **PySpark** | Python API for Apache Spark (DE / analytics / DS) |

---

## 2. Big Data 3 Vs

| V | Meaning |
|---|----------|
| Volume | Huge size (PB searches, YouTube video) |
| Variety | Structured / Semi (JSON,XML) / Unstructured (audio,video,image) |
| Velocity | Need high performance processing |

**Hadoop:** HDFS (storage) + MapReduce (processing) — writes intermediate to disk → slower.  
**Spark:** parallel tasks **in-memory (RAM)** → faster.  
**Databricks:** managed Spark platform (Microsoft partnership for Azure Databricks).

---

## 3. Spark / Databricks architecture

```
Driver Node
  → Cluster Manager
  → Worker Nodes
       → Executors
            → Tasks

DAG (Directed Acyclic Graph)
Job → Stages → Tasks
```

**Notebook flow:**
```
Read → RDD / DataFrame / Dataset → Transform1 → Transform2 → Action/Load
```
Transforms are **lazy**; Actions trigger DAG execution.

### Cluster & Notebook
| Object | Role |
|--------|------|
| Cluster | Compute (CPU/RAM) to run notebooks |
| Notebook | Commands (PySpark/SQL/Scala/R) |
| Attach | Notebook must attach to cluster |

**Cluster types:**
- **All-purpose / interactive** — for development  
- **Job clusters** — created for job run, then terminate (cost efficient)

---

## 4. Editions comparison (as taught)

| Edition | Cost | External ADLS/SQL | Notes |
|---------|------|-------------------|-------|
| Community (legacy) | Free | Possible | Custom cluster |
| Community (latest) | Free | External often **not** supported | DBFS + Unity Catalog style |
| Azure Databricks | Paid (free credit ~28 days) | Full Azure integration | Unity Catalog, Access Connector |

Community: https://community.cloud.databricks.com/

---

## 5. SQL & Python basics (covered before PySpark)

See also assignments in section 14.

Python structures: List, Tuple, Set, Dictionary + loops + functions.

---

## 6. RDD vs DataFrame vs Dataset

| | RDD | DataFrame |
|--|-----|-----------|
| Abstraction | Low-level | Higher-level table-like |
| Optimization | Manual | Catalyst + Tungsten |
| Speed | Generally slower for analytics | Faster |
| Schema | Unstructured/typed loosely | Schema + columns |

Prefer DataFrame for DE work unless you need low-level RDD control.

**Lazy evaluation:** transformations build plan; action (`show`, `count`, `write`, `collect`) executes.

**Narrow vs Wide transforms:**
- Narrow: map, filter (no shuffle)  
- Wide: groupBy, join, distinct (shuffle)

**Action examples:** count, collect, show, take, write  
**Transform examples:** select, filter, withColumn, join, groupBy  

**Warning:** `.collect()` pulls data to **Driver** — use carefully on large data.

---

## 7. Unity Catalog

```
Metastore
  └── Catalog (e.g. eclasess2026)
        └── Schema (sales, finance, marketing)
              ├── Tables (managed / external)
              └── Volumes (files)
```

**Benefits:**
- Central permissions (catalog → schema → table → column/row)  
- Auditing (who/what/when/which workspace)  
- Lineage (Notebook1 → Notebook2 → Notebook3)  
- Volumes for files  

### Managed vs External tables

| | Managed | External |
|--|---------|----------|
| Data location | Databricks-managed | Customer ADLS/S3/GCS |
| Metadata | Databricks | Databricks |
| DROP TABLE | Deletes data + metadata | Deletes **metadata only** |

Default table type under UC often **Delta**.

### Connect ADLS Gen2 → Databricks

| Method | When |
|--------|------|
| Access Key / SAS | Practice / simple |
| **Managed Identity + Access Connector** | Azure-internal recommended |
| **Service Principal** (client id, tenant, secret) | Cross-cloud / apps; mount patterns |

Steps (taught):
1. Create ADLS Gen2  
2. Create Access Connector for Databricks  
3. RBAC on SA: Storage Blob Data Contributor/Reader to connector  
4. Enable Unity Catalog / create metastore  
5. Create Catalog → Schema → Volume / Tables  
6. Create External Location (ADLS path + connector resource id)  
7. Read/transform/write Delta  

---

## 8. Reading / writing patterns

### Azure Databricks
```
ADLS / DBFS → Notebook → ADLS / Azure SQL / Synapse
```

### Fabric Notebooks
```
Lakehouse files/tables ↔ PySpark
Lakehouse tables → Warehouse via T-SQL notebook
```

**Important Fabric constraints (class):**
- PySpark notebook → Lakehouse files/tables: OK  
- PySpark notebook → Warehouse write: **Not OK** (as taught)  
- T-SQL notebook → Warehouse DML: OK  
- Pattern: PySpark Files→Tables, then T-SQL Tables→Warehouse  

```python
df = spark.read.format('csv') \
  .option('header', True) \
  .option('inferSchema', True) \
  .load('File Path')
```

Read only Transactions* files:
```python
df = spark.read.parquet("/mnt/ADLS/raw/Transactions*.parquet")
```

Mount check:
```python
dbutils.fs.mounts()
```

---

## 9. Delta Lake

| Feature | Meaning |
|---------|---------|
| Format | Open table format on Parquet + transaction log |
| ACID | Atomicity, Consistency, Isolation, Durability |
| Schema enforcement | Reject bad schema writes |
| Time travel | Query older versions by version/timestamp |
| Versioning | Each change creates a new version |

**Delta vs Parquet (interview):**
- Parquet = columnar file format  
- Delta = Parquet files + `_delta_log` for ACID, time travel, upserts/MERGE, schema evolution  

Prefer Delta for lakehouse tables; Parquet OK for simple interchange.

---

## 10. Auto Loader

Incremental file ingestion from cloud storage with checkpoints.

**Advantages:**
- Auto-detect new files  
- Process only new data  
- Scale to millions of files  
- Schema evolution handling  
- Checkpointing  

```
Daily files: jansales01.csv, jansales02.csv…
Auto Loader → Bronze → Silver → Gold
```

---

## 11. Delta Live Tables (DLT)

Framework to build automated pipelines on Delta Lake.

**Provides:**
- Dependency management  
- Data quality expectations  
- Monitoring  
- Incremental processing  
- Scheduling  

**Table types:**
- Streaming tables  
- Materialized views  

Flow:
```
Source files → Auto Loader → Bronze → Silver → Gold (materialized views) → Power BI
```

---

## 12. Workflows / Jobs

Schedule notebooks in order:
```
Notebook1 → Notebook2 → Notebook3 (+ email on success/fail)
```

**Conditional example:**
```
If weekday → Notebook2 (weekdayssales.csv → table)
Else       → Notebook3 (weekendssales.csv → table)
```

**Run many notebooks:**
- Databricks Workflow / Job with multiple tasks  
- `%run` / `dbutils.notebook.run()`  
- YAML job definition (CI)  

**Fabric scheduling:** Notebook scheduler or Data Pipelines.  
**Azure scheduling:** Workflows or ADF.

---

## 13. SCD / MERGE patterns (from notes)

SCD1: overwrite attributes.  
SCD2: close old row (`is_active=0`, end date) + insert new version.

```sql
MERGE INTO trg t
USING source s
ON s.id = t.id AND t.is_active = '1'
WHEN MATCHED AND (s.email <> t.email OR s.phone <> t.phone OR s.salary <> t.salary)
THEN UPDATE SET t.future_date = current_date(), t.is_active = '0';

-- Then insert new active versions for changed / new keys
MERGE INTO trg t
USING source s
ON s.id = t.id AND t.is_active = '1'
WHEN NOT MATCHED THEN INSERT (...);
```

Atomic MERGE supports SCD reliability in Delta.

---

## 14. Python assignments (from class)

### Assignment 1 — Basics
Add two numbers; square; even/odd; largest of two; C→F.

### Assignment 2 — Conditionals
Positive/negative/zero; leap year; grade; pass/fail; biggest of three.

### Assignment 3 — Loops
1–10; even 1–20; sum first 10; multiplication table; factorial.

### Assignment 4 — Structures
List of 5; max/min; remove duplicates; student dict; frequency count.

### Assignment 5 — Functions
Add; is_prime; factorial; palindrome; average of list.

---

## 15. Optimization topics (interview-ready)

| Technique | Idea |
|-----------|------|
| Partitioning / repartition / coalesce | Control file/partition count; coalesce to reduce without full shuffle |
| Cache / Persist | Reuse DF in memory |
| Broadcast join | Small table to all nodes — avoid shuffle |
| Salting | Fix data skew on hot keys |
| Predicate / column prune | Read less data |
| Prefer Parquet/Delta over CSV | Columnar + compression |
| Z-ORDER / OPTIMIZE (Delta) | Improve data skipping |
| Avoid collect on large data | Driver OOM risk |
| Right cluster size / autoscaling | Cost + SLA |

**Pandas vs PySpark:** Pandas = single machine; PySpark = distributed.

---

## 16. Joins in PySpark/SQL

Inner, Left, Right, Full Outer, Cross, Left Anti, Left Semi.

---

## 17. Magic commands / views (quick answers)

**Views:** saved queries / virtual tables (temp or permanent); don’t always store data physically (unless materialized).  
**Magic commands:** notebook shortcuts like `%sql`, `%python`, `%run`, `%fs`, `%md`.
