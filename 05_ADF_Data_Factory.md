# 05 — Azure Data Factory & Fabric Data Pipelines

---

## 1. What is ETL & ADF?

**ETL:** Extract → Transform → Load  

**Azure Data Factory** = cloud ETL/orchestration service in Azure.  
**Fabric Data Pipelines** = Fabric equivalent (same scenario types, less customization).

---

## 2. Major ADF components

| Component | Role |
|-----------|------|
| **Integration Runtime (IR)** | Compute to connect & run activities |
| **Linked Service** | Connection info to a source/sink |
| **Dataset** | Pointer to specific table/file/folder |
| **Activity** | One task (Copy, Get Metadata, ForEach, If, Lookup, Web…) |
| **Pipeline** | Orchestration of activities |
| **Trigger** | Schedule / event / tumbling window / manual |
| **Parameters** | Dynamic values |

### Integration Runtimes

| IR | Use |
|----|-----|
| **Auto Resolve / Azure IR** | Cloud services (Blob, Azure SQL, ADLS…) |
| **Self-Hosted IR (SHIR)** | On-prem / private network |
| **SSIS IR** | Lift-and-shift SSIS packages |

**Fabric on-prem equivalent:** Gateway (instead of SHIR).

**Note:** If Data Factory-2 is new, reinstall SHIR using **new key** from that factory.

---

## 3. Scenario 1 — On-prem SQL → Storage Account / ADLS

```
Source: On-prem SQL Server table(s)
Target: Storage Account container → CSV
```

| Piece | Choice |
|-------|--------|
| On-prem SQL IR | Self-Hosted |
| Storage IR | Auto Resolve |
| Linked services | SQL (SHIR) + Storage (Azure IR) |
| Activity | Copy Data |

**Lab server details (class):**
```
DESKTOP-U2I7P9O\SQL2026LATEST
DB: krogersalesdb
sa / Sql@1234
```

Single table → single file: `Customer` → `Customer.csv`

---

## 4. Scenario 2 — Blob → Blob (Copy behavior)

Both source & sink: Auto Resolve IR + Linked Services for SA1 and SA2.

**Sink Copy behaviors:**

| Behavior | Effect |
|----------|--------|
| **Merge Files** | Many files → one file (e.g. 10+20+5 emp files → 35-row employee.csv) |
| **Flatten hierarchy** | Nested folders → all files in one folder |
| **Preserve hierarchy** | Keep same folder structure |

Example:
```
Source: storage1/source/2025_src/customer_jan.txt … Feb … Mar
Target: storage2/target/2025_target/customer.txt  (Merge)
```

Extra practice points:
1. Merge with override vs append  
2. Remove `""` quotes in target while copying  
3. Merge including subfolders (`2025/3 files` + `2026/1 file`)  

---

## 5. Multiple files → SQL patterns

### Same schema → one table
Many CSVs same columns → one Azure SQL table.

### Different schema → different tables
`employee.csv` → employee; `dept.csv` → dept.

### Pattern with Get Metadata + ForEach
```
Get Metadata (child items / file names)
  → ForEach
      → Copy Data (dynamic file → dynamic table)
```

**Get Metadata** returns: file names, exists?, item type, last modified.

**Lookup** activity: run queries / read file rows to feed next steps.

### Multiple tables → multiple files (on-prem → blob)
```sql
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE table_name IN ('EMP','EMPHISTORY','Department');
```
Lookup table list → ForEach → Copy each table to CSV.

---

## 6. If file exists → copy else email

```
Get Metadata (exists)
  → If Condition
      True  → Copy Data
              (optional upsert / truncate-load / SCD)
      False → Web Activity → Logic Apps email
```

**Email body parameters example (from notes):**
```json
{
  "DataFactoryName": "@{pipeline().DataFactory}",
  "PipelineName": "@{pipeline().Pipeline}",
  "Subject": "An error has occurred!",
  "ErrorMessage": "The file is missing in blob storage. Plz place file.",
  "EmailTo": "admin@iclasess.com"
}
```

Schema for Logic App payload:
```json
{
  "properties": {
    "DataFactoryName": { "type": "string" },
    "EmailTo": { "type": "string" },
    "ErrorMessage": { "type": "string" },
    "PipelineName": { "type": "string" },
    "Subject": { "type": "string" }
  },
  "type": "object"
}
```

**Upsert ideas when file exists:**
- Truncate then load  
- If row exists → update; else insert (SCD1 / upsert)  

---

## 7. Incremental loading (watermark pattern)

**Goal:** Don’t full-load every day — only new/changed rows.

### Control / watermark tables

```sql
CREATE TABLE dbo.ETLControl
(
    TableName VARCHAR(100) PRIMARY KEY,
    LastRunDate DATETIME
);

INSERT INTO dbo.ETLControl VALUES ('Customer','1999-01-01 00:00:00');
```

```sql
CREATE TABLE watermarktable (
  TableName varchar(255),
  WatermarkValue datetime
);

INSERT INTO watermarktable
VALUES ('data_source_table','1/1/1990 12:00:00 AM');
```

