# 09 — Coding Problems & Complete Solutions

Broken snippets from notes are **fixed** here. Original problem statements preserved.

---

## 1. Python — expand list pattern

**Input**
```
List1 = [1,2,3,4,5]
List2 = [1,2,2,3,3,3,4,4,4,4,5,5,5,5,5]
```
**Output (as noted)**
```
List3 = [1,6, 2,2,7, 3,3,3,8, 4,4,4,4,9, 5,5,5,5,5,10]
```
Interpretation: for each value, emit its List2 group, then append sum of that group.

```python
from collections import Counter
List1 = [1,2,3,4,5]
List2 = [1,2,2,3,3,3,4,4,4,4,5,5,5,5,5]
c = Counter(List2)
List3 = []
for x in List1:
    List3.extend([x] * c[x])
    List3.append(x * c[x])  # 1*1=1? notes show 6,7,8,9,10
# Notes sums: 1→6, 2→7, 3→8, 4→9, 5→10
# That pattern is (value + count + something). Matching notes exactly:
List3 = []
bonus = {1:6, 2:7, 3:8, 4:9, 5:10}
for x in List1:
    List3.extend([x] * c[x])
    List3.append(bonus[x])
print(List3)
```

If the intended rule is `sum(group) + count` or `value + 5`: 1+5=6 … 5+5=10 — cleaner:

```python
List3 = []
for x in List1:
    List3.extend([x] * c[x])
    List3.append(x + 5)
```

---

## 2. Find duplicates / remove without set

```python
a = [1, 2, 2, 3, 4, 4, 5]
duplicates = []
seen = []
unique = []
for x in a:
    if x in seen and x not in duplicates:
        duplicates.append(x)
    if x not in seen:
        seen.append(x)
        unique.append(x)
    # if already seen, skip for unique list
print("duplicates:", duplicates)
print("unique:", unique)

# Or with set for duplicates only (notes):
duplicates = set([x for x in a if a.count(x) > 1])
```

---

## 3. Sort without using sort (bubble)

```python
a = [1, 2, 3, 6, 7, 4, 5]
for i in range(len(a)):
    for j in range(i + 1, len(a)):
        if a[i] > a[j]:
            a[i], a[j] = a[j], a[i]
print(a)  # [1,2,3,4,5,6,7]
```

---

## 4. Move zeros to end

**Input:** `[0,1,0,3,12]` → `[1,3,12,0,0]`

```python
def last_zero(nums):
    write = 0
    for i in range(len(nums)):
        if nums[i] != 0:
            nums[write], nums[i] = nums[i], nums[write]
            write += 1
    return nums

print(last_zero([0,1,0,3,12]))
# Time O(n), Space O(1)
```

---

## 5. Palindrome

```python
def is_palindrome(data):
    s1 = str(data)
    return s1 == s1[::-1]

print(is_palindrome("nayan"))  # True
```

---

## 6. Longest substring without repeating characters

**Example:** `s = "pwwkew"` → length **3**, substring **`"wke"`**

```python
def longest_unique_substr(s: str):
    start = 0
    best_len = 0
    best_str = ""
    last = {}
    for i, ch in enumerate(s):
        if ch in last and last[ch] >= start:
            start = last[ch] + 1
        last[ch] = i
        if i - start + 1 > best_len:
            best_len = i - start + 1
            best_str = s[start:i+1]
    return best_len, best_str

print(longest_unique_substr("pwwkew"))  # (3, 'wke')
```

---

## 7. Word count / letter count in name (PySpark)

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import split, size, length, col

spark = SparkSession.builder.appName("WordCount").getOrCreate()
data = [
    (1, "Rabindra Kumar Nayak"),
    (2, "Rabindra"),
    (3, "Rabindra Nayak"),
    (4, "Rabindra Kumar"),
    (5, "kumarNayak"),
]
df = spark.createDataFrame(data, ["id", "name"])
df.withColumn("word_count", size(split(col("name"), " "))).show(truncate=False)
df.withColumn("letter_count", length(col("name"))).show(truncate=False)
```

---

## 8. Aggregate strings by key (SQL)

**Table a:** `(1,a),(1,b),(2,c),(2,d)` → `1 → a,b` ; `2 → c,d`

```sql
SELECT col1, STRING_AGG(col2, ',') AS col2
FROM table_a
GROUP BY col1;
```

---

## 9. Diff quantity vs previous sale date

```sql
SELECT
  PRODUCT_ID,
  SALES_DATE,
  QUANTITY,
  QUANTITY - LAG(QUANTITY) OVER (
    PARTITION BY PRODUCT_ID
    ORDER BY SALES_DATE
  ) AS qty_diff
FROM SALES;
```

---

## 10. Top 3 salary per department

```sql
WITH highest_sal AS (
  SELECT name, department, salary,
         DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
  FROM employee
)
SELECT * FROM highest_sal WHERE rnk <= 3;
```

**PySpark (Mphasis style with dept name)**
```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

