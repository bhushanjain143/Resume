# 08 — Interview Q&A Bank (Company-wise + Concept Answers)

All questions from your notes are kept. **Missing or broken answers are completed** below.

---

## A. Your personal intro bank (from email drafts)

### Headline options
- Immediate Joiner | Data Engineer | ETL & Big Data | PySpark | SQL | AWS | Azure | Medallion | 2+ Years  
- Serving notice | Data Engineer | Azure Certified | Oracle Certified | PySpark • ADF • Databricks • Kafka • SQL  

### Short intro (keep truthful to your real experience)
> I am a Data Engineer with 3+ years designing scalable pipelines in telecom/enterprise domains. I build ETL with PySpark, SQL, AWS/Azure. I implement Medallion (Bronze/Silver/Gold), process large datasets, and use Spark, Hive, Hadoop, S3/ADLS, with performance tuning.

### Stronger Azure-focused intro (sample from notes)
> Data Engineer with 3+ years on Azure. Migrated on-prem Oracle/SQL Server to cloud; built ETL/ELT on Databricks/ADF; Kafka streaming to Delta; Airflow/ADF orchestration; star/snowflake models; Azure DevOps CI/CD.

**Core skills to mention:** ADF, Databricks, ADLS Gen2, Synapse, PySpark, SQL, Python, Medallion, Delta, CI/CD.

### Project storytelling checklist
Items **1–16** below are the exact interview flow from your notes — with full ready answers.

---

## A2. Interview flow 1–16 (FULL ANSWERS)

### 1. Intro
**Say (30–45 sec):**  
I am a Data Engineer with 3+ years experience building cloud ETL/ELT pipelines on **Azure (ADF, Databricks, ADLS Gen2)** and also exposure to AWS. I work on **Medallion Architecture (Bronze → Silver → Gold)**, transform large datasets with **PySpark/SQL**, and deliver analytics-ready tables for BI. I focus on incremental loads, data quality, and performance/cost optimization.

---

### 2. Projects
**Structure (STAR):**  
- **Domain:** Telecom / enterprise sales (pick your real project)  
- **Source:** On-prem SQL / Oracle / Kafka / files  
- **Ingest:** ADF Copy + Self-Hosted IR (or Auto Loader) → **ADLS Gen2 Bronze**  
- **Transform:** Databricks notebooks (PySpark) → **Silver** (clean, dedupe, join) → **Gold** (aggregates / star schema)  
- **Serve:** Synapse / Delta Gold → **Power BI**  
- **Impact:** reduced manual jobs, faster refresh, incremental instead of full load  

Have **1 end-to-end project** ready + 1 optimization story (partition, broadcast, job cluster).

---

### 3. DAG, Lineage

| Term | Meaning | How you use it |
|------|---------|----------------|
| **DAG** | Directed Acyclic Graph — Spark’s execution plan of stages/tasks with no cycles | Built when an **action** runs (`write`, `count`). Check Spark UI → Jobs/Stages |
| **Lineage** | Track where data came from and which notebook/table produced it | Unity Catalog lineage; Delta history; ADF pipeline run lineage |

**Answer:**  
In Spark, transformations are lazy and form a DAG; an action triggers job → stages → tasks. Lineage tells us Bronze file → Silver table → Gold table → report, useful for impact analysis and debugging.

---

### 4. How to show data to end user / business — Power BI team
**Flow:**  
Gold Delta / Warehouse tables → **Semantic model** (relationships, measures) → Power BI **Dataset** → Reports/Dashboards → publish to workspace → share with business (RLS if needed).

**Your role vs Power BI team:**  
- DE: reliable Gold tables, incremental refresh-ready, documented grain/keys  
- PBI team: visuals, DAX, workspace access  
- Handoff: table names, refresh SLA, row counts, data dictionary  

---

### 5. ETL
**ETL = Extract → Transform → Load**  
- **Extract:** ADF Copy / Auto Loader from SQL, files, APIs  
- **Transform:** Databricks PySpark (or ADF Data Flow) — clean, join, SCD  
- **Load:** write to Delta / Synapse / Azure SQL  

Also mention **ELT**: land raw first (Bronze), transform in lake/warehouse.

