# 03 — Storage Account, Lakehouse & Medallion Architecture

---

## 1. What is a Storage Account / Lakehouse?

| Platform | Service | Analogy |
|----------|---------|---------|
| Azure | Storage Account (Blob or ADLS Gen2) | Google Drive for any files |
| Fabric | Lakehouse | Google Drive + table layer |

Stores: text, csv, excel, audio, video, images, parquet, delta, etc.

**Azure Storage types inside a Storage Account:**
- **Containers** (Blob)  
- **File shares** — secure share of config/secrets-style files  
- **Tables**  
- **Queues**  

---

## 2. Blob vs Data Lake Gen2 (ADLS Gen2)

| Feature | Blob Storage | Data Lake Gen2 |
|---------|--------------|----------------|
| Cost | Lower | Higher |
| Use case | Backup / infrequently accessed | Big data analytics / frequent access |
| Example | 1000 old movies on HDD | Daily sales CSVs for Power BI |
| Namespace | Flat | **Hierarchical Namespace** enabled |
| Versioning | Supported | Versioning **not supported** (as taught) |
| Performance | Lower for analytics | Better for analytics processing |
| Default | Default blob storage created | Enable Hierarchical Namespace at create |

**HDD vs SSD analogy (taught):**
- HDD → Standard — less cost, less performance  
- SSD → Premium — higher cost, higher performance  

### Hierarchical namespace example

```
sales/2026/Jan/customerjan01.csv
            /customerjan02.csv
            /customerjan03.csv
sales/2026/Feb/...
```

### Upgrade rules
| From → To | Possible? |
|-----------|-----------|
| Blob → Data Lake Gen2 | **Yes** |
| Data Lake Gen2 → Blob | **No** |

**Key interview answer:** Blob for cheap backup/cold archives; ADLS Gen2 for hierarchical big-data analytics workloads.

**File-level permissions:** Both can restrict permissions to file level (ACLs more natural on Gen2 with HNS).

---

## 3. Blob types

| Type | When to use |
|------|-------------|
| **Block blob** (default) | csv, excel, audio, video, images, text; large files split into blocks (~10 GB mentioned) |
| **Page blob** | Azure VM OS/data disks, SQL DB backups |
| **Append blob** | Append-only (logs); don’t rewrite existing content |

Every uploaded file = **BLOB** (Binary Large Object).

---

## 4. Access tiers

Impact: **storage cost** vs **access cost**.

| Tier | Access pattern | Storage cost | Access cost | Rough retention idea |
|------|----------------|--------------|-------------|----------------------|
| **Hot** | Daily / frequent | Highest | Lowest | Daily access |
| **Cool** | Infrequent | Lower | Higher | ~30 days |
| **Cold** | Rare | Lower still | Higher | ~90 days |
| **Archive** | Very rare (offline) | Lowest | Highest | ~yearly |
| **Smart** | Auto-optimized (platform) | Varies | Varies | — |

Example: 1000 old movies, 10 GB, access once/year → Archive/Cool style thinking.

Lifecycle rule examples:
- Delete files older than 30 days  
- Hot → Cool after 30 days  
- After upload Hot → after 20 days Cool  

---

## 5. Soft delete, versioning, snapshot, lifecycle

| Feature | Meaning |
|---------|---------|
| **Blob soft delete** | Delete file → restore within retention (e.g. 7 days) via undelete |
| **Container soft delete** | Delete container → restore within retention (e.g. 7 days) |
| **Snapshot** | Manual backup copy of a blob |
| **Versioning** | Auto backup on every update (e.g. Customer.csv updates) |
| **Lifecycle management** | Rules to auto-tier or auto-delete based on age |

**Snapshot vs Versioning:** Snapshot = manual point-in-time copy; Versioning = automatic versions on change.

---

## 6. Security mechanisms

### For users (email)
- **RBAC / IAM** — Role Based Access Control  
  Example users: ram@tcs.com, rajesh@tcs.com  

### For applications (Netflix-style access to movies storage)
| Method | Notes |
|--------|-------|
| **Access Keys** | Full account access by default |
| **Connection Strings** | Full account access by default |
| **SAS (Shared Access Signature)** | Scoped permissions |

**SAS can restrict:**
- Permissions: Read / Write / Delete  
- Specific IP  
- Start & expiry datetime  

Default keys/connection strings → entire storage account / all containers / all permissions.

---

## 7. Fabric Lakehouse & Shortcuts

```
Azure: Storage Account → Container → sales.csv
Fabric: Lakehouse → Folder → Files → customer.csv
```

**Shortcut:** Access files in Lakehouse from external sources without full copy:
- Azure Storage Account  
- AWS  
- GCP  
- Google Drive  
- SharePoint  

Example:
```
Azure → Customer.csv
AWS   → product.csv
GCP   → sales.csv
Fabric Lakehouse → 3 shortcuts
```

**Advanced features to practice:**
- Create Blob SA  
- Create ADLS Gen2 SA  
- Create Lakehouse  
- Advanced features on both  
- Shortcuts from Lakehouse → Azure Storage  

---

## 8. Medallion Architecture

### Azure medallion (taught)

```
Super Market Business Server Data
  → Data Factory
  → Storage Account
      Bronze → Databricks (PySpark notebooks)
      Silver → Databricks
      Gold   → Databricks / Synapse Dedicated SQL Pool
  → Power BI (Reports & Dashboards)
```

### Fabric medallion (taught)

```
Super Market Business Server Data
  → Data Pipelines
  → Lakehouse
      Bronze → Notebooks
      Silver → Notebooks
      Gold   → Notebooks
  → Power BI
```

### Layer definitions

| Layer | Purpose | Typical tools | Formats |
|-------|---------|---------------|---------|
| **Bronze** | Raw business data as landed from source | ADF / Data Pipelines | Files as-is |
| **Silver** | Cleaned, deduped, null-handled, joined | Databricks / Notebooks | Parquet / Delta |
| **Gold** | Aggregated / summarized for BI | Databricks / Notebooks / DWH | Delta / Synapse tables |

**Silver example transforms:**
- Remove duplicates  
- Null handling  
- Join Customer + Product + Sales  

**Silver volume example:** 10 customers × 100 sales → 100 rows  

**Gold examples:**
- Gender-wise sales (Female 40, Male 60)  
- Product category-wise sales  
- Store-wise order counts  

**Some notes mention 4 layers** in interviews — usually Bronze/Silver/Gold (+ optional Landing/Staging or Platinum). Stick to 3-layer medallion unless asked for staging.

---

## 9. Auto Loader + Medallion (Databricks path)

```
On-prem SQL → Data Factory → Storage (files)
  → Auto Loader → Bronze (raw)
  → Silver (clean)
  → Gold (materialized views)
  → Power BI
```

---

## 10. Storage interview Q checklist (from class)

1. Which storage account used in your project?  
2. Diff Blob vs ADLS?  
3. Blob soft delete / container soft delete?  
4. Versioning vs Snapshot?  
5. Blob types?  
6. Access tiers?  
7. Storage types: Containers, File shares, Queues, Tables?  
8. Lifecycle Management?  
9. Upgrade Blob → Gen2 possible?  
10. Security: RBAC, Access keys, Connection strings, SAS?  

*(Full answers are in this file above + Interview Q&A notes.)*
