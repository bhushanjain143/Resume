# 07 — Power BI, Data Flow Gen2, Triggers Detail, Git & CI/CD

---

## 1. Power BI module (day-wise from curriculum)

| Day | Topic |
|-----|-------|
| 1 | Intro Power BI & install Desktop |
| 2 | Semantic modeling in Fabric & Power BI |
| 3 | Design visualizations |
| 4 | Filters, Bookmarks, DAX |
| 5 | Security & publishing |

**Extra timing notes in class:**
- Visualizations ~2 days  
- Power Query Editor ~2 days  
- Deployment & security ~1 day  
- DAX ~2 days  

### Report vs Dashboard
| | Report | Dashboard |
|--|--------|-----------|
| Content | One file, multiple pages/visuals | Pin visuals from **different** reports |
| Depth | Interactive multi-page analysis | At-a-glance monitoring |

---

## 2. Power Query vs Data Flow Gen2

| Tool | Transforms | Destination |
|------|------------|-------------|
| Power BI Desktop Power Query | Similar UI | Mainly Power BI model |
| Fabric **Data Flow Gen2** | Power Query style (~98% similar) | Lakehouse, Warehouse, Azure SQL, SharePoint, etc. (~6 sinks) |

ADF Mapping Data Flows ≈ code-gen Spark transforms in cloud.  
Fabric Data Flow Gen2 ≈ Power Query Gen2 in Fabric pipelines.

---

## 3. Gold layer / missing records in Power BI

If Gold has 100 expected rows but BI shows fewer:
1. Check pipeline row counts Bronze→Silver→Gold  
2. Check filters on report / RLS  
3. Check incremental refresh windows  
4. Compare keys — anti-join expected vs actual  
5. Check nulls dropped in transforms  
6. Validate semantic model relationships (many-to-many / blank rows)

---

## 4. Deployment — Fabric

```
Dev Workspace  → items (Pipelines, Lakehouse, Notebooks…)
Test / UAT Workspace
Prod Workspace

Use Fabric Deployment Pipelines
```

---

## 5. Deployment — Azure + Git

```
Azure DevOps / Git Repo
  → Data Factory (ARM/JSON)
  → Databricks (repos / jobs / notebooks)
  → Fabric (git integration where enabled)

CI/CD: Dev → UAT → Master → Prod
```

**Git topics in curriculum:**
- Day 1: What is Git Repo  
- Day 2: Deploy ADF, Databricks, Power BI reports  
- Day 3: Complete flow design Azure & Fabric  

Typical ADF Linked Service for Git: Azure DevOps Git / GitHub (not “GitLab dataset type” — Git is for repo CI, not copy dataset).

---

## 6. Triggers deep dive (keep for interviews)

### 1) Schedule Trigger
- Cron-like: minutes/hours/weeks/months at fixed time (e.g. 8 AM)  
- If server down at 2 PM → that run **skipped**  
- If 1 PM still running → 2 PM can still start (overlap possible unless concurrency limited)

### 2) Event Trigger
- Blob created/deleted → pipeline starts  
- Use when client drops file and you must react immediately  

### 3) Tumbling Window
- Contiguous non-overlapping time slices  
- Supports **backfill** of historic financial data  
- Can depend on previous window success  
- If down at 2–3 PM window → runs that window when back  

### 4) Manual
- Debug / ad-hoc  

**Interview picks:**
- Historic catch-up / finance → Tumbling Window  
- Daily 5 AM except weekends → Schedule with weekly recurrence (exclude Sat/Sun) or Logic App / If activity checking day  
- File arrival → Storage Event  

---

## 7. Dataset types in ADF (catalog)

1. **Azure:** Blob, ADLS Gen1/Gen2, Azure SQL, Synapse  
2. **Databases:** SQL Server, Oracle, MySQL, PostgreSQL, DB2, Redshift  
3. **Files:** S3, GCS, FTP/SFTP, local FS  
4. **Protocols:** HTTP, OData, ODBC, REST  
5. **NoSQL:** Cassandra, MongoDB, Couchbase, DynamoDB  
6. **SaaS:** Salesforce, Dynamics 365, ServiceNow, BigQuery, etc.  

---

## 8. Logic Apps / Azure Functions

Used with Web Activity for:
- Email notifications  
- Custom alerts  
- Lightweight orchestration hooks  

---

## 9. End-to-end “complete flow” (study map)

```
On-prem / SaaS sources
  → ADF / Fabric Pipelines (ingest)
  → ADLS / Lakehouse Bronze
  → Databricks / Notebooks Silver
  → Delta / Warehouse Gold
  → Semantic model
  → Power BI Report / Dashboard
  → Git + Deployment Pipelines (Dev/UAT/Prod)
```

---

## 10. Program-2 reminder (placement track)

- 200+ tasks with solutions (weekends)  
- 2 projects: Azure + Fabric  
- Resume + mock interviews  
- Sprints, performance optimization, Power BI reports  
- DP-700 certification push (client often mandatory)  