---

### 6. Narrow / Wide transformations

| Type | Shuffle? | Examples |
|------|----------|----------|
| **Narrow** | No (data stays on same partition) | `map`, `filter`, `withColumn`, `select` |
| **Wide** | Yes (data moves across executors) | `groupBy`, `join`, `distinct`, `repartition` |

**Answer:** Narrow is cheaper; wide causes shuffle — optimize joins (broadcast), partition keys, and avoid unnecessary wide ops.

---

### 7. ADF
**Azure Data Factory** orchestrates pipelines.  
**I use:** Integration Runtime (Azure + Self-Hosted), Linked Services, Datasets, Activities (Copy, Get Metadata, ForEach, If, Lookup, Stored Proc, Web, Data Flow), Triggers (Schedule / Event / Tumbling).  

**Typical pipeline:** Get Metadata → If file exists → Copy to ADLS → execute Databricks notebook → update watermark.

---

### 8. Which format is good to read/write — Parquet or CSV? (also Delta vs Parquet)

| Format | Best for |
|--------|----------|
| **CSV** | Human exchange, small files, source landing only |
| **Parquet** | Analytics read/write — columnar, compressed, predicate pushdown |
| **Delta** | Lakehouse tables needing ACID, upserts, time travel |

**Interview answer:**  
For processing and storage in the lake I prefer **Parquet**; for curated tables I use **Delta (Parquet + transaction log)**. CSV only for raw ingest/handoff — not for heavy read/write.

---

### 9. Difference: Delta vs Parquet

| | Parquet | Delta Lake |
|--|---------|------------|
| What | Columnar file format | Table format on top of Parquet |
| ACID | No | Yes (`_delta_log`) |
| UPDATE/MERGE | Hard (rewrite files manually) | Native `MERGE` / `UPDATE` / `DELETE` |
| Time travel | No | Yes (version / timestamp) |
| Schema enforcement | Weak alone | Strong |
| Concurrent writes | Risky | Transactionally handled |

**One line:** Delta = Parquet files + transaction log for reliability.

---

### 10. Execute 10 notebooks in one go
**Options:**  
1. **Databricks Job / Workflow** — multi-task job; tasks = notebooks; set dependencies (1→2→… or parallel). Can define via UI or **YAML/JSON** job definition in CI (Azure DevOps / Databricks Asset Bundles).  
2. **`dbutils.notebook.run()`** from a master notebook:

```python
for nb in [
  "/Repos/bronze_load",
  "/Repos/silver_clean",
  # ... up to 10
]:
    dbutils.notebook.run(nb, timeout_seconds=3600, arguments={"env": "prod"})
```

3. **`%run ./notebook`** — same cluster, shares context (good for libs; weaker isolation).  

**Prefer:** Workflow/Job with YAML for production; `run()` for simple orchestration.

---

### 11. How a job is executed in Databricks
1. Job triggered (manual / schedule / API / ADF)  
2. **Job cluster** (or existing all-purpose) starts  
3. Tasks run as notebooks/JAR/Python — each task = Spark **application**  
4. Driver builds **DAG**; cluster manager schedules stages on executors  
5. Tasks succeed/fail → retries per policy → cluster terminates (job cluster)  
6. Run history, logs, Spark UI available in Jobs UI  

---

### 12. How to schedule a job in Databricks
**Jobs → Schedule → Quartz cron / simple timeframe**  
Examples: daily 2 AM, every hour, Mon–Fri 5 AM.  
Also: trigger from **ADF** (Databricks Notebook activity) or REST API.  
Set timezone, concurrency, email alerts, timeout, retry.

---

### 13. SCD in ADF — perform atomicity
**SCD1:** overwrite changed attributes.  
**SCD2:** keep history (end-date old row, insert new version).  

**Atomicity approaches:**  
1. **Staging + MERGE (best):** Copy to staging table → single **Stored Procedure** / Synapse / SQL `MERGE` (one transaction) → success or full rollback.  
2. **Databricks Delta `MERGE`:** ADF runs notebook; Delta transaction is atomic.  
3. **Data Flow Alter Row** upsert — prefer wrapping with staging so partial failure doesn’t corrupt target.  
4. Avoid many row-by-row updates without a transaction boundary.

