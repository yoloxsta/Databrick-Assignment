# Databricks အပြည့်အစုံလေ့လာရေးလမ်းညွှန်

> **အာရုံစိုက်ထားသည့် Platform**: Azure Databricks နှင့် Databricks Free Edition  
> **သင့် Lab Environment**: Databricks Free Edition  
> **သင့် Catalog / Schema**: `workspace.default`  
> **နောက်ဆုံးပြင်ဆင်သည့်ရက်**: 2026-11-11

ဤစာတမ်းသည် Databricks ကို “ဘာလဲ၊ ဘာကြောင့်သုံးလဲ၊ ဘယ်လိုအလုပ်လုပ်လဲ” မှစ၍ Apache Spark, Delta Lake, Unity Catalog, Lakeflow, SQL, Dashboard, MLflow, Security, Performance နှင့် လက်တွေ့ Demo Project အထိ မြန်မာဘာသာဖြင့် ရှင်းပြထားသည်။ UI တွင် ရှာဖွေရလွယ်စေရန် technical terms များကို English အတိုင်း ထားထားသည်။

---

## မာတိကာ

1. [Databricks ဆိုတာဘာလဲ](#1-databricks-ဆိုတာဘာလဲ)
2. [Azure Databricks နှင့် Free Edition](#2-azure-databricks-နှင့်-free-edition)
3. [Databricks ကို ဘာကြောင့်သုံးသလဲ](#3-databricks-ကို-ဘာကြောင့်သုံးသလဲ)
4. [Lakehouse နှင့် Platform Architecture](#4-lakehouse-နှင့်-platform-architecture)
5. [Workspace Objects](#5-workspace-objects)
6. [Apache Spark](#6-apache-spark)
7. [PySpark DataFrame နှင့် Spark SQL](#7-pyspark-dataframe-နှင့်-spark-sql)
8. [Delta Lake](#8-delta-lake)
9. [Unity Catalog နှင့် Storage](#9-unity-catalog-နှင့်-storage)
10. [Compute](#10-compute)
11. [Data Ingestion နှင့် Medallion Architecture](#11-data-ingestion-နှင့်-medallion-architecture)
12. [Lakeflow, Batch နှင့် Streaming](#12-lakeflow-batch-နှင့်-streaming)
13. [Databricks SQL နှင့် Dashboard](#13-databricks-sql-နှင့်-dashboard)
14. [Machine Learning, MLflow နှင့် AI](#14-machine-learning-mlflow-နှင့်-ai)
15. [Security, Networking နှင့် Encryption](#15-security-networking-နှင့်-encryption)
16. [Performance နှင့် Cost Optimization](#16-performance-နှင့်-cost-optimization)
17. [Git, CI/CD နှင့် Production Practices](#17-git-cicd-နှင့်-production-practices)
18. [NYC Taxi Demo Project](#18-nyc-taxi-demo-project)
19. [Common Errors](#19-common-errors)
20. [Learning Roadmap နှင့် Glossary](#20-learning-roadmap-နှင့်-glossary)
21. [Official References](#21-official-references)

---

# 1. Databricks ဆိုတာဘာလဲ

Databricks သည် data engineering, analytics, business intelligence, machine learning နှင့် generative AI workload များကို platform တစ်ခုတည်းတွင် build, run, govern နှင့် share လုပ်နိုင်သည့် **Data + AI Platform** ဖြစ်သည်။ Microsoft ၏ [Azure Databricks overview](https://learn.microsoft.com/en-us/azure/databricks/introduction/index) အရ Azure Databricks သည် enterprise-scale data, analytics နှင့် AI solution များကို တည်ဆောက်၊ deploy၊ share နှင့် maintain လုပ်ရန် unified analytics platform ဖြစ်သည်။

Databricks ကို database တစ်ခုတည်းဟု မမြင်သင့်ပါ။ ၎င်းတွင် distributed compute, transactional data storage, governance catalog, notebooks, SQL engine, workflow orchestration, dashboards, ML lifecycle နှင့် AI tooling တို့ပါဝင်သည်။

```text
Sources → Ingestion → Delta Tables → Transformations → Gold Data Products
                         │                    ├─ Dashboard / BI
                         │                    ├─ ML / AI
                         │                    └─ Applications
                         └──── Unity Catalog က access, lineage, audit ကိုထိန်း
```
# 2. Azure Databricks နှင့် Free Edition

## 2.1 Azure Databricks

Azure Databricks သည် Microsoft Azure subscription နှင့် ပေါင်းစပ်ထားသော managed Databricks service ဖြစ်သည်။ Enterprise environment တွင် Microsoft Entra ID, Azure Data Lake Storage Gen2, Azure Key Vault, Azure Virtual Network, Private Link, Azure Monitor, Power BI နှင့် Azure DevOps တို့နှင့် ချိတ်ဆက်နိုင်သည်။ Azure နှင့် Databricks က control-plane/compute infrastructure အပိုင်းများကို service model အလိုက် စီမံပေးပြီး team က data product နှင့် code ကို အာရုံစိုက်နိုင်သည်။

## 2.2 Databricks Free Edition

Free Edition သည် သင်ယူခြင်း၊ personal experimentation နှင့် non-commercial prototype အတွက် no-cost environment ဖြစ်သည်။ Legacy Community Edition ကို 2025 တွင် ရပ်ဆိုင်းပြီး Free Edition ဖြင့် အစားထိုးခဲ့သည်။ အသေးစိတ်ကို [Free Edition overview](https://learn.microsoft.com/en-us/azure/databricks/getting-started/free-edition) တွင်ကြည့်နိုင်သည်။

သင့် workspace တွင် လက်ရှိသုံးမည့် namespace သည်:

```text
Catalog : workspace
Schema  : default
Table   : taxi_trips_bronze
Full name: workspace.default.taxi_trips_bronze
```

[Free Edition limitations](https://learn.microsoft.com/en-us/azure/databricks/getting-started/free-edition-limitations) အရ serverless compute သာရရှိပြီး custom compute configuration မရပါ။ SQL warehouse တစ်ခုသာရပြီး အများဆုံး 2X-Small ဖြစ်သည်။ Concurrent job tasks အများဆုံး 5 ခု၊ pipeline type တစ်မျိုးလျှင် active pipeline တစ်ခုဖြစ်သည်။ R နှင့် Scala မရ၊ outbound internet ကို trusted domains အချို့သာ ခွင့်ပြုသည်။ Account တစ်ခုတွင် workspace နှင့် metastore တစ်ခုစီသာရှိပြီး custom private networking, enterprise SSO/SCIM နှင့် guaranteed SLA မပါပါ။ Fair-usage quota ပြည့်လျှင် compute ကို reset အချိန်အထိ ယာယီရပ်နိုင်သည်။

Free Edition တွင် သင်ယူထားသော SQL, PySpark, Delta Lake, Unity Catalog namespace, Medallion architecture, Jobs, pipelines နှင့် MLflow concepts များသည် paid Azure Databricks သို့ သယ်ဆောင်အသုံးချနိုင်သည်။ Azure-specific VNet, Private Link, customer-managed keys နှင့် enterprise identity ကို paid Azure workspace တွင်သာ အပြည့် lab လုပ်နိုင်သည်။

# 3. Databricks ကို ဘာကြောင့်သုံးသလဲ

Traditional architecture တွင် data lake, warehouse, ETL tool, notebook service, ML platform နှင့် governance catalog များ သီးခြားဖြစ်တတ်သည်။ Data copy များပြားခြင်း၊ result မတူခြင်း၊ permission model များပြားခြင်း၊ lineage ရှာရခက်ခြင်းနှင့် operation cost တိုးခြင်းတို့ ဖြစ်နိုင်သည်။ Databricks lakehouse approach သည် governed data foundation တစ်ခုပေါ်တွင် workload များကို စုစည်းရန် ရည်ရွယ်သည်။

| Use case | Databricks တွင်လုပ်ဆောင်နိုင်သည့်အရာ |
|---|---|
| Data Engineering | CSV/JSON/database/event data ကို ingest, clean, transform |
| Data Warehousing | SQL model, KPI, dimensional tables, BI queries |
| Streaming | Kafka/Event Hubs/file-arrival data ကို incremental process |
| Data Science | Exploration, feature engineering, experiments |
| Machine Learning | Train, evaluate, register, deploy, monitor |
| Generative AI | RAG, agents, model evaluation, tracing, serving |
| Governance | Permissions, lineage, auditing, discovery, classification |
| Sharing | Governed data/AI assets ကို controlled access ဖြင့် share |

အဓိကအသုံးပြုသူများမှာ Data Engineer, Analytics Engineer, Data Analyst, Data Scientist, ML Engineer, Platform Engineer, Data Steward နှင့် Business User တို့ဖြစ်သည်။

# 4. Lakehouse နှင့် Platform Architecture

**Data Lake** သည် raw နှင့် format မျိုးစုံကို စျေးသက်သာစွာ သိမ်းနိုင်သော်လည်း transaction, quality နှင့် governance ကို ထပ်တည်ဆောက်ရနိုင်သည်။ **Data Warehouse** သည် structured SQL analytics အတွက်ကောင်းသော်လည်း raw/streaming/ML use case တွင် data copy သို့မဟုတ် system အသစ်လိုနိုင်သည်။ **Lakehouse** သည် open cloud storage ၏ flexibility နှင့် warehouse ၏ transaction, governance, performance တို့ကို ပေါင်းစပ်ရန် ရည်ရွယ်သည်။

[High-level architecture](https://learn.microsoft.com/en-us/azure/databricks/getting-started/high-level-architecture) တွင် Account, Workspace, Unity Catalog metastore, Control Plane, Compute Plane နှင့် Storage ကို အဓိကခွဲထားသည်။

- **Account**: users, groups, service principals, workspaces, metastores, usage နှင့် policy တို့၏ organization-level boundary ဖြစ်သည်။
- **Workspace**: notebook, SQL, job, pipeline, dashboard နှင့် experiment များလုပ်သည့် collaboration environment ဖြစ်သည်။
- **Control Plane**: Web UI, APIs, scheduler, metadata နှင့် compute lifecycle services ပါဝင်သည်။
- **Serverless Compute Plane**: Databricks က infrastructure provision, scale နှင့် optimize လုပ်ပေးသည်။ Free Edition သည် serverless-only ဖြစ်သည်။
- **Classic Compute Plane**: paid Azure deployment တွင် customer Azure subscription အတွင်း custom network/VM configuration ဖြင့် run နိုင်သည်။
- **Storage**: business data tables/volumes, workspace files နှင့် system-generated data တို့ကို သီးခြားစဉ်းစားရသည်။

```text
Users / Apps / BI → Control Plane → Serverless or Classic Compute → Data Storage
                           │                       ▲
                           └──── Unity Catalog ────┘
```
# 5. Workspace Objects

- **Notebook**: Python/SQL code, Markdown, chart နှင့် result ကို cell အလိုက်ထားသည့် interactive document ဖြစ်သည်။ `Shift+Enter` ဖြင့် cell ကို run ပြီး နောက် cell သို့သွားနိုင်သည်။
- **Workspace File**: `.py`, `.sql`, `.yml`, `.json`, `.md` စသည့် source/config files ဖြစ်သည်။ Business dataset သိမ်းရန် မသုံးသင့်ပါ။
- **SQL Query**: SQL Editor တွင် သိမ်းထားသော query ဖြစ်ပြီး SQL warehouse ပေါ်တွင် run သည်။
- **Dashboard**: governed datasets မှ visualization, KPI နှင့် filter များကို author/publish လုပ်သည်။
- **Job**: notebook, SQL, Python သို့မဟုတ် pipeline task များကို dependency ဖြင့် orchestrate လုပ်သည်။
- **Pipeline**: source မှ target datasets သို့ batch/streaming transformations နှင့် dependencies ကို ကြေညာထားသည်။
- **Experiment**: ML training runs များကို parameters, metrics, artifacts နှင့်စုထားသည်။
- **Registered Model**: version, lineage, permission နှင့် deployment reference ပါသော governed model object ဖြစ်သည်။
- **Git Folder**: Git provider နှင့်ချိတ်ထားသော source-controlled project folder ဖြစ်သည်။

Notebook တွင် language magic သုံးနိုင်သည်။

```python
# Python cell
trips_df = spark.read.table("samples.nyctaxi.trips")
display(trips_df.limit(5))
```

```sql
-- Notebook cell အစတွင် %sql ထည့်ပြီး run နိုင်သည်
SELECT * FROM samples.nyctaxi.trips LIMIT 5;
```

# 6. Apache Spark

Apache Spark သည် dataset ကြီးကို partitions ခွဲပြီး distributed/parallel process လုပ်နိုင်သော engine ဖြစ်သည်။ Databricks notebook တွင် `spark` ဟူသော `SparkSession` ကို အသင့်ပေးထားသောကြောင့် ကိုယ်တိုင် create လုပ်ရန် မလိုပါ။

```text
Notebook/Application → Driver → Execution Plan → Executors/Workers → Results
                         │                              │
                         └─ tasks ခွဲဝေ                 └─ partitions process
```

- **Driver**: SparkSession ကိုထိန်း၊ query plan တည်ဆောက်၊ tasks ခွဲဝေ၊ job ကို coordinate လုပ်သည်။
- **Executor/Worker**: partitions များပေါ်တွင် task run၊ shuffle data ဖတ်/ရေး၊ cache လုပ်သည်။ Serverless တွင် infrastructure ကို မမြင်ရသော်လည်း execution concept တူသည်။
- **Transformation**: `select`, `filter`, `withColumn`, `join`, `groupBy` ကဲ့သို့ DataFrame အသစ်၏ logic ကို သတ်မှတ်သည်။
- **Action**: `show`, `count`, `collect`, `write` ကဲ့သို့ execution ကို trigger လုပ်သည်။
- **Lazy Evaluation**: transformations များကို ချက်ချင်းမ run ဘဲ logical plan စုထားပြီး action ရောက်မှ optimizer ပြင်ဆင်၍ run သည်။
- **Job**: action တစ်ခုကြောင့် ဖြစ်လာသော execution ဖြစ်သည်။
- **Stage**: shuffle boundary အလိုက် ခွဲထားသော task group ဖြစ်သည်။
- **Task**: partition တစ်ခုအပေါ် run သည့် အနိမ့်ဆုံး execution unit ဖြစ်သည်။

```text
Action → Job → Stage 1 → Tasks per partition
                  │
                  └─ Shuffle → Stage 2 → Tasks per partition
```

**Partition** များလွန်းလျှင် scheduling/small-file overhead တက်ပြီး နည်းလွန်းလျှင် parallelism ကျနိုင်သည်။ `repartition()` သည် shuffle ဖြစ်နိုင်ပြီး partition count တိုး/လျှော့ရာတွင်သုံးသည်။ `coalesce()` သည် partition လျှော့ရာတွင် shuffle နည်းနိုင်သည်။

`groupBy`, `join`, `distinct`, `orderBy`, `repartition` များတွင် executor များကြား network မှ data ပြန်ဝေရသော **shuffle** ဖြစ်တတ်သည်။ Shuffle သည် memory, disk နှင့် network cost မြင့်နိုင်သည်။

```python
print(spark.version)
print(trips_df.rdd.getNumPartitions())
balanced_df = trips_df.repartition(8)
```

`collect()` နှင့် `toPandas()` သည် data အားလုံးကို driver memory သို့ ဆွဲယူနိုင်သည်။ Dataset ကြီးတွင် မသုံးမီ `limit`, aggregate သို့မဟုတ် sample လုပ်ပါ။

```python
small_pdf = trips_df.limit(1000).toPandas()
```
# 7. PySpark DataFrame နှင့် Spark SQL

DataFrame သည် named columns နှင့် schema ပါသော distributed tabular data abstraction ဖြစ်သည်။ Spreadsheet သို့မဟုတ် SQL table နှင့်ဆင်တူသော်လည်း distributed execution အတွက် ဒီဇိုင်းလုပ်ထားသည်။ Production ingestion တွင် `inferSchema` သာအားကိုးခြင်းထက် explicit schema သတ်မှတ်ခြင်းက ပိုယုံကြည်ရသည်။

```python
from pyspark.sql import functions as F

data = [(1, "Yangon", 1200.0), (2, "Mandalay", 950.0), (3, "Yangon", 800.0)]
sales_df = spark.createDataFrame(data, "id INT, city STRING, amount DOUBLE")
sales_df.printSchema()

clean_df = (
    sales_df
    .filter(F.col("amount") > 0)
    .withColumn("tax", F.round(F.col("amount") * 0.05, 2))
    .select("id", "city", "amount", "tax")
)

summary_df = clean_df.groupBy("city").agg(
    F.count("*").alias("sale_count"),
    F.sum("amount").alias("total_amount"),
    F.avg("amount").alias("average_amount")
)
display(summary_df)
```

အသုံးများသော DataFrame operations များမှာ `select`, `filter/where`, `withColumn`, `drop`, `fillna`, `dropDuplicates`, `groupBy`, `agg`, `join`, `orderBy`, `unionByName` နှင့် window functions တို့ဖြစ်သည်။ Join types များတွင် `inner`, `left`, `right`, `full`, `left_semi`, `left_anti` ပါဝင်သည်။

Spark SQL နှင့် DataFrame API နှစ်ခုလုံးသည် Spark optimizer ကို အသုံးပြုသောကြောင့် team skill, readability နှင့် reuse လိုအပ်ချက်အရ ရွေးနိုင်သည်။

```sql
SELECT pickup_zip,
       COUNT(*) AS total_trips,
       AVG(fare_amount) AS average_fare
FROM samples.nyctaxi.trips
WHERE trip_distance > 0 AND fare_amount > 0
GROUP BY pickup_zip
ORDER BY total_trips DESC
LIMIT 10;
```

DataFrame ကို temporary view လုပ်၍ SQL နှင့်ပေါင်းသုံးနိုင်သည်။ Temp view သည် current Spark session အတွင်းသာရှိသည်။

```python
clean_df.createOrReplaceTempView("sales_temp")
result_df = spark.sql("SELECT city, SUM(amount) total FROM sales_temp GROUP BY city")
result_df.explain("formatted")
```

# 8. Delta Lake

[Delta Lake documentation](https://learn.microsoft.com/en-us/azure/databricks/delta/) အရ Delta Lake သည် Parquet data files များကို file-based transaction log ဖြင့် ဖြည့်စွက်ထားသော optimized lakehouse storage layer ဖြစ်သည်။ Databricks table များ၏ default format သည် ယေဘုယျအားဖြင့် Delta ဖြစ်ပြီး batch နှင့် streaming နှစ်မျိုးလုံးနှင့် အဆင်ပြေသည်။

```text
Delta table directory
├── part-....parquet       ← actual data
└── _delta_log/            ← commits, metadata, table versions
```

Data files သို့မဟုတ် `_delta_log` ကို manual edit/delete မလုပ်သင့်ပါ။ Delta commands နှင့် APIs ကိုသာ သုံးရမည်။
## 8.1 ACID Transactions

- **Atomicity**: Transaction အားလုံးအောင်မြင် သို့မဟုတ် အားလုံး rollback ဖြစ်သည်။
- **Consistency**: Schema နှင့် constraints များကို မဖျက်စီးစေပါ။
- **Isolation**: Concurrent readers/writers များက consistent table state ကို မြင်သည်။
- **Durability**: Commit ဖြစ်ပြီးသော state ကို transaction log ဖြင့် ထိန်းသည်။

## 8.2 Create, Append, Update, Delete, Merge

```sql
CREATE TABLE workspace.default.city_sales (
  id INT, city STRING, amount DOUBLE, sale_date DATE
) USING DELTA;

INSERT INTO workspace.default.city_sales VALUES
  (1, 'Yangon', 1200.0, DATE '2026-11-11');

UPDATE workspace.default.city_sales SET amount = amount * 1.05 WHERE city = 'Yangon';
DELETE FROM workspace.default.city_sales WHERE amount < 0;
```

Upsert အတွက် `MERGE` သုံးသည်။ Stable business key နှင့် duplicate-source handling မရှိလျှင် incorrect result ဖြစ်နိုင်သောကြောင့် source ကို အရင် deduplicate လုပ်ပါ။

```sql
MERGE INTO workspace.default.city_sales AS target
USING workspace.default.city_sales_updates AS source
ON target.id = source.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

## 8.3 History, Time Travel နှင့် Schema

```sql
DESCRIBE HISTORY workspace.default.city_sales;
SELECT * FROM workspace.default.city_sales VERSION AS OF 0;
```

Time travel ရနိုင်သည့်ကာလသည် retention settings နှင့် `VACUUM` အပေါ်မူတည်သည်။ **Schema enforcement** က incompatible write ကိုတားပြီး **schema evolution** က controlled ပုံစံဖြင့် column အသစ်များကို လက်ခံနိုင်စေသည်။ Auto evolution ကို data contract နှင့် review မရှိဘဲ အမြဲဖွင့်မထားသင့်ပါ။

## 8.4 Performance Features

- `OPTIMIZE` က small files များကို compact လုပ်နိုင်သည်။
- Data skipping က column statistics ဖြင့် မလိုအပ်သော files မဖတ်အောင် လျှော့သည်။
- Liquid clustering သည် changing query patterns အတွက် flexible data layout ပေးနိုင်သည်။
- Change Data Feed က table versions ကြား row-level changes ကို downstream incremental processing အတွက်ဖတ်နိုင်စေသည်။
- `VACUUM` က unreferenced old files ဖယ်သဖြင့် storage လျော့သော်လည်း old time-travel versions ကို ဖတ်မရနိုင်တော့စေသည်။ Retention ကို မသိဘဲ အလွန်တိုမလုပ်ပါနှင့်။

```sql
OPTIMIZE workspace.default.city_sales;
DESCRIBE DETAIL workspace.default.city_sales;
```

# 9. Unity Catalog နှင့် Storage

[Unity Catalog overview](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/) အရ Unity Catalog သည် data နှင့် AI assets များအတွက် access control, lineage, auditing, discovery နှင့် governance ကို unified layer တစ်ခုဖြင့် ထိန်းသည်။

```text
Metastore
└── Catalog: workspace
    └── Schema: default
        ├── Table: taxi_trips_bronze
        ├── View / Materialized View
        ├── Volume
        ├── Function
        └── Registered Model
```

Three-level namespace သည် `catalog.schema.object` ဖြစ်သည်။ Catalog နှင့် schema နှစ်ခုလုံးကို qualify လုပ်ခြင်းက notebook default context ပြောင်းသွားသော်လည်း မှန်ကန်သော object ကို ရည်ညွှန်းစေသည်။
## 9.1 Managed နှင့် External Assets

- **Managed table/volume**: Unity Catalog က governance နှင့် storage lifecycle နှစ်ခုလုံးကို စီမံသည်။ New learning workload အတွက် စတင်ရန်လွယ်သည်။
- **External table/volume**: Data files သည် user သတ်မှတ်ထားသော cloud location တွင်ရှိပြီး Unity Catalog က governance လုပ်သည်။ Storage lifecycle ကို organization က ထိန်းသည်။

## 9.2 Volumes

Volume သည် CSV, JSON, images, documents ကဲ့သို့ non-tabular files အတွက် governed storage ဖြစ်သည်။

```sql
CREATE VOLUME IF NOT EXISTS workspace.default.raw_files;
```

```text
/Volumes/workspace/default/raw_files/example.csv
```

## 9.3 Permissions

အသုံးများသော privileges များမှာ `USE CATALOG`, `USE SCHEMA`, `SELECT`, `MODIFY`, `CREATE TABLE`, `CREATE VOLUME` နှင့် `EXECUTE` တို့ဖြစ်သည်။ Enterprise တွင် user တစ်ဦးချင်းထက် group သို့ least-privilege grant လုပ်ခြင်းက ပိုကောင်းသည်။

```sql
GRANT USE CATALOG ON CATALOG workspace TO `data_analysts`;
GRANT USE SCHEMA ON SCHEMA workspace.default TO `data_analysts`;
GRANT SELECT ON TABLE workspace.default.gold_daily_metrics TO `data_analysts`;
```

Free Edition တွင် enterprise group/admin feature အချို့မရှိနိုင်သဖြင့် UI တွင်ရရှိသော permission options ကိုသာ သုံးပါ။ Catalog Explorer မှ owner, columns, comments, sample data, permissions နှင့် lineage ကို ကြည့်နိုင်သည်။

## 9.4 Storage ရွေးချယ်မှု

| Artifact | သင့်တော်သောနေရာ |
|---|---|
| Structured business data | Unity Catalog table |
| Raw CSV/JSON/images | Unity Catalog volume |
| Notebook/source/config | Workspace သို့မဟုတ် Git folder |
| Session-only query result | Temp view/cache |
| Governed ML model | Unity Catalog registered model |

DBFS root နှင့် DBFS mounts သည် legacy pattern ဖြစ်သည်။ New design တွင် Unity Catalog tables, volumes နှင့် external locations ကို ဦးစားပေးပါ။

# 10. Compute

[Compute overview](https://learn.microsoft.com/en-us/azure/databricks/compute/) အရ workload အလိုက် serverless compute, classic compute နှင့် SQL warehouses ကို အသုံးပြုနိုင်သည်။

- **Serverless**: Databricks က provisioning, scaling, runtime နှင့် optimization ကို စီမံသည်။ Startup မြန်ပြီး Free Edition တွင် ဤပုံစံသာ ရသည်။
- **Classic compute**: Paid Azure workspace တွင် VM type, runtime, libraries, network နှင့် autoscaling ကို ပိုထိန်းချုပ်နိုင်သည်။
- **SQL warehouse**: SQL Editor, BI နှင့် dashboard queries အတွက် optimized compute ဖြစ်သည်။ Free Edition တွင် warehouse တစ်ခု၊ အများဆုံး 2X-Small ဖြစ်သည်။
- **Interactive compute**: Development/exploration အတွက် long-lived compute concept ဖြစ်သည်။
- **Job compute/serverless jobs**: Automated task run အတွက် allocate လုပ်ပြီး workload ပြီးသည့်အခါ release လုပ်ရန် ရည်ရွယ်သည်။

| Workload | ဦးစားပေးရွေးချယ်မှု |
|---|---|
| Free Edition notebook | Serverless notebook compute |
| SQL/dashboard | SQL warehouse |
| Scheduled ETL | Serverless Jobs သို့မဟုတ် job compute |
| Custom VNet/VM/library need | Paid workspace classic compute |

Free Edition UI တွင် cluster VM settings မပေါ်ခြင်းသည် error မဟုတ်ပါ။ Serverless-only design ကြောင့်ဖြစ်သည်။
# 11. Data Ingestion နှင့် Medallion Architecture

Data ingestion သည် source system မှ lakehouse သို့ data ယူဆောင်ခြင်းဖြစ်သည်။ Sources များတွင် Azure Data Lake Storage, Blob Storage, databases, Kafka/Event Hubs, SaaS applications, REST APIs နှင့် CSV/JSON/Parquet files ပါဝင်နိုင်သည်။

## 11.1 Ingestion နည်းလမ်းများ

- **UI File Upload**: Free Edition တွင် local file ကို Volume/table သို့တင်ရန် အလွယ်ဆုံးနည်းဖြစ်သည်။
- **`COPY INTO`**: Files များကို SQL ဖြင့် incremental/idempotent load လုပ်ရန်သုံးသည်။
- **Auto Loader**: Cloud directory ထဲသို့ အသစ်ရောက်လာသော files ကို `cloudFiles` Structured Streaming source ဖြင့် incremental process လုပ်သည်။ [Auto Loader overview](https://learn.microsoft.com/en-us/azure/databricks/ingestion/cloud-object-storage/auto-loader/) ကိုကြည့်ပါ။
- **Lakeflow Connect**: Supported sources အတွက် managed connector နှင့် ingestion ပေးသည်။ Availability သည် edition/region/source ပေါ်မူတည်သည်။
- **Database/JDBC/Federation**: Database ကို copy ingest သို့မဟုတ် federated query pattern ဖြင့် သုံးနိုင်သည်။

```sql
COPY INTO workspace.default.bronze_orders
FROM '/Volumes/workspace/default/raw_files/orders/'
FILEFORMAT = CSV
FORMAT_OPTIONS ('header' = 'true');
```

Free Edition တွင် arbitrary outbound internet ကို ကန့်သတ်ထားသောကြောင့် notebook မှ URL download ပေါ်မမှီဘဲ UI upload သို့မဟုတ် included sample data ကို သုံးပါ။

## 11.2 Medallion Architecture

[Medallion architecture](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/medallion) သည် Bronze → Silver → Gold အလွှာများဖြင့် data quality နှင့် structure ကို တဖြည်းဖြည်းတိုးတက်စေသော recommended pattern ဖြစ်သည်။ မဖြစ်မနေစည်းမျဉ်းမဟုတ်ဘဲ project complexity အလိုက် ပြင်ဆင်နိုင်သည်။

### Bronze: Raw and Traceable

- Source state ကို အများဆုံးထိန်းထားသည်
- Append/incremental ingestion လုပ်သည်
- Source filename, ingestion time, source system စသည့် metadata ထည့်သည်
- Audit နှင့် reprocessing အတွက် history ထားသည်
- Heavy business logic မထည့်သေးပါ

### Silver: Validated and Reusable

- Type casting, null handling, deduplication လုပ်သည်
- Invalid records ကို drop မလုပ်မီ quarantine/reason သိမ်းနိုင်သည်
- Time zone, units နှင့် codes ကို standardize လုပ်သည်
- Late-arriving/out-of-order data ကို စီမံသည်
- Reference data နှင့် join/enrich လုပ်သည်
- Record-level detail ကို အနည်းဆုံးတစ်မျိုး ထိန်းထားသည်

### Gold: Business-Ready

- Business definition နှင့်ကိုက်သော KPI, dimension, fact နှင့် aggregate တည်ဆောက်သည်
- Dashboard, reporting, ML သို့မဟုတ် application use case အလိုက် optimize လုပ်သည်
- Frequently repeated calculation ကို governed table/materialized view အဖြစ် centralize လုပ်သည်

```text
Raw Sources → Bronze(raw) → Silver(clean/detail) → Gold(KPI/domain model)
                    │               │                    ├─ BI
                    │               │                    ├─ ML
                    └─ replay       └─ reusable          └─ Apps
```

Learning lab တွင် `workspace.default.taxi_trips_bronze` ကဲ့သို့ table prefix သုံးနိုင်သည်။ Enterprise တွင် `operations.bronze.taxi_trips`, `operations.silver.taxi_trips`, `operations.gold.daily_metrics` ကဲ့သို့ schema အလိုက် ခွဲနိုင်သည်။
# 12. Lakeflow, Batch နှင့် Streaming

## 12.1 Lakeflow Product Family

- **Lakeflow Connect**: Source systems မှ managed ingestion လုပ်သည်။
- **Lakeflow Spark Declarative Pipelines**: SQL/Python ဖြင့် batch နှင့် streaming data pipelines ကို declarative ပုံစံတည်ဆောက်သည်။ ယခင် documentation များတွင် Delta Live Tables (DLT) ဟု တွေ့နိုင်သည်။ Current [pipeline concepts](https://learn.microsoft.com/en-us/azure/databricks/ldp/concepts) တွင် pipeline, flow, streaming table, materialized view နှင့် sink တို့ကို အဓိကထားသည်။
- **Lakeflow Jobs**: Tasks, dependencies, schedules, parameters, retries နှင့် notifications ကို orchestrate လုပ်သည်။ [Lakeflow Jobs overview](https://learn.microsoft.com/en-us/azure/databricks/workflows/jobs/jobs/) ကိုကြည့်ပါ။

Declarative pipeline ဆိုသည်မှာ low-level execution loop ထက် output dataset နှင့် query dependency ကို ကြေညာခြင်းဖြစ်သည်။ Platform က dependency order, incremental execution, infrastructure နှင့် recovery အပိုင်းအချို့ကို စီမံနိုင်သည်။

```text
Bronze_Ingest
      │
      ▼
Silver_Validate
      │
      ├──────────────┐
      ▼              ▼
Gold_Daily      Gold_Locations
      │              │
      └──────┬───────┘
             ▼
        Quality_Check
```

Job တစ်ခုတွင် task dependency, parameters, trigger/schedule, retries, timeout, notifications, run history နှင့် repair run ပါနိုင်သည်။ Free Edition တွင် concurrent task နှင့် active pipeline quotas ရှိကြောင်း သတိပြုပါ။

## 12.2 Batch Processing

Batch သည် data ကို interval တစ်ခုအလိုက် စုပြီး run သည်။ Daily report, monthly close နှင့် bulk backfill အတွက် သင့်တော်သည်။ Design ရိုးရှင်းပြီး cost ထိန်းရလွယ်သော်လည်း result latency ပိုမြင့်သည်။

## 12.3 Structured Streaming

Streaming သည် events/files အသစ်ရောက်လာသည်နှင့် incremental process လုပ်သည်။ Structured Streaming သည် DataFrame/SQL semantics ကို အသုံးပြု၍ state, checkpoint နှင့် fault recovery ကိုထောက်ပံ့သည်။

```python
stream_df = spark.readStream.table("workspace.default.bronze_events")
query = (
    stream_df.writeStream
    .option("checkpointLocation", "/Volumes/workspace/default/checkpoints/events")
    .toTable("workspace.default.silver_events")
)
```

- **Checkpoint**: ဘယ် offset/version အထိ process ပြီးသည်နှင့် state ကိုမှတ်တမ်းတင်သည်။ Query မတူသည့်အခါ checkpoint path မတော်တဆ reuse မလုပ်ပါနှင့်။
- **Watermark**: Late-arriving events ကို ဘယ်အထိ လက်ခံမည်နှင့် state ဘယ်လောက်ကြာထိထားမည်ကို ထိန်းသည်။
- **Trigger**: Available-now/processing-time/continuous စသည့် execution timing ကို သတ်မှတ်သည်။ Supported mode ကို workspace/runtime ဖြင့် စစ်ပါ။
- **Stateful operation**: aggregation, deduplication နှင့် stream-stream join များသည် state/storage cost လိုနိုင်သည်။

## 12.4 Idempotency

Pipeline တူကို ပြန် run လုပ်သော်လည်း duplicate/corrupt result မဖြစ်အောင် stable business key, Delta `MERGE`, checkpoint, processed-file tracking, controlled overwrite နှင့် source deduplication ကို သုံးရသည်။
# 13. Databricks SQL နှင့် Dashboard

Databricks SQL တွင် SQL Editor, SQL warehouse, query history, alerts နှင့် AI/BI dashboards ပါဝင်သည်။ SQL warehouse သည် interactive analytics နှင့် BI concurrency အတွက် optimized compute ဖြစ်သည်။ [Photon](https://learn.microsoft.com/en-us/azure/databricks/runtime/photon) သည် SQL နှင့် DataFrame operations များကို အရှိန်မြှင့်ရန် vectorized query engine ဖြစ်သည်။

`samples.nyctaxi.trips` ကိုသုံးသော safe dashboard dataset ဥပမာ:

```sql
SELECT
  DATE(tpep_pickup_datetime) AS trip_date,
  COUNT(*) AS total_trips,
  AVG(fare_amount) AS average_fare,
  AVG(trip_distance) AS average_distance
FROM samples.nyctaxi.trips
WHERE trip_distance > 0
  AND trip_distance < 100
  AND fare_amount > 0
GROUP BY DATE(tpep_pickup_datetime)
ORDER BY trip_date;
```

Official [AI/BI dashboard tutorial](https://learn.microsoft.com/en-us/azure/databricks/dashboards/tutorials/create-dashboard) သည် NYC taxi sample ဖြင့် bar chart, scatterplot, date filter နှင့် zip filters တည်ဆောက်ပုံကို ပြထားသည်။ Dashboard authoring အတွက်:

1. **Dashboards → Create Dashboard** ကိုရွေးပါ။
2. **Data → Add SQL dataset** ဖြင့် query ထည့်ပြီး run ပါ။
3. Canvas တွင် visualization ထည့်၍ X/Y fields နှင့် aggregation ရွေးပါ။
4. Date range, pickup zip, dropoff zip filters ထည့်ပါ။
5. Result နှင့် units ကို verify ပြီးမှ publish လုပ်ပါ။
6. Sharing mode နှင့် data permissions ကို မျှဝေမည့် audience အလိုက် စစ်ပါ။

Dashboard design တွင် KPI definition တစ်ခုကို centralize လုပ်ခြင်း၊ chart တစ်ခုတွင် message တစ်ခုသာ အဓိကထားခြင်း၊ units/date range ထည့်ခြင်း၊ color consistency ထားခြင်းနှင့် draft ကို verify ပြီးမှ publish လုပ်ခြင်းတို့ကို လိုက်နာပါ။

# 14. Machine Learning, MLflow နှင့် AI

[MLflow on Databricks](https://learn.microsoft.com/en-us/azure/databricks/mlflow/) သည် traditional ML, deep learning နှင့် GenAI lifecycle အတွက် experiment tracking, evaluation, model management နှင့် observability ကို ပံ့ပိုးသည်။

```text
Problem → Prepare Data → Features → Train → Evaluate → Register → Deploy → Monitor
```

| MLflow term | အဓိပ္ပါယ် |
|---|---|
| Experiment | Related runs များ၏ container |
| Run | Training/evaluation တစ်ကြိမ် |
| Parameter | `max_depth`, `learning_rate` ကဲ့သို့ input setting |
| Metric | accuracy, F1, RMSE ကဲ့သို့ result |
| Artifact | model, plot, report, file |
| Registered model | Governed model နှင့် versions |

```python
import mlflow
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

mlflow.set_experiment("/Shared/fare_classification")
with mlflow.start_run():
    model = RandomForestClassifier(n_estimators=100, max_depth=8, random_state=42)
    model.fit(X_train, y_train)
    accuracy = accuracy_score(y_test, model.predict(X_test))
    mlflow.log_params({"n_estimators": 100, "max_depth": 8})
    mlflow.log_metric("accuracy", accuracy)
    mlflow.sklearn.log_model(model, name="model")
```
MLflow API version အလိုက် `name` သို့မဟုတ် older `artifact_path` parameter ကွာနိုင်သဖြင့် notebook ၏ installed MLflow version/documentation ကိုစစ်ပါ။ Current practice တွင် governed sharing အတွက် Unity Catalog registered models ကို ဦးစားပေးသည်။

Classification အတွက် accuracy တစ်ခုတည်းမကြည့်ဘဲ precision, recall, F1, ROC-AUC, confusion matrix နှင့် class imbalance ကို စစ်ပါ။ Regression အတွက် MAE, RMSE, R² နှင့် residual analysis ကိုကြည့်ပါ။ Prediction time တွင် မရနိုင်သေးသော actual result column ကို feature အဖြစ်ထည့်ခြင်းသည် **data leakage** ဖြစ်သည်။

Databricks AI capabilities များတွင် edition/region ပေါ်မူတည်၍ Foundation Models, AI Playground, Model Serving, AI Search/Vector Search, RAG, agents, MLflow tracing/evaluation, AI Gateway နှင့် Genie Code ပါဝင်နိုင်သည်။

```text
Documents → Parse/Chunk → Embeddings → Vector Index
                                      │
Question → Retrieve relevant context ─┴→ LLM → Answer + citations
```

RAG solution တွင် document permissions, chunk quality, retrieval precision, prompt safety, hallucination evaluation, latency, token cost နှင့် tracing ကို တပြိုင်နက်စဉ်းစားရသည်။ Free Edition endpoint/model quotas နှင့် unsupported features ကို limitations page တွင် အမြဲပြန်စစ်ပါ။

# 15. Security, Networking နှင့် Encryption

Azure Databricks security ကို identity, authorization, network, data protection, secrets, audit နှင့် compliance အလွှာများဖြင့် စဉ်းစားပါ။ [Security overview](https://learn.microsoft.com/en-us/azure/databricks/security/) ကို reference အဖြစ်သုံးနိုင်သည်။

## 15.1 Identity နှင့် Authorization

- Paid Azure workspace တွင် Microsoft Entra ID users, groups နှင့် service principals ကို သုံးနိုင်သည်။
- Unity Catalog permissions နှင့် workspace object ACLs သည် မတူသော control layers ဖြစ်နိုင်သည်။ Table ဖတ်ခွင့်ရှိသော်လည်း job edit ခွင့်ရှိမည်ဟု မယူဆသင့်ပါ။
- Human user အစား automated production jobs ကို service principal/managed identity ဖြင့် run လုပ်ခြင်းက ownership နှင့် offboarding risk လျှော့နိုင်သည်။
- Least privilege, group-based grants နှင့် regular access review ကို လိုက်နာပါ။

## 15.2 Networking

- **Secure Cluster Connectivity / No Public IP**: Classic compute node များပေါ် public IP မလိုစေဘဲ control plane သို့ outbound secure connection အသုံးပြုသည်။
- **VNet injection**: Classic compute ကို customer-managed Azure VNet subnets အတွင်း deploy လုပ်နိုင်သည်။
- **Private Link**: User-to-workspace သို့မဟုတ် compute-to-control/storage traffic ကို private endpoints ဖြင့် ထိန်းနိုင်သည်။
- **Firewall/IP access lists**: Public endpoint အသုံးပြုလျှင် allowed source networks ကို ကန့်သတ်နိုင်သည်။
- **Egress control**: Data exfiltration လျှော့ရန် destination, DNS, firewall နှင့် routing ကို design လုပ်ရသည်။

Free Edition တွင် custom private networking နှင့် security customization မရသဖြင့် ထိုအပိုင်းများကို theory အဖြစ်သာ လေ့လာနိုင်သည်။

## 15.3 Encryption နှင့် Secrets

- Data in transit ကို TLS/encrypted channels ဖြင့်ကာကွယ်သည်။
- Data at rest သည် platform/cloud-managed encryption ကို default အသုံးပြုနိုင်ပြီး paid supported deployment တွင် customer-managed key options ရှိနိုင်သည်။
- Password, token, connection string နှင့် key ကို notebook code, Git, table သို့မဟုတ် log ထဲ hard-code မလုပ်ပါနှင့်။ Secret scope, Key Vault, managed identity သို့မဟုတ် OAuth သုံးပါ။
- Secret ကို output/exception/log တွင် print မဖြစ်စေရန် စစ်ပါ။

## 15.4 Audit နှင့် Data Protection

Unity Catalog lineage, audit logs/system tables, row filters, column masks, tags/classification နှင့် workspace bindings တို့ကို requirement နှင့် edition support အလိုက်သုံးနိုင်သည်။ PII data အတွက် retention, masking, purpose limitation နှင့် deletion policy ကို architecture အစကတည်းက ထည့်သွင်းပါ။
# 16. Performance နှင့် Cost Optimization

Performance ကို cluster ကြီးအောင်လုပ်ခြင်းတစ်ခုတည်းဖြင့် မဖြေရှင်းသင့်ပါ။ Query plan, data layout, file sizes, statistics, shuffle, skew နှင့် concurrency ကို အရင်စစ်ပါ။

## 16.1 Performance Checklist

1. Query အစပိုင်းတွင် မလိုသော rows ကို filter လုပ်ပါ။
2. လိုသော columns သာ select လုပ်ပါ; production query တွင် `SELECT *` ကို အလွန်မသုံးပါနှင့်။
3. Join keys ၏ type နှင့် null/duplicate distribution ကိုစစ်ပါ။
4. Small dimension table အတွက် optimizer/broadcast strategy ကို စစ်သော်လည်း blind hint မပေးပါနှင့်။
5. Data skew ရှိသည့် key ကို Spark UI/query profile ဖြင့်ရှာပါ။
6. Repeated scans ဖြစ်မှ cache လုပ်ပြီး ပြီးသည့်အခါ unpersist လုပ်ပါ။
7. Delta small files ကို `OPTIMIZE` သို့မဟုတ် managed optimization ဖြင့်ထိန်းပါ။
8. Frequently filtered columns အတွက် statistics/data layout ကိုစစ်ပါ။
9. Python UDF ထက် built-in Spark SQL functions ကို ဦးစားပေးပါ။
10. `collect()`/`toPandas()` ကို bounded data ပေါ်တွင်သာသုံးပါ။

```python
# Built-in expression က optimizer နှင့်အလုပ်လုပ်နိုင်သည်
optimized_df = trips_df.filter("fare_amount > 0").select(
    "pickup_zip", "fare_amount", "trip_distance"
)
optimized_df.explain("formatted")
```

## 16.2 Cost Drivers

- Compute type, size နှင့် run duration
- Serverless/SQL warehouse usage
- Streaming 24/7 နှင့် triggered processing ကွာခြားမှု
- Storage amount နှင့် retention
- Model serving/AI endpoint usage
- Data transfer/egress
- Inefficient repeated scans နှင့် failed retries

[Cost optimization guidance](https://learn.microsoft.com/en-us/azure/databricks/lakehouse-architecture/cost-optimization/best-practices) နှင့် paid account ၏ billing system tables ကို အသုံးပြုနိုင်သည်။ Free Edition တွင် monetary bill ထက် fair-usage quota သည် အဓိကဖြစ်သောကြောင့် sample size လျှော့ခြင်း၊ unnecessary reruns မလုပ်ခြင်းနှင့် heavy `count()` repeated actions မလုပ်ခြင်းက အရေးကြီးသည်။

# 17. Git, CI/CD နှင့် Production Practices

Notebook ကို တစ်ဦးတည်း ad-hoc run လုပ်ခြင်းမှ production solution သို့ပြောင်းရာတွင် source control, reusable code, testing, environment separation, deployment automation နှင့် monitoring လိုသည်။

- Git folder/repository ဖြင့် branch, pull request နှင့် review သုံးပါ။
- Notebook ကို orchestration/presentation အတွက်သုံးပြီး reusable logic ကို Python modules/functions ထဲခွဲပါ။
- Hard-coded catalog/path မသုံးဘဲ dev/test/prod parameter သတ်မှတ်ပါ။
- Table contract, primary business key, expected freshness နှင့် owner ကို document လုပ်ပါ။
- Job retries မထည့်မီ operation ကို idempotent ဖြစ်စေပါ။
- Production run owner ကို personal user အပေါ်မမှီစေပါနှင့်။

[Declarative Automation Bundles](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/bundles/) သည် ယခင် Databricks Asset Bundles အမည်ဖြင့် သိရှိခဲ့သော source-controlled deployment format ဖြစ်ပြီး jobs, pipelines နှင့် related resources ကို validate/deploy/run လုပ်နိုင်သည်။

```text
Developer branch → Pull request → Automated checks → Deploy dev → Validate
                                                   → Deploy prod with approval
```

Tests ကို layer ခွဲနိုင်သည်: pure Python unit tests, Spark transformation tests, data quality checks, integration tests နှင့် post-deployment smoke tests။ Production data ကို test fixture အဖြစ် မကူးမီ privacy နှင့် size ကို စစ်ပါ။
# 18. NYC Taxi Demo Project

## 18.1 အရေးကြီးသော Schema Correction

သင့် source `samples.nyctaxi.trips` ၏ official tutorial အသုံးပြုသည့် columns များမှာ `tpep_pickup_datetime`, `tpep_dropoff_datetime`, `trip_distance`, `fare_amount`, `pickup_zip`, `dropoff_zip` တို့ဖြစ်သည်။ **`tip_amount`, `passenger_count`, `high_tip_flag` မပါပါ**။ ထို့ကြောင့် ထို columns ကိုသုံးသော “tip prediction” code သည် ဒီ sample table နှင့် run မည်မဟုတ်ပါ။ ဤ guide တွင် dashboard ကို fare/distance analysis အဖြစ်၊ ML demo ကို **high-fare classification** အဖြစ် မှန်ကန်အောင် ပြောင်းထားသည်။

သင့်လက်ရှိအခြေအနေ:

- `NYC_Taxi_Project/01_Bronze` မှ `05_Dashboards` အထိ folders တည်ဆောက်ပြီးပြီ
- `Bronze_Raw_Taxi_Data` notebook တည်ဆောက်ပြီးပြီ
- Configuration cell မှ `workspace.default.taxi_trips_bronze` ကို အောင်မြင်စွာပြပြီးပြီ

## 18.2 ယခုချက်ချင်းလုပ်ရမည့် Cell 2

Bronze notebook ရှိ cell အသစ်တွင် အောက်ပါ code ကိုထည့်ပြီး `Shift+Enter` နှိပ်ပါ။

```python
# CELL 2: Source schema နှင့် sample rows စစ်ခြင်း
raw_trips_df = spark.read.table("samples.nyctaxi.trips")

print("Source table: samples.nyctaxi.trips")
print(f"Columns: {raw_trips_df.columns}")
raw_trips_df.printSchema()
display(raw_trips_df.limit(5))
```

Expected columns ခြောက်ခုနှင့် sample rows ကို မြင်ရမည်။ Exact row count ကို သိလိုမှသာ `count()` လုပ်ပါ။ Free quota ချွေတာရန် development cell တိုင်းတွင် full count ပြန်မခေါ်ပါနှင့်။

## 18.3 Bronze Table တည်ဆောက်ခြင်း

Cell 3:

```python
from pyspark.sql import functions as F

bronze_df = (
    raw_trips_df
    .withColumn("ingested_at", F.current_timestamp())
    .withColumn("source_table", F.lit("samples.nyctaxi.trips"))
)

display(bronze_df.limit(5))
```

Cell 4:

```python
bronze_df.write.mode("overwrite").saveAsTable(
    "workspace.default.taxi_trips_bronze"
)

spark.sql("DESCRIBE TABLE workspace.default.taxi_trips_bronze").show(truncate=False)
```

Validation:

```python
check_df = spark.read.table("workspace.default.taxi_trips_bronze")
assert "ingested_at" in check_df.columns
assert "source_table" in check_df.columns
print("Bronze table validation passed")
```

Catalog Explorer → `workspace` → `default` တွင် `taxi_trips_bronze` ပေါ်လာလျှင် Bronze step ပြီးပါပြီ။
## 18.4 Silver Notebook

`02_Silver/Silver_Cleansed_Taxi_Data` notebook တည်ဆောက်ပြီး အောက်ပါ cells ကို run ပါ။ Silver layer တွင် invalid data ဖယ်၊ derived columns ထည့်ပြီး quality status ကိုစစ်မည်။

```python
from pyspark.sql import functions as F

bronze_df = spark.read.table("workspace.default.taxi_trips_bronze")

silver_df = (
    bronze_df
    .filter(F.col("tpep_pickup_datetime").isNotNull())
    .filter(F.col("tpep_dropoff_datetime").isNotNull())
    .filter(F.col("tpep_dropoff_datetime") > F.col("tpep_pickup_datetime"))
    .filter((F.col("trip_distance") > 0) & (F.col("trip_distance") < 100))
    .filter((F.col("fare_amount") > 0) & (F.col("fare_amount") < 500))
    .withColumn(
        "trip_duration_minutes",
        F.round(
            (F.unix_timestamp("tpep_dropoff_datetime") -
             F.unix_timestamp("tpep_pickup_datetime")) / 60,
            2
        )
    )
    .filter(F.col("trip_duration_minutes") > 0)
    .withColumn("pickup_date", F.to_date("tpep_pickup_datetime"))
    .withColumn("pickup_hour", F.hour("tpep_pickup_datetime"))
    .withColumn("pickup_day_of_week", F.dayofweek("tpep_pickup_datetime"))
    .withColumn(
        "average_speed_mph",
        F.round(F.col("trip_distance") / (F.col("trip_duration_minutes") / 60), 2)
    )
    .withColumn("is_high_fare", (F.col("fare_amount") >= 30).cast("int"))
)

display(silver_df.limit(10))
```

အလွန်မြန်သော impossible-speed outliers များကို business rule သေချာသတ်မှတ်ပြီးမှ ဖယ်ပါ။ Demo အတွက် quality summary ကို အရင်ကြည့်နိုင်သည်။

```python
silver_df.select(
    "trip_distance", "fare_amount", "trip_duration_minutes", "average_speed_mph"
).summary().display()

silver_df.write.mode("overwrite").saveAsTable(
    "workspace.default.taxi_trips_silver"
)

assert spark.read.table("workspace.default.taxi_trips_silver").filter(
    "trip_distance <= 0 OR fare_amount <= 0 OR trip_duration_minutes <= 0"
).limit(1).count() == 0
print("Silver table validation passed")
```

## 18.5 Gold Notebook

`03_Gold/Gold_Business_Metrics` notebook တည်ဆောက်ပါ။
```python
from pyspark.sql import functions as F

silver_df = spark.read.table("workspace.default.taxi_trips_silver")

hourly_df = (
    silver_df.groupBy("pickup_hour")
    .agg(
        F.count("*").alias("total_trips"),
        F.round(F.avg("fare_amount"), 2).alias("average_fare"),
        F.round(F.avg("trip_distance"), 2).alias("average_distance"),
        F.round(F.sum("fare_amount"), 2).alias("total_fare")
    )
    .orderBy("pickup_hour")
)
hourly_df.write.mode("overwrite").saveAsTable("workspace.default.gold_hourly_metrics")

daily_df = (
    silver_df.groupBy("pickup_date")
    .agg(
        F.count("*").alias("total_trips"),
        F.round(F.avg("fare_amount"), 2).alias("average_fare"),
        F.round(F.sum("fare_amount"), 2).alias("total_fare"),
        F.round(F.avg("trip_distance"), 2).alias("average_distance")
    )
    .orderBy("pickup_date")
)
daily_df.write.mode("overwrite").saveAsTable("workspace.default.gold_daily_metrics")

route_df = (
    silver_df.groupBy("pickup_zip", "dropoff_zip")
    .agg(
        F.count("*").alias("total_trips"),
        F.round(F.avg("fare_amount"), 2).alias("average_fare"),
        F.round(F.avg("trip_distance"), 2).alias("average_distance")
    )
    .orderBy(F.desc("total_trips"))
)
route_df.write.mode("overwrite").saveAsTable("workspace.default.gold_route_metrics")

print("Gold tables created")
display(hourly_df)
```

Validation:

```python
for table_name in ["gold_hourly_metrics", "gold_daily_metrics", "gold_route_metrics"]:
    table_df = spark.read.table(f"workspace.default.{table_name}")
    print(table_name, table_df.limit(1).count(), table_df.columns)
```

## 18.6 AI/BI Dashboard

1. Sidebar → **Dashboards** → **Create Dashboard** သို့သွားပါ။
2. **Data** tab → **Add SQL dataset** ရွေးပါ။
3. Dataset တစ်ခုစီအတွက် `workspace.default.gold_hourly_metrics`, `gold_daily_metrics`, `gold_route_metrics` ကို query လုပ်ပါ။
4. Hourly trips အတွက် bar chart, daily trend အတွက် line chart, top routes အတွက် table/bar chart ထည့်ပါ။
5. Date/zip filter ထည့်၍ result ကိုစစ်ပါ။
6. Draft ကမှန်ကြောင်း verify ပြီးမှ publish လုပ်ပါ။
## 18.7 ML Demo: High-Fare Classification

ဤ model သည် MLflow workflow လေ့လာရန် educational demo ဖြစ်သည်။ Target ကို `fare_amount >= 30` ဖြင့်ဆောက်ထားသဖြင့် `fare_amount` ကို feature အဖြစ်မထည့်ရပါ။ `trip_duration_minutes` နှင့် `average_speed_mph` ကို feature အဖြစ်သုံးသောကြောင့် trip မစမီ prediction မဟုတ်ဘဲ completed-trip classification ဖြစ်ကြောင်းလည်း နားလည်ထားရမည်။

`04_ML_Models/High_Fare_Classifier` notebook တွင်:

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, roc_auc_score
from sklearn.model_selection import train_test_split

feature_columns = [
    "trip_distance", "pickup_hour", "pickup_day_of_week",
    "trip_duration_minutes", "average_speed_mph"
]
target_column = "is_high_fare"

ml_pdf = (
    spark.read.table("workspace.default.taxi_trips_silver")
    .select(*feature_columns, target_column)
    .dropna()
    .limit(50000)
    .toPandas()
)

X = ml_pdf[feature_columns]
y = ml_pdf[target_column]
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

mlflow.set_experiment("/Shared/nyc_taxi_high_fare")
with mlflow.start_run(run_name="random_forest_baseline"):
    model = RandomForestClassifier(
        n_estimators=100, max_depth=10, class_weight="balanced", random_state=42
    )
    model.fit(X_train, y_train)
    predictions = model.predict(X_test)
    probabilities = model.predict_proba(X_test)[:, 1]
    auc = roc_auc_score(y_test, probabilities)

    mlflow.log_params({"n_estimators": 100, "max_depth": 10})
    mlflow.log_metric("roc_auc", auc)
    mlflow.sklearn.log_model(model, name="model")

    print(classification_report(y_test, predictions))
    print("ROC-AUC:", auc)
```

`Experiments` UI တွင် parameters, metrics နှင့် model artifact ကိုစစ်ပါ။ Result ကောင်းသည်ဟုဆိုရန် test set တစ်ခုသာမဟုတ်ဘဲ time-based split, leakage, drift, class imbalance နှင့် business value ကိုပါ နောက်ထပ်စစ်ရမည်။

## 18.8 Lakeflow Job

Jobs UI တွင် `Bronze_Raw_Taxi_Data → Silver_Cleansed_Taxi_Data → Gold_Business_Metrics` task dependency ဆောက်ပါ။ စစမ်းချိန်တွင် **Run now** ဖြင့် manual run လုပ်ပြီး task တစ်ခုချင်း output စစ်ပါ။ Source သည် static sample table ဖြစ်သဖြင့် daily schedule ထားခြင်းသည် learning အတွက်သာဖြစ်ပြီး business value မရှိသေးပါ။ Schedule မတင်မီ repeated `overwrite` behavior, quota နှင့် expected freshness ကိုသတ်မှတ်ပါ။

Demo project အောင်မြင်မှုစစ်ဆေးရန်:

- Catalog Explorer တွင် Bronze, Silver နှင့် Gold tables ပေါ်သည်
- Silver quality assertion အောင်မြင်သည်
- Dashboard charts တွင် data ပေါ်သည်
- MLflow experiment တွင် run/metric/model artifact ပေါ်သည်
- Job tasks အစဉ်လိုက်အောင်မြင်ပြီး run history ရှိသည်
# 19. Common Errors

## `TABLE_OR_VIEW_NOT_FOUND`

- Three-level name ကို အပြည့်ရေးပါ: `workspace.default.table_name`
- `SHOW CATALOGS`, `SHOW SCHEMAS IN workspace`, `SHOW TABLES IN workspace.default` ဖြင့်စစ်ပါ။
- Notebook တစ်ခုစီတွင် variable များကို သီးခြား run ရသည်ကို သတိပြုပါ။

## `UNRESOLVED_COLUMN`

Source schema ကို မစစ်ဘဲ column name ယူဆထားခြင်းကြောင့်ဖြစ်တတ်သည်။

```python
print(raw_trips_df.columns)
raw_trips_df.printSchema()
```

NYC sample တွင် `tip_amount` မရှိသောကြောင့် tip-related code ကိုမ run ပါနှင့်။

## Permission Denied

Catalog/schema/table privileges နှင့် current user/object owner ကို Catalog Explorer တွင်စစ်ပါ။ Free Edition တွင် `workspace.default` ကို သုံးပြီး `samples` catalog ကို read-only source အဖြစ်ယူပါ။

## Quota Exceeded / Compute Unavailable

Free Edition fair-usage quota ပြည့်နိုင်သည်။ Unnecessary full counts, repeated model training နှင့် large `toPandas()` ကိုလျှော့ပါ။ Quota reset ဖြစ်သည့်အခါ ဆက်လုပ်ပါ။ Data/settings များရှိနေကြောင်း Catalog/Workspace မှ စစ်နိုင်သည်။

## Driver Out of Memory

`collect()` သို့မဟုတ် `toPandas()` မှ data အားလုံးကို driver သို့ဆွဲယူခြင်းဖြစ်နိုင်သည်။ `limit`, aggregate, sample သို့မဟုတ် distributed ML ကိုသုံးပါ။

## Slow Query

Query profile/`explain`, shuffle, skew, repeated scans, too many files, unfiltered rows နှင့် unnecessary columns ကို စစ်ပါ။ Compute ကြီးအောင်လုပ်ခြင်းမတိုင်မီ data/query design ကိုပြင်ပါ။

## Streaming Restart/Checkpoint Error

Checkpoint path ကို query မတူရာတွင် reuse လုပ်ထားခြင်း၊ schema မကိုက်ခြင်း သို့မဟုတ် source state ပြောင်းခြင်းကို စစ်ပါ။ Checkpoint ကိုဖျက်ခြင်းက data ပြန်ဖတ်ခြင်း/duplicate risk ဖြစ်နိုင်သောကြောင့် အကြောင်းရင်းမသိဘဲ မဖျက်ပါနှင့်။

## ML Metric အလွန်ကောင်းလွန်းခြင်း

Target leakage, duplicate train/test rows, random split က time leakage ဖြစ်ခြင်းနှင့် target imbalance ကိုစစ်ပါ။ Feature တစ်ခုသည် prediction အချိန်တွင် အမှန်တကယ်ရနိုင်မရနိုင် သေချာမေးပါ။

# 20. Learning Roadmap နှင့် Glossary

## ပထမအပတ်: Foundation

- Workspace, Catalog Explorer နှင့် notebook UI ကျွမ်းကျင်အောင်လုပ်ပါ
- `samples.nyctaxi.trips` ကို SQL/PySpark နှစ်မျိုးဖြင့် query လုပ်ပါ
- DataFrame select, filter, groupBy, join နှင့် null handling လေ့လာပါ
- Bronze table တည်ဆောက်ပြီး Delta history ကြည့်ပါ

## ဒုတိယအပတ်: Data Engineering

- Silver quality rules နှင့် deduplication လုပ်ပါ
- Gold aggregate tables နှင့် AI/BI dashboard တည်ဆောက်ပါ
- `MERGE`, schema enforcement/evolution နှင့် time travel လေ့လာပါ
- Lakeflow Job ဖြင့် Bronze → Silver → Gold dependency run ပါ

## တတိယအပတ်: Streaming နှင့် Governance

- Volume သို့ file upload လုပ်ပါ
- Auto Loader/Structured Streaming concepts လေ့လာပါ
- Catalog permissions, lineage, comments/tags နှင့် managed/external assets နားလည်ပါ
- Checkpoint, watermark, idempotency ကို lab လုပ်ပါ
## စတုတ္ထအပတ်နှင့်နောက်ပိုင်း: ML, Production နှင့် Azure

- MLflow experiment, model evaluation နှင့် registry လေ့လာပါ
- Git workflow နှင့် Declarative Automation Bundles concept လေ့လာပါ
- Monitoring, data quality, retries နှင့် cost governance ထည့်ပါ
- Paid Azure lab ရလျှင် Entra ID, ADLS, VNet, Secure Cluster Connectivity, Private Link နှင့် Key Vault ကို lab လုပ်ပါ

## အရေးကြီးဝေါဟာရများ

| Term | မြန်မာလို အဓိပ္ပါယ်အကျဉ်း |
|---|---|
| Lakehouse | Data lake flexibility နှင့် warehouse reliability/performance ပေါင်းထားသော architecture |
| Spark | Distributed data processing engine |
| DataFrame | Schema ပါသော distributed tabular data abstraction |
| Partition | Parallel process လုပ်သည့် data အပိုင်း |
| Shuffle | Partitions/executors ကြား data ပြန်ဝေခြင်း |
| Delta Lake | Parquet နှင့် transaction log အခြေခံ transactional table layer |
| Unity Catalog | Data/AI governance, permissions, lineage နှင့် discovery layer |
| Catalog | Three-level namespace ၏ အပေါ်ဆုံး data container |
| Schema | Catalog အတွင်း objects စုထားသော namespace |
| Table | Structured rows/columns ကို query လုပ်နိုင်သော object |
| Volume | Governed non-tabular file storage |
| Bronze | Raw, traceable ingestion layer |
| Silver | Cleaned, validated, reusable detail layer |
| Gold | Business-ready aggregate/domain layer |
| Lakeflow | Ingestion, declarative pipelines နှင့် jobs ပါသော data engineering family |
| Job | Tasks များကို dependency/schedule ဖြင့် run သည့် workflow |
| Checkpoint | Streaming progress နှင့် state record |
| MLflow | ML/AI experiment, evaluation, model နှင့် observability platform |
| Photon | SQL/DataFrame workload အတွက် vectorized execution engine |
| Idempotent | တူညီသော input ဖြင့် rerun သော်လည်း duplicate/incorrect result မဖြစ်သည့် property |
| Data lineage | Source မှ table, model, dashboard အထိ data flow record |
| Data leakage | Prediction အချိန်မရနိုင်သော information ကို training feature ထဲ ထည့်မိခြင်း |

# 21. Official References

- [What is Azure Databricks?](https://learn.microsoft.com/en-us/azure/databricks/introduction/index)
- [Azure Databricks high-level architecture](https://learn.microsoft.com/en-us/azure/databricks/getting-started/high-level-architecture)
- [Databricks Free Edition](https://learn.microsoft.com/en-us/azure/databricks/getting-started/free-edition)
- [Free Edition limitations](https://learn.microsoft.com/en-us/azure/databricks/getting-started/free-edition-limitations)
- [Apache Spark on Azure Databricks](https://learn.microsoft.com/en-us/azure/databricks/spark/)
- [Delta Lake](https://learn.microsoft.com/en-us/azure/databricks/delta/)
- [Unity Catalog](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/)
- [Compute](https://learn.microsoft.com/en-us/azure/databricks/compute/)
- [Medallion architecture](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/medallion)
- [Auto Loader](https://learn.microsoft.com/en-us/azure/databricks/ingestion/cloud-object-storage/auto-loader/)
- [Lakeflow pipeline concepts](https://learn.microsoft.com/en-us/azure/databricks/ldp/concepts)
- [Lakeflow Jobs](https://learn.microsoft.com/en-us/azure/databricks/workflows/jobs/jobs/)
- [AI/BI dashboard tutorial](https://learn.microsoft.com/en-us/azure/databricks/dashboards/tutorials/create-dashboard)
- [MLflow on Databricks](https://learn.microsoft.com/en-us/azure/databricks/mlflow/)
- [Security](https://learn.microsoft.com/en-us/azure/databricks/security/)
- [Cost optimization best practices](https://learn.microsoft.com/en-us/azure/databricks/lakehouse-architecture/cost-optimization/best-practices)
- [Declarative Automation Bundles](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/bundles/)

> **Licensing note:** External documentation was summarized and paraphrased rather than copied. Content was rephrased for compliance with licensing restrictions. Feature names, limits and availability can change; verify the linked official documentation before production decisions.

---

**ပြီးဆုံးမှုမှတ်ချက်**: သင့်လက်ရှိ lab ၏ နောက်တစ်ဆင့်မှာ `Bronze_Raw_Taxi_Data` notebook တွင် Section 18.2 ရှိ Cell 2 ကို run၍ source columns ခြောက်ခုကို အတည်ပြုခြင်းဖြစ်သည်။ ထို့နောက် Section 18.3 အတိုင်း Bronze Delta table ကို တည်ဆောက်ပါ။
