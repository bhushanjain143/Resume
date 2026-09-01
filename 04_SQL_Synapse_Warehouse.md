# 04 — SQL Server, Azure SQL, Synapse DWH & Fabric Warehouse

---

## 1. Three places to create SQL databases

1. **SQL Server on local computer** (SSMS)  
2. **Azure SQL Database** (portal + Query Editor)  
3. **Fabric SQL Database**  

**Install path (class):**
- SQL Server 2025 download (Microsoft landing page)  
- SSMS from Drive link  
- Create sample DB, tables, insert data  

**Connect via:**
- Local: SQL Server Management Studio  
- Azure SQL: Query Editor / SSMS  
- Synapse DWH: SSMS / Synapse studio  

In real time: **Azure Admin** often creates servers/options.

---

## 2. SQL Database vs Warehouse (core distinction)

| | SQL Database (Azure / Fabric / On-prem) | Warehouse (Synapse Dedicated / Fabric Warehouse) |
|--|----------------------------------------|--------------------------------------------------|
| Data type | Direct business **raw** transactions | Cleaned / transformed **historical** analytics data |
| Workload | **OLTP** — insert, update, delete | **OLAP** — select / reporting |
| Speed focus | Fast writes | Fast analytical queries |
| Compute unit | **DTU** / **vCore** | **DWU** (Synapse); Fabric capacity |
| Example | Billing app writes sales now | Dashboard queries last 5 years |

```
SQL Database (customer) — 100 rows — 1 VM — ~1 hour (illustrative)
Synapse DWH (customer)  — 100 rows — many compute nodes — ~6 min (illustrative)
```

**DTU / DWU relation taught:** `5 DTU power ≈ 1 DWU` (institute figure — verify with Microsoft docs for exams).

---

## 3. Azure SQL Database compute options

| Option | Meaning |
|--------|---------|
| **DTU** | Data Transaction Units — bundled CPU/RAM/IO unit |
| **vCore** | Virtual cores — clearer CPU allocation |
| **Serverless** | Pay for compute when used |
| **Provisioned** | Fixed compute; pay even if idle |
| **Elastic pool** | Shared compute/storage pool across multiple DBs |

**Logical server example:**
```
eclasesssql1987
  └── eclasessdbindia — Logical Server / DB
```

**Purpose in project:** store supermarket billing raw transactions (OLTP).

---

## 4. Synapse Dedicated SQL Pool — Architecture

```
Control Node  → brain: receives request, returns final result
MPP Engine    → Massive Parallel Processing: splits query into tasks
Compute Nodes → 0–64 VMs execute tasks
Distributions → how data is shared across nodes
```

### Distribution methods

| Distribution | Best for | Behavior |
|--------------|----------|----------|
| **ROUND_ROBIN** (default often) | Medium dimension tables (Customer, Product, Store) | Rows assigned round-robin |
| **HASH(column)** | Large fact tables (Sales) | Rows grouped by hash of column |
| **REPLICATE** | Tiny lookup tables (City master ~5 rows) | Full copy on every node |

**Why REPLICATE helps joins:** lookup sits on every compute node → less data movement.

### Example DDL

```sql
CREATE TABLE dbo.Customer
(
    CustomerID INT,
    CustomerName VARCHAR(100),
    Gender VARCHAR(10),
    Sales DECIMAL(10,2),
    City CHAR(3)
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    HEAP
);

CREATE TABLE dbo.Customer
(
    CustomerID INT,
    CustomerName VARCHAR(100),
    Gender VARCHAR(10),
    Sales DECIMAL(10,2),
    City CHAR(3)
)
WITH
(
    DISTRIBUTION = HASH(City),
    HEAP
);

CREATE TABLE dbo.CityInfo
(
    city_code VARCHAR(10),
    city_name VARCHAR(100)
)
WITH
(
    DISTRIBUTION = REPLICATE,
    HEAP
);
```

Join example:
```sql
SELECT *
FROM customer AS c
INNER JOIN cityinfo AS ct
  ON c.citycode = ct.citycode;
```

---

## 5. Synapse Serverless — External table pattern (from notes)

Serverless queries files on ADLS; no dedicated warehouse storage for the data itself.