**Answer line:** “We land to staging, then apply one atomic MERGE (SQL or Delta) so SCD never leaves half-updated target.”

---

### 14. While creating pipeline, what activities are you using?

| Activity | Why |
|----------|-----|
| **Get Metadata** | File exists? child items? last modified |
| **Lookup** | Read watermark / table list |
| **ForEach** | Loop files/tables |
| **If Condition / Switch** | Branch logic |
| **Copy Data** | Source → sink |
| **Delete** | Cleanup / truncate pattern |
| **Stored Procedure** | Update watermark / MERGE SCD |
| **Databricks Notebook** | Heavy transforms |
| **Web** | Logic App email alert |
| **Until** | Wait until file arrives |
| **Set Variable** | Dynamic paths/names |
| **Data Flow** | UI transforms when needed |

---

### 15. What file type are you using — batch or streaming? (comma separated)
**Answer:**  
Mostly **batch** files: **CSV, Parquet, JSON** (comma-separated / delimited for CSV).  
Streaming when needed: **Kafka + Spark Structured Streaming** / Auto Loader (cloudFiles) writing to **Delta**.  

**One line:** “Daily batch CSV/Parquet on ADLS; near-real-time Kafka → Delta when SLA needs streaming.”

---

### 16. How to sync 2 files and execute with low cost + optimization
**Problem:** Pipeline should run only when **both** files are present; keep cost low.

**Pattern:**  
1. **Event trigger** on container OR schedule every N minutes (prefer event).  
2. **Get Metadata** on `file_A` and `file_B` (exists).  
3. **If** both exist → Copy / process; else **Until** (wait with sleep) or exit success without heavy cluster.  
4. **Do not** start Databricks until both files validated.  

**Low cost optimizations:**  
- Use **job cluster** (auto-terminate), smallest SKU that meets SLA  
- Process **Parquet/Delta**, not repeated CSV full scans  
- **Partition prune** / read only needed columns  
- **Broadcast** small lookup  
- Merge files once; avoid re-copy  
- Tumbling/event so idle compute isn’t running  

**Answer line:** “Validate both files with Get Metadata/Until, then start one short-lived job cluster to process Parquet/Delta with partition pruning.”

---

## B. Concept answers (high frequency)

### Storage
| Q | A |
|---|---|
| Blob vs ADLS Gen2 | Blob = cheap/flat/backup; Gen2 = HNS + analytics performance |
| Soft delete | Retain deleted blob/container for N days; undelete |
| Versioning vs Snapshot | Auto versions on change vs manual point-in-time copy |
| Blob types | Block, Page, Append |
| Access tiers | Hot, Cool, Cold, Archive, Smart |
| Lifecycle | Auto tier/delete by age rules |
| Security | RBAC (users); Keys/ConnString/SAS (apps) |
| Upgrade | Blob→Gen2 yes; reverse no |

### Azure SQL vs Synapse
| Q | A |
|---|---|
| Purpose Azure SQL | OLTP raw transactions |
| Synapse Dedicated | OLAP cleaned warehouse; MPP |
| DTU vs vCore | Bundled unit vs explicit cores |
| Serverless vs Provisioned | Pay-per-use vs reserved |
| Elastic pool | Shared compute across DBs |
| Distributions | Round robin, Hash, Replicate |

### ADF
| Q | A |
|---|---|
| Components | IR, Linked Service, Dataset, Activity, Pipeline, Trigger, Params |
| SHIR | Agent on-prem for private sources |
| Copy path | Linked services → datasets → Copy activity |
| Incremental | Watermark / LastModified filter + update control table |
| Triggers | Schedule, Event, Tumbling, Manual |
| Activities used | Copy, GetMeta, ForEach, If, Lookup, Web, SP, Data Flow |