### Source with ModifiedDate

```sql
INSERT INTO Customer (...)
VALUES (119,'Kevin',...), (120,'Linda',...);

UPDATE Customer
SET city='Banaglore', ModifiedDate='2026-07-12 05:40:00.000'
WHERE customerid=106;
```

### Pipeline idea
1. Lookup old watermark  
2. Copy where `LastModifytime > old_watermark`  
3. Stored proc / activity to update watermark to `MAX(LastModifytime)`

```sql
SELECT * FROM data_source_table
WHERE LastModifytime >
  '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}';

CREATE PROCEDURE updateinfo AS
BEGIN
  UPDATE watermarktable
  SET WatermarkValue = (SELECT MAX(LastModifytime) FROM data_source_table)
  WHERE tablename = 'data_source_table';
END;
```

### Fabric SQL watermark example

```sql
CREATE TABLE WatermarkTable (
    TableName VARCHAR(100) PRIMARY KEY,
    LastModifiedDate DATETIME
);

INSERT INTO WatermarkTable (TableName, LastModifiedDate)
VALUES ('Customer', DATEADD(DAY, -2, GETDATE()));

CREATE PROCEDURE sp_updatewatermark
AS
BEGIN
  UPDATE WatermarkTable SET LastModifiedDate = GETDATE();
END;
```

### Demo Customer incremental script

```sql
CREATE TABLE Customer (
    Id INT PRIMARY KEY,
    Name VARCHAR(100),
    Gender VARCHAR(10),
    Salary DECIMAL(10,2),
    City VARCHAR(50),
    LastModifiedDate DATE
);

INSERT INTO Customer VALUES
(1,'Ravi Kumar','Male',55000,'Hyderabad', DATEADD(DAY,-2,GETDATE())),
(2,'Sneha Reddy','Female',62000,'Bangalore', DATEADD(DAY,-2,GETDATE())),
(3,'Arjun Rao','Male',48000,'Chennai', DATEADD(DAY,-2,GETDATE()));

-- New + modified
INSERT INTO Customer VALUES
(4,'Lakshmi Devi','Female',70000,'Vijayawada', GETDATE()),
(5,'Kiran Kumar','Male',53000,'Visakhapatnam', GETDATE());

UPDATE Customer SET City='Pune', LastModifiedDate=GETDATE() WHERE Id=2;

SELECT * FROM customer WHERE LastModifiedDate > '2026-04-27';
```

---

## 8. Until activity (wait for file)

```
If file exists → Copy
Else → Until (wait until file placed) → then Copy
```

---

## 9. Triggers (ADF)

| Trigger | Use |
|---------|-----|
| **Schedule** | Daily 9 PM / every hour, etc. Skipped if window missed while down |
| **Storage Event** | Fire when file created/deleted in blob |
| **Tumbling Window** | Fixed contiguous windows; **backfill** historic gaps; can wait for previous window |
| **Manual** | Debug / on-demand |

**Tumbling window intuition:** pipeline off all January → turn on Feb 1 with start Jan 1 → runs each missed hour until caught up. Critical for financial/historic completeness.

**Fabric:** Schedule on Data Pipelines / Jobs (less “trigger type” jargon).

---

## 10. Data Flows

| Platform | Transform UI |
|----------|--------------|
| ADF | Mapping Data Flows |
| Fabric | **Data Flow Gen2** (Power Query Editor style) |

Power Query Editor (Power BI Desktop) → load mainly into Power BI.  
Data Flow Gen2 → can load to ~6 destinations (Azure SQL, Lakehouse, Warehouse, SharePoint, etc.).

---

## 11. ADF vs Fabric Pipelines (summary)

| Topic | Azure ADF | Fabric |
|-------|-----------|--------|
| Name | Data Factory | Data Pipelines |
| On-prem connect | Self-Hosted IR | Gateway |
| Customization | High | Low / AI |
| Warehouse dist. | Round robin / Hash / Replicate | AI-managed |
| Email notify | Web + Logic Apps | Similar patterns |
| Scheduling | Triggers | Schedule / jobs |

Same business scenarios implementable on both.

---

## 12. Dynamic file routing example (cust vs dept)

Copy `cust.csv` → cust dir; `dept.csv` → dept dir:

```
Get Metadata (child items)
  → ForEach
      → If Condition (filename contains cust / dept)
          → Copy to matching folder
```

---

## 13. Lookup + Union table list pattern

```sql
SELECT 'customer' AS tablename
UNION ALL
SELECT 'product' AS tablename;
```
Lookup output → ForEach → Copy.

---

## 14. Activities frequently asked in interviews

Copy, Get Metadata, ForEach, If Condition, Until, Lookup, Web, Stored Procedure, Delete, Data Flow, Wait, Set Variable, Execute Pipeline.

**HSBC-style:** run only 5 of 10 activities → use If/Switch, disable activities, or separate pipelines / Execute Pipeline with parameters; no dependency after step 5.