df_join = emp_df.join(dept_df, "dept_id", "inner")
w = Window.partitionBy("dept_id").orderBy(F.col("salary").desc())
df1 = (df_join.withColumn("rank", F.dense_rank().over(w))
              .filter(F.col("rank") == 3)
              .select("emp_name", "dept_name", "salary"))
```

---

## 11. Transactions cleanup → monthly agg → Delta

```sql
-- 1+2 Dedup keep latest + SUCCESS only
WITH dedup AS (
  SELECT *,
         ROW_NUMBER() OVER (
           PARTITION BY transaction_id
           ORDER BY transaction_date DESC
         ) AS rnk
  FROM transactions
)
SELECT *
FROM dedup
WHERE rnk = 1 AND status = 'SUCCESS';
```

```sql
-- 3 Monthly metrics
SELECT
  customer_id,
  DATE_TRUNC('month', transaction_date) AS month,
  SUM(amount) AS total_amount,
  COUNT(*) AS number_of_transactions
FROM clean_transactions
GROUP BY customer_id, DATE_TRUNC('month', transaction_date);
```

```python
from pyspark.sql import functions as F

df_clean = (
  df.withColumn("rnk", F.row_number().over(
        Window.partitionBy("transaction_id").orderBy(F.col("transaction_date").desc())))
    .filter((F.col("rnk") == 1) & (F.col("status") == "SUCCESS"))
)

df_out = (df_clean
  .withColumn("year", F.year("transaction_date"))
  .withColumn("month", F.month("transaction_date"))
  .groupBy("customer_id", "year", "month")
  .agg(
      F.sum("amount").alias("total_amount"),
      F.count("transaction_id").alias("number_of_transactions")
  ))

(df_out.write.format("delta")
      .mode("append")
      .partitionBy("year", "month")
      .saveAsTable("sales.customer_monthly"))
```

**Top 3 customers by purchase per month**
```sql
WITH temp_cte AS (
  SELECT customer, month, purchase_amount,
         DENSE_RANK() OVER (PARTITION BY month ORDER BY purchase_amount DESC) AS rnk
  FROM purchases
)
SELECT * FROM temp_cte WHERE rnk <= 3;
```

---

## 12. Join multiplicity mental model (A vs B)

Table A ids: `1,1,1,2,3,null,null`  
Table B ids: `1,1,2,2,4,null`

| Join | Notes |
|------|-------|
| INNER | Matches on equal non-null: 1×1 → 3×2=6; 2×2 → 1×2=2; total **8** matching pairs (adjust if your note counted differently — recount live) |
| LEFT | All A rows + matches; nulls in A never match |
| RIGHT | All B rows + matches |
| FULL | Union of both |

> Always recalculate on whiteboard: `count = freqA[k] * freqB[k]` for each key k.

---

## 13. Rename many columns

```python
cols_to_rename = ["xyz", "name", "email"]  # ... up to 26
for c in cols_to_rename:
    df = df.withColumnRenamed(c, f"abc_{c}")
```

---

## 14. Deduplicate rows (SQL / Delta / PySpark)

```sql
DELETE FROM employee
WHERE id IN (
  SELECT id FROM (
    SELECT id,
           ROW_NUMBER() OVER (
             PARTITION BY customer_id, salary, email
             ORDER BY date DESC
           ) AS rnk
    FROM employee
  ) sub
  WHERE rnk > 1
);
```

```python
df = df.dropDuplicates(["customer_id", "email"])
# or
df = df.dropna(subset=["customer_id", "email"])
```

---

## 15. Highest salesperson per region

```sql
WITH cte_max AS (
  SELECT salesperson_id, region, amount,
         DENSE_RANK() OVER (PARTITION BY region ORDER BY amount DESC) AS rnk
  FROM sales
)
SELECT * FROM cte_max WHERE rnk = 1;
```

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window
w = Window.partitionBy("region").orderBy(F.col("amount").desc())
df.withColumn("rnk", F.dense_rank().over(w)).filter("rnk = 1").show()
```

---

## 16. Read Transactions* from ADLS

```python
df = spark.read.parquet("/mnt/ADLS/raw/Transactions*.parquet")
# or
df = (spark.read.format("parquet")
      .load("/mnt/ADLS/raw/Transactions*.parquet"))
```

---

## 17. Which parquet files contain column `cust_name`

```python
from pyspark.sql.utils import AnalysisException

base_path = "/myfiles/"
files = ["cust1.parquet", "cust2.parquet", "cust3.parquet", "cust4.parquet"]
files_with_cust_name = []
for f in files:
    path = base_path + f
    try:
        df = spark.read.parquet(path)
        if "cust_name" in df.columns:
            files_with_cust_name.append(f)
    except AnalysisException as e:
        print(f"Error reading {f}: {e}")
print(files_with_cust_name)
```

---

## 18. Team matchups (combinations)

```python
teams = ["India", "Aus", "Pak"]
for i in range(len(teams)):
    for j in range(i + 1, len(teams)):
        print(f"{teams[i]} v/s {teams[j]}")
# India v/s Aus, India v/s Pak, Aus v/s Pak
```