### Spark / Databricks
| Q | A |
|---|---|
| Architecture | Driver, cluster manager, workers, executors, tasks, DAG |
| Lazy eval | Transforms plan; actions execute |
| RDD vs DF | DF faster with Catalyst/Tungsten |
| Delta vs Parquet | Delta adds ACID, time travel, MERGE log |
| Managed vs External | Drop deletes data vs metadata only |
| Autoloader | Incremental cloud file ingest + checkpoint |
| Unity Catalog | Governed catalog.schema.table/volume |
| Lakehouse vs Data Lake | Lakehouse = lake files + table ACID/governance; lake = files mainly |
| Is Databricks lake or lakehouse? | Platform that implements lakehouse on cloud storage |
| Null handling | dropna, fillna, coalesce, business rules |
| Skew | salting, AQE, broadcast, repartition by key |
| Broadcast | small dimension to all executors |
| Cache vs Checkpoint | memory reuse vs truncate lineage to reliable storage |
| groupByKey vs reduceByKey | reduceByKey aggregates before shuffle — better |
| `.collect()` | brings data to **driver** |

### Power BI / Gold
Missing rows → validate counts, filters, RLS, joins, incremental window.

---

## C. Deloitte

**Highest salary per dept**
```sql
SELECT dept, MAX(salary) AS high_salary
FROM employee
GROUP BY dept;
```

**Employees earning more than manager**
```sql
SELECT e.emp_name AS emp, e.salary AS emp_salary,
       m.emp_name AS manager, m.salary AS mngr_salary
FROM employee e
JOIN employee m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

Other asked: current project, delta vs parquet, copy source→sink, CI/CD Dev-UAT-Prod, SCD1 vs SCD2, null transforms, Logic Apps, which DWH, incremental load, long Databricks job automation, filter `xyz` files from 100 in ADLS.

**SCD1 vs SCD2:** SCD1 overwrites; SCD2 keeps history with effective dates / is_current.

---

## D. eClerx

Employees earning more than department average:

```sql
SELECT e.name, e.department, e.salary, d.avg_salary
FROM emp e
JOIN (
  SELECT department, AVG(salary) AS avg_salary
  FROM emp
  GROUP BY department
) d ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

```python
from pyspark.sql import Window
from pyspark.sql import functions as F

w = Window.partitionBy("department")
df1 = (df.withColumn("avg_dept_salary", F.avg("salary").over(w))
         .filter(F.col("salary") > F.col("avg_dept_salary"))
         .select("name", "department", "salary"))
```

Skills mentioned: AWS, ADF, Databricks, ADLS Gen2.

---

## E. EY

Topics: Project; copy Azure SQL→cloud; linked service types; dataset types; Pandas vs PySpark; Spark architecture; Silver layer storage; Lazy eval / Action vs Transform; Managed vs External Delta; Autoloader; `.collect()` on driver.

**Total order amount per customer**
```sql
SELECT customer_id, SUM(amount) AS total_amt
FROM orders
GROUP BY customer_id;
```
```python
df1 = df_order.groupBy("customer_id").agg(F.sum("amount").alias("total_amt"))
```

---

## F. Generic Azure DE round (from notes)

- Intro/project  
- Azure SQL?  
- Copy Azure SQL → Data Lake  
- Cluster types  
- Unity Catalog  
- Lakehouse vs data lake  
- Delta Lake  
- Incremental in Databricks (Auto Loader, MERGE, Change Data Feed, watermarks)  
- ADLS vs Blob  
- Null handling  
- Rank salary dept-wise:

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F

