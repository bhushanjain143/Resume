# 02 — Azure vs Microsoft Fabric

---

## 1. Big picture

| Concept | Azure | Fabric |
|---------|-------|--------|
| Portal | portal.azure.com | app.powerbi.com |
| Project container | **Resource Group** | **Workspace** |
| Software after create | **Resource / Service** | **Item** |
| Storage (files) | Storage Account (Blob / ADLS Gen2) | **Lakehouse** |
| OLTP DB | Azure SQL Database | Fabric **SQL Database** |
| Warehouse | Synapse Dedicated SQL Pool DWH | Fabric **Warehouse** |
| ETL orchestration | **Data Factory** | **Data Pipelines** |
| Transform / notebooks | **Azure Databricks** (PySpark) | **Notebooks** (PySpark / T-SQL) |
| Customization | Highly customizable — many decisions | Less customization — more AI / simplified |
| Billing model | Pay per service + time used | Capacity plan (F2, F4…F256) + storage ₹/GB |

```
Microsoft Azure  → Customizable; developer chooses options
Microsoft Fabric → Less customization; in-built AI helps pick approach
Comparing to Azure: Fabric more AI-involved, more simplified in development & pricing
```

---

## 2. Azure project model

```
Microsoft Azure → Resource Group Manager = Project
Each created service for project = Resource

Example:
eclasesssalesproject → Resource Group
  Resource — Data Factory     (~₹10/hour as demo figure)
  Resource — Storage Account  (~₹2/hour)
  Resource — Databricks       (~₹5–20/hour as demo figures)
```

**Billing (Azure):** depends on **type of service** + **how long used**.  
Free subscription credit noted as ~₹17,000 / 28 days.

```
Resource Group 1 → Project1 → DF, Databricks, Synapse, Storage
Resource Group 2 → Project2 → DF, Storage
```

**Azure services used in course:**
- Storage Account (Blob or Data Lake Gen2)
- Azure SQL Database
- Synapse Dedicated SQL Pool DWH / Synapse Analytics
- Data Factory
- Databricks

**Synapse Analytics can contain:**
- Synapse Dedicated SQL Pool
- Synapse Serverless Pool
- Data Pipelines
- Notebooks

---

## 3. Fabric project model

```
Microsoft Fabric → Workspace = Project
Capacity plan example: F2 = $256/month, then F4, F6 … F256
Storage: ~₹2 per GB per month (as taught)

Workspace1 → Free Fabric / F2 → Items (Data Pipelines, Lakehouse, Notebooks…)
Workspace2 → Paid Fabric
```

**Items:**
- Data Pipelines  
- Lakehouse  
- Notebooks  
- Warehouse  
- SQL Database  
- KQL Database (mentioned)  
- Power BI / Semantic models  

**Capacity notes:**
- Free trial capacity available  
- Capacity **< F64** → may need separate Power BI license  
- **F64+** → Power BI license included (as taught)  
- Capacity **> 64** → Copilot enabled (as taught)

**AI-powered Fabric SQL DB:** often only need database name (admin options reduced vs Azure SQL).

---

## 4. Side-by-side service map

| Purpose | Azure | Fabric |
|---------|-------|--------|
| File storage (csv, excel, audio, video…) | Storage Account Blob / ADLS Gen2 | Lakehouse |
| Raw business tables (OLTP) | Azure SQL Database | SQL Database |
| Cleaned / analytical tables (OLAP) | Synapse Dedicated SQL Pool | Warehouse |
| ETL pipelines | Data Factory | Data Pipelines |
| Clean/transform code | Databricks notebooks | Fabric Notebooks |
| Reports | Power BI | Power BI (native) |

---

## 5. End-to-end data flows (taught scenarios)

### Scenario A — Classic on-prem → Azure medallion

```
Super Market transactions
  → On-prem SQL Server
  → Data Factory (Self-Hosted IR)
  → Storage Account (ADLS Gen2) Bronze
  → Databricks (PySpark) Silver/Gold
  → Synapse DWH
  → Power BI
```

### Scenario B — Cloud SQL already in Azure

```
Super Market
  → Azure SQL Database
  → Databricks clean/transform
  → Synapse DWH
  → Power BI
```

### Scenario C — Fabric medallion

```
Super Market
  → On-prem / sources
  → Data Pipelines (Gateway for on-prem)
  → Lakehouse Bronze
  → Notebooks Silver
  → Notebooks Gold
  → Power BI
```

### Scenario D — Fabric with cloud SQL

```
SQL Database (Azure or Fabric)
  → Clean/transform (Databricks / Notebooks)
  → Warehouse
  → Power BI
```

---

## 6. How to create subscriptions & projects (checklist)

### Azure
1. Go to portal.azure.com  
2. Create free / paid subscription (Gmail + card)  
3. Create **Resource Group** = project  
4. Create resources: Storage Account, SQL, DF, Databricks, Synapse  

### Fabric
1. Go to app.powerbi.com  
2. Start Fabric trial / capacity  
3. Create **Workspace** = project  
4. Create items: Lakehouse, Pipelines, Notebooks, Warehouse, SQL DB  

**Demo sequence from notes:**
1. Install SQL Server  
2. Create free subscription Azure & Fabric  
3. Create project  
4. Create storage account — Blob  
5. Create storage account — Data Lake Gen2  
6. Watch 3 hrs SQL + 3 hrs Python  
7. Advanced storage concepts  

---

## 7. Dedicated vs Serverless (Azure concept)

| Mode | Meaning |
|------|---------|
| **Dedicated / Provisioned** | Reserve compute; pay even if idle |
| **Serverless** | Pay when used; based on usage |

**Dedicated SQL Pool:** has its own storage.  
**Serverless pool:** can query files on ADLS Gen2 via external tables; no own warehouse storage for data files.

---

## 8. Interview-ready one-liners

- **Azure** = build-your-own stack (Storage + SQL + Synapse + ADF + Databricks).  
- **Fabric** = unified SaaS workspace (Lakehouse + Warehouse + Pipelines + Notebooks + Power BI).  
- Same logical patterns (medallion, incremental load, orchestration) apply on both; tools/names differ.  
- Azure IR / Self-Hosted IR ↔ Fabric Gateway for on-prem.  
- ADF Triggers ↔ Fabric Pipeline Schedule / jobs.  