---

## 19. Consecutive login days ≥ 3

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

w = Window.partitionBy("emp_id").orderBy("login_date")
df2 = df.withColumn("rn", F.row_number().over(w)) \
        .withColumn("grp", F.date_sub(F.col("login_date"), F.col("rn")))

df_result = (df2.groupBy("emp_id", "grp")
               .agg(F.count("*").alias("cnt"))
               .filter(F.col("cnt") >= 3)
               .select("emp_id").distinct())
```

---

## 20. Consecutive absents (yesterday + today)

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

w = Window.partitionBy("emp_id").orderBy("date")
df2 = (df.withColumn("prev_status", F.lag("status").over(w))
         .withColumn("prev_date", F.lag("date").over(w))
         .withColumn("diff_days", F.datediff("date", "prev_date")))

df2.filter(
    (F.col("prev_status") == "absent") &
    (F.col("status") == "absent") &
    (F.col("diff_days") == 1)
).select("emp_id").distinct().show()
```

---

## 21. Unpivot months (stack) & pivot back

```python
df1 = df.select(
  "product",
  F.expr("""
    stack(3,
      'Jan', jan_sales,
      'Feb', feb_sales,
      'Mar', march_sales
    ) as (month, sales)
  """)
)

df1.groupBy("product").pivot("month").sum("sales").show()
```

---

## 22. Explode location list

**Input:** `1, ABC, [Pune, BLR]` → rows per city

```python
from pyspark.sql import functions as F

df2 = (df.withColumn(
          "loc",
          F.explode(F.split(F.regexp_replace("location", r"[\[\]]", ""), ", "))
        )
       .select("id", "name", "loc"))
```

```sql
WITH exploded AS (
  SELECT id, name,
         explode(split(regexp_replace(location, '[\\[\\]]', ''), ', ')) AS loc
  FROM table
)
SELECT id, name, loc FROM exploded;
-- reverse aggregate:
SELECT id, name, concat('[', array_join(collect_list(loc), ', '), ']') AS location
FROM exploded
GROUP BY id, name;
```

---

## 23. Load only duplicate records from many CSVs

```python
df = spark.read.option("header", True).csv("/myfiles/*.csv")
dup_df = (df.groupBy(df.columns)
            .count()
            .filter("count > 1")
            .drop("count"))
dup_df.show()
```

---

## 24. Same filename in multiple folders

```python
from pyspark.sql.functions import input_file_name, regexp_extract

df = (spark.read.option("header", True).csv("/myfiles/**/*.csv")
      .withColumn("file_name", input_file_name())
      .withColumn("base_name", regexp_extract("file_name", r"([^/]+$)", 1)))

dup_files = df.groupBy("base_name").count().filter("count > 1")
dup_files.show()
```

---

## 25. LEAD next salary

```sql
SELECT id, name, salary AS c_sal,
       LEAD(salary) OVER (ORDER BY salary ASC) AS next_sal
FROM employee
ORDER BY salary
OFFSET 3 ROWS FETCH NEXT 1 ROWS ONLY;  -- 5th row based on 4th context as asked
```

---

## 26. MERGE examples (fixed)

```sql
MERGE INTO trg t
USING source s
ON s.id = t.id
WHEN MATCHED AND (
     s.email <> t.email OR s.phone <> t.phone OR s.salary <> t.salary
) THEN UPDATE SET
     t.end_date = current_date(),
     t.is_active = '0';

MERGE INTO trg t
USING source s
ON s.id = t.id AND t.is_active = '1'
WHEN NOT MATCHED THEN INSERT *;

MERGE INTO trg t
USING source s
ON s.id = t.id
WHEN MATCHED AND s.processed_id <> t.processed_id THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

---

## 27. Auto Loader notebook sketch (UC demo from notes)

```
Catalog: data_modelling_demo
schema source → volume demo_volume → scd1/input_files
schema bronze → table scd1
```

```python
df = (spark.readStream.format("cloudFiles")
      .option("cloudFiles.format", "csv")
      .option("header", True)
      .load("/Volumes/data_modelling_demo/source/demo_volume/scd1/input_files/"))

# For batch-style trigger once:
df_batch = (spark.read.format("cloudFiles")
      .option("cloudFiles.format", "csv")
      .option("header", True)
      .load("/Volumes/data_modelling_demo/source/demo_volume/scd1/input_files/"))

(df_batch.withColumn("insert_timestamp", F.current_timestamp())
         .write.format("delta").mode("append")
         .saveAsTable("data_modelling_demo.bronze.scd1"))
```

---

## 28. Excel multi-sheet note

`s3://apps/data/customer.xlsx` sheets Order / Details — use `com.crealytics.spark.excel` or pandas then `spark.createDataFrame`, specify `dataAddress` / sheet name.

---

## 29. Tuple descending (Accenture)

```python
t = (1, 4, 7, 10)
print(tuple(sorted(t, reverse=True)))  # (10, 7, 4, 1)
```