w = Window.partitionBy("dept").orderBy(F.col("salary").desc())
r_sal = df.withColumn("ranked_sal", F.rank().over(w))
r_sal.display()
```

---

## G. TCS

**SQL basics**
| Topic | Answer |
|-------|--------|
| Star schema | Fact + dimensions |
| SP vs Function | SP: procedures/side effects/no mandatory return; Function: returns value, usable in SELECT |
| LIMIT vs TOP | LIMIT (many engines) vs TOP (SQL Server) |
| PK vs Unique | PK: one, not null; Unique: allows one NULL (SQL Server) |
| DELETE/DROP/TRUNCATE | rows / remove object / deallocate all rows fast |

**3rd highest salary**
```sql
SELECT salary
FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank1
  FROM emp
) t
WHERE rank1 = 3;
```

**Gold missing records:** reconcile counts, anti-join keys, check filters/RLS.

Process: TCS round → Client (Canada BFSI) video; focus SQL, PySpark, Databricks fundamentals.

---

## H. Accenture

| Q | A |
|---|---|
| Data warehousing | Subject-oriented, integrated, time-variant store for analytics |
| coalesce | First non-null expression / Spark coalesce partitions |
| repartition | Full shuffle to N partitions |
| SCD / SCD2 | History tracking with versions |
| Azure services | ADF, ADLS, Databricks, SQL, Synapse… |
| Connect DBX–Blob | mount / abfss / UC external location |
| Path to destination | often curated → Azure SQL / Synapse / Gold Delta → BI |
| Cost optimization | cache wisely, partition prune, AQE, right SKU, job clusters |
| Mount | `dbutils.fs.mount(...)` |
| Read CSV | `spark.read.format("csv").option("header",True).option("inferSchema",True).load(path)` |
| Flatten JSON | `from_json` + explode / `pandas.json_normalize` then to DF |
| Unity Catalog | governance layer |
| RDD vs DF | DF optimized |

**Coding**
1. Group employees by dept  
2. Tuple descending: `sorted(t, reverse=True)` or bubble  
3. Job every 10 min; letter size 3; digit size 8 — validate with regex + Jobs schedule:

```python
import re
def valid(s):
    return bool(re.fullmatch(r"[A-Za-z]{3}\d{8}", s))
```

---

## I. Apexon

| Q | A |
|---|---|
| Spark structure | Driver/executors/DAG/stages |
| Tungsten | Off-heap / whole-stage codegen engine |
| groupBy vs reduceByKey | reduceByKey better for PairRDD aggregates |
| cache vs checkpoint | memory vs break lineage |
| Managed vs external speed | similar compute; managed simpler lifecycle |
| Aggregate vs window | collapse rows vs keep rows with analytic calc |
| DF vs RDD speed | DF usually faster |
| Slow query | Spark UI stages, skew, shuffle, file sizes, EXPLAIN |

**SQL order:** FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY  

**Join counts puzzle (Table1×Table2 as given in notes):**  
Use multiplicity of matching keys (including NULLs never match in SQL). Practice counting carefully; answers they listed: inner 11, left 15, right 14, full 18 — verify against actual multiplicity if asked live.

**Islands problem (clm2=1 consecutive groups length>1):**
```sql
WITH GetResult AS (
  SELECT clm1, clm2,
         clm1 - ROW_NUMBER() OVER (PARTITION BY clm2 ORDER BY clm1) AS grp
  FROM table1
)
SELECT clm1
FROM GetResult
WHERE clm2 = 1
  AND grp IN (
    SELECT grp FROM GetResult
    WHERE clm2 = 1
    GROUP BY grp
    HAVING COUNT(*) > 1
  );