```sql
CREATE DATABASE SalesDB;
USE SalesDB;

-- Step 1: External Data Source
CREATE EXTERNAL DATA SOURCE LakeStorage
WITH (
    LOCATION = 'https://eclasesslake123445.dfs.core.windows.net/sales'
);

-- Step 2: External File Format
CREATE EXTERNAL FILE FORMAT CsvFileFormat
WITH (
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS (
        FIELD_TERMINATOR = ',',
        STRING_DELIMITER = '"',
        FIRST_ROW = 2
    )
);

-- Step 3: External Table
CREATE EXTERNAL TABLE dbo.Customer
(
    CustomerID INT,
    FirstName   VARCHAR(50),
    LastName    VARCHAR(50),
    Email       VARCHAR(100),
    Phone       VARCHAR(20),
    Address     VARCHAR(100),
    City        VARCHAR(50),
    State       VARCHAR(50),
    ZIPCode     VARCHAR(20),
    Country     VARCHAR(50)
)
WITH
(
    LOCATION = '2026/customer.csv',
    DATA_SOURCE = LakeStorage,
    FILE_FORMAT = CsvFileFormat
);

-- Step 4
SELECT * FROM dbo.Customer;
```

---

## 6. Fabric SQL Database & Warehouse

- Fabric SQL DB: store supermarket direct transactions; fewer admin knobs (AI-assisted).  
- Fabric Warehouse: cleaned/transformed reporting data.  
- Notebook **T-SQL** can read Lakehouse tables and write to Warehouse (DML possible).  
- Notebook **PySpark** connects well to Lakehouse files/tables; **not** directly to Warehouse for write (as taught) — use T-SQL notebook or Lakehouse tables as bridge.

**Recommended Fabric pattern:**
```
Notebook1 (PySpark): Lakehouse Files → Lakehouse Tables
Notebook2 (T-SQL):   Lakehouse Tables → Warehouse Tables
```

---

## 7. SQL topics checklist (Databricks/SQL basics module)

- DML: insert, update, delete  
- DDL: create, alter, drop  
- Operators, keywords (between, union, union all, top, distinct)  
- Clauses: where, having, group by, order by  
- Aggregates: sum, min, max, count, avg  
- Strings: left, right, trim, ltrim, rtrim, substring, replace, charindex  
- Datetime: day, getdate, year, dateadd, datediff, datepart, datename  
- Analytical: row_number(), rank(), dense_rank()  
- Joins: inner, left, right, full, cross  

Also interview favorites:
- Star schema  
- SP vs Functions  
- LIMIT vs TOP  
- PK vs Unique  
- DELETE vs DROP vs TRUNCATE  

---

## 8. Interview Q checklist (from class)

11. Diff compute tiers Azure SQL — DTU vs vCore?  
12. Serverless vs Provisioned?  
13. Elastic pool?  
14. Purpose of Azure SQL in your project?  
15. Azure SQL vs Synapse Dedicated SQL Pool?  
16. Architecture of Synapse DWH?  
17. What is MPP?  
18. Distributions available; default?  

---

## 9. Sample Customer data used in class (20 rows)

| CustomerID | Name | Gender | Sales | City |
|------------|------|--------|-------|------|
| 1 | Ravi Kumar | Male | 45000 | HYD |
| 2 | Sneha Reddy | Female | 52000 | HYD |
| 3 | Amit Sharma | Male | 38000 | DEL |
| 4 | Pooja Mehta | Female | 61000 | MUM |
| 5 | Rahul Verma | Male | 47000 | BLR |
| 6 | Anjali Singh | Female | 54000 | DEL |
| 7 | Kiran Rao | Male | 29000 | HYD |
| 8 | Neha Patel | Female | 75000 | MUM |
| 9 | Suresh Naidu | Male | 33000 | CHN |
| 10 | Divya Iyer | Female | 68000 | CHN |
| 11 | Mohan Das | Male | 41000 | BLR |
| 12 | Priya Nair | Female | 59000 | BLR |
| 13 | Arjun Malhotra | Male | 72000 | DEL |
| 14 | Kavya Joshi | Female | 36000 | MUM |
| 15 | Ramesh Goud | Male | 28000 | HYD |
| 16 | Swathi Rao | Female | 47000 | HYD |
| 17 | Nikhil Jain | Male | 51000 | DEL |
| 18 | Meera Kulkarni | Female | 64000 | BLR |
| 19 | Vinod Kumar | Male | 39000 | CHN |
| 20 | Asha Menon | Female | 56000 | CHN |