-- Expected sample o/p: 3,4,5
```

**IN vs EXISTS:** EXISTS often short-circuits; `IN (1,2,NULL)` three-valued logic pitfalls; prefer `IS NULL` explicit.

---

## J. Incedo

End-to-end pipeline design; optimization; skew; RDD vs Dataset; Azure services; pipeline failure triage (ADF monitor, retry, dead-letter, alerts); lazy evaluation.

---

## K. HSBC

| Q | A |
|---|---|
| Run only 5 of 10 | Separate pipeline / If false stops / concurrency & dependency off |
| Optimization | partitions, broadcast, predicate pushdown, Delta OPTIMIZE |
| Where IR lives | Azure IR in cloud; SHIR on your VM/network |
| DF pipeline vs ADF | Spark code transforms vs ADF orchestration (+ mapping DF) |
| Historic finance trigger | Tumbling window |
| Daily 5 AM except weekend | Schedule Mon–Fri |
| PySpark disadvantages | Cluster cost, latency for small data, JVM complexity |
| Skew + salting | add random salt to hot keys then unsalt |
| Architecture | explain driver/workers |
| Palindrome | see coding file |
| Next salary | `LEAD(salary) OVER (ORDER BY salary)` |

---

## L. TekSystems

Delta, Databricks pipeline, ADF, incremental, optimization.

**Top 3 emp per dept**
```sql
SELECT emp_id
FROM (
  SELECT emp_id,
         ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rank1
  FROM emp
) r
WHERE rank1 <= 3;
```

**Move zeros to end:** see coding file — O(n) time, O(1) space.

---

## M. Synoptek

ADF components; configure SHIR (install gateway, register key, allow outbound); dynamic filenames via parameters + `@pipeline()` / dataset params / Get Metadata.

---

## N. Mphasis

Join counts on their Tab1/Tab2 (practice NULL matching).  
3rd highest salary per dept with dense_rank — see coding file.

---

## O. Sulopa

Project/role; Databricks vs open Spark (managed UC, notebooks, jobs, Delta); Delta Lake; Auto Loader; salting; broadcast; medallion (they asked 4 layers — Landing+Bronze+Silver+Gold).

---

## O2. Citi Bank

- Package noted: **~20 LPA**  
- Focus: **Python coding, Databricks, Data Structures (DS)**  
- Prep: strong Python coding + Databricks fundamentals + DSA basics.

---

## P. Common “design & ops”

| Q | A |
|---|---|
| Pipeline long running — automate | Jobs alerts, timeout, retries, cluster policy, AQE |
| 100 files get only xyz | glob `xyz*` or filter `input_file_name()` |
| DML in ADF | Stored Proc activity, Mapping DF alter row, Script activity, pre-copy script |
| File batch vs streaming | batch micro-batches vs Structured Streaming/Kafka |
| Sync 2 files low cost | event trigger when both arrive (Get Meta + Until), small cluster, parquet |

---

## Q. Sample resume bullets (from notes — customize truthfully)

- PySpark ETL on Databricks processing TB-scale daily (~20% efficiency claim in sample)  
- Spark Structured Streaming + Kafka → Delta  
- ADF + Airflow orchestration (−30% manual)  
- Cost optimization (−15% compute)  
- Star/snowflake for CRM/OMS  
- Stack: Azure Databricks, ADF, ADLS, Synapse, Python, SQL, DevOps  

---

## R. Bus route passengers problem (SQL)

Passengers board a bus if same origin/destination and passenger time ≤ bus time; each passenger at most one bus (earliest feasible). Classic approach: for each passenger find min bus time matching route with time ≥ passenger time; count.

```sql
WITH assigned AS (
  SELECT p.id AS passenger_id,
         (
           SELECT TOP 1 b.id
           FROM bus_route b
           WHERE b.origin = p.origin
             AND b.destination = p.destination
             AND b.time >= p.time
           ORDER BY b.time
         ) AS bus_id
  FROM passengers p
)
SELECT b.id,
       COUNT(a.passenger_id) AS passengers_on_board
FROM bus_route b
LEFT JOIN assigned a ON a.bus_id = b.id
GROUP BY b.id
ORDER BY b.id;
-- Expected: 100→5, 200→2, 300→0
```

---

## S. Same month/year as manager

```sql
SELECT e.emp_id
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE MONTH(e.hire_date) = MONTH(m.hire_date)
  AND YEAR(e.hire_date) = YEAR(m.hire_date);
```

```python
from pyspark.sql.functions import col, month, year
e = df.alias("e")
m = df.alias("m")
result = (e.join(m, col("e.manager_id") == col("m.emp_id"))
           .filter(
             (month(col("e.hire_date")) == month(col("m.hire_date"))) &
             (year(col("e.hire_date")) == year(col("m.hire_date")))
           )
           .select(col("e.emp_id")))
```

---

## T. Age brackets / last 3 months

```sql
SELECT
  CASE
    WHEN age BETWEEN 20 AND 30 THEN '20-30'
    WHEN age BETWEEN 31 AND 40 THEN '31-40'
  END AS age_brkts,
  COUNT(*) AS cnt
FROM employees
GROUP BY
  CASE
    WHEN age BETWEEN 20 AND 30 THEN '20-30'
    WHEN age BETWEEN 31 AND 40 THEN '31-40'
  END
ORDER BY age_brkts;

-- Last 3 months (SQL Server)
SELECT * FROM employees
WHERE join_date >= DATEADD(MONTH, -3, CAST(GETDATE() AS DATE));
```
