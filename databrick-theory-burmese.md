# Azure Databricks အခြေခံအယူအဆများ (Burmese)

## မာတိကာ

1. [Azure Databricks ဆိုတာဘာလဲ](#azure-databricks-ဆိုတာဘာလဲ)
2. [အဓိကအစိတ်အပိုင်းများ](#အဓိကအစိတ်အပိုင်းများ)
3. [Architecture ဖွဲ့စည်းပုံ](#architecture-ဖွဲ့စည်းပုံ)
4. [Core Concepts](#core-concepts)
5. [Delta Lake](#delta-lake)
6. [Unity Catalog](#unity-catalog)
7. [Compute Options](#compute-options)
8. [Data Pipeline Architecture](#data-pipeline-architecture)
9. [MLflow](#mlflow)
10. [Best Practices](#best-practices)

---

## Azure Databricks ဆိုတာဘာလဲ

### အဓိပ္ပါယ်

**Azure Databricks** ဆိုတာ Microsoft Azure ပေါ်မှာ run တဲ့ unified analytics platform တစ်ခုဖြစ်ပါတယ်။ Apache Spark ကို base လုပ်ထားတဲ့ data processing platform တစ်ခုဖြစ်ပြီး data engineering, data science, machine learning နဲ့ analytics အလုပ်တွေကို တစ်နေရာတည်းမှာ လုပ်ဆောင်နိုင်ပါတယ်။

### ဘယ်လိုဖြစ်လာသလဲ

- **Databricks Company**: Apache Spark ကိုဖန်တီးခဲ့တဲ့ UC Berkeley မှာပါဝင်ခဲ့တဲ့သူတွေက တည်ထောင်ခဲ့တယ်
- **Azure Integration**: Microsoft နဲ့ Databricks ပူးပေါင်းထားတဲ့ managed service
- **Launch Year**: 2018 မှာ Azure ပေါ်မှာစတင်ခဲ့တယ်

### အဓိက Feature များ

| Feature | ရှင်းလင်းချက် |
|---------|--------------|
| **Unified Platform** | Data engineering, ML, analytics တစ်နေရာတည်းမှာ |
| **Apache Spark** | Big data processing engine |
| **Lakehouse Architecture** | Data lake + Data warehouse ပေါင်းစပ်ထားတယ် |
| **Managed Service** | Infrastructure ကို Azure က manage ပေးတယ် |
| **Collaboration** | Notebooks, dashboards, jobs တွေကို share လုပ်နိုင်တယ် |
| **MLflow Integration** | Machine learning lifecycle ကို manage လုပ်တယ် |

---

## အဓိကအစိတ်အပိုင်းများ

### 1. Workspace

**Workspace** ဆိုတာ Databricks မှာအလုပ်လုပ်တဲ့ environment ဖြစ်ပါတယ်။

**Workspace မှာပါဝင်တာတွေ:**
- Notebooks (code files)
- Jobs (scheduled tasks)
- Clusters (compute resources)
- Dashboards
- Models (ML models)
- Workflows

### 2. Notebook

**Notebook** ဆိုတာ interactive coding environment ဖြစ်ပါတယ်။ Jupyter notebook နဲ့ ဆင်တူပါတယ်။

**Support လုပ်တဲ့ Languages:**
- **Python** - Data science, ML အတွက်အသုံးများ
- **SQL** - Data querying
- **Scala** - Spark native language
- **R** - Statistical analysis

**Notebook Cell Types:**
```
┌─────────────────────────────────┐
│  Code Cell                      │
│  - Code ရေးသားရန်               │
│  - Run လုပ်လို့ရတယ်               │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│  Markdown Cell                  │
│  - Documentation                │
│  - Instructions                 │
│  - Explanations                 │
└─────────────────────────────────┘
```

### 3. Cluster

**Cluster** ဆိုတာ data processing လုပ်ဖို့ compute resources တွေပါဝင်တဲ့ system ဖြစ်ပါတယ်။

**Cluster Types:**

| Type | အသုံးပြုမှု | ရှင်းလင်းချက် |
|------|-----------|--------------|
| **Interactive Cluster** | Development, exploration | Manual creation, shared use |
| **Job Cluster** | Production jobs | Auto-created for jobs |
| **Serverless Cluster** | Quick queries, SQL | No management needed |

**Cluster Components:**
```
┌──────────────────────────────────────┐
│           Driver Node                │
│  - Coordinates work                  │
│  - Manages Spark context             │
│  - Collects results                  │
└──────────────────────────────────────┘
              │
              ▼
┌──────────────────────────────────────┐
│         Worker Nodes (1-N)           │
│  - Execute tasks                     │
│  - Process data                      │
│  - Store partitions                  │
└──────────────────────────────────────┘
```

### 4. Job

**Job** ဆိုတာ notebook သို့မဟုတ် code တစ်ခုကို schedule လုပ်ပြီး auto-run လုပ်တာဖြစ်ပါတယ်။

**Job Features:**
- Schedule (cron-based)
- Retry policies
- Email alerts
- Multiple tasks
- Dependencies between tasks

---

## Architecture ဖွဲ့စည်းပုံ

### High-Level Architecture

```
┌────────────────────────────────────────────────────────┐
│              AZURE DATABRICKS ACCOUNT                   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │            CONTROL PLANE                          │  │
│  │   (Managed by Databricks in Azure)               │  │
│  │                                                   │  │
│  │  • Web Application UI                            │  │
│  │  • Notebook Management                           │  │
│  │  • Job Scheduler                                 │  │
│  │  • Security & Access Control                     │  │
│  │  • Unity Catalog Metastore                       │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
                           │
                           │
        ┌──────────────────┴──────────────────┐
        │                                      │
        ▼                                      ▼
┌───────────────────┐              ┌───────────────────┐
│ SERVERLESS        │              │ CLASSIC           │
│ COMPUTE PLANE     │              │ COMPUTE PLANE     │
│                   │              │                   │
│ • Fast startup    │              │ • Your Azure      │
│ • No management   │              │   subscription    │
│ • Auto-scale      │              │ • Full control    │
│ • SQL, ML serving │              │ • Custom configs  │
└───────────────────┘              └───────────────────┘
        │                                      │
        └──────────────────┬──────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   STORAGE LAYER        │
              │                        │
              │ • Azure Data Lake      │
              │   Storage Gen2         │
              │ • Delta Tables         │
              │ • Unity Catalog        │
              └────────────────────────┘
```

### Control Plane vs Compute Plane

**Control Plane (Databricks မှ manage):**
- Web UI
- Job scheduler
- Notebook management
- Security services
- Unity Catalog

**Compute Plane (Data processing လုပ်တယ်):**

| Serverless | Classic |
|------------|---------|
| Databricks account မှာ run | Customer's Azure subscription မှာ run |
| Seconds အတွင်း startup | 5-10 minutes startup |
| Auto-scale | Manual/Auto-scale |
| Higher cost per hour | Lower cost per hour |
| No infrastructure management | Full infrastructure control |

### Workspace Hierarchy

```
Azure Databricks Account
│
├── Workspaces
│   ├── Workspace 1 (Development)
│   │   ├── Notebooks
│   │   ├── Jobs
│   │   ├── Clusters
│   │   └── Dashboards
│   │
│   ├── Workspace 2 (Production)
│   │   ├── Notebooks
│   │   ├── Jobs
│   │   ├── Clusters
│   │   └── Dashboards
│   │
│   └── Workspace 3 (Testing)
│       └── ...
│
└── Unity Catalog Metastore
    ├── Catalog 1
    │   ├── Schema (Database)
    │   │   ├── Tables
    │   │   ├── Volumes
    │   │   └── Functions
    │   └── Schema 2
    │
    └── Catalog 2
        └── ...
```

---

## Core Concepts

### 1. Apache Spark

**Apache Spark** ဆိုတာ big data processing အတွက် open-source distributed computing framework ဖြစ်ပါတယ်။

**Spark Components:**

```
┌─────────────────────────────────────────┐
│         APACHE SPARK STACK              │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │   Spark SQL & DataFrames        │   │
│  │   - Structured data processing  │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │   Spark Streaming               │   │
│  │   - Real-time processing        │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │   MLlib (Machine Learning)      │   │
│  │   - ML algorithms               │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │   GraphX (Graph Processing)     │   │
│  │   - Graph algorithms            │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │   Spark Core                    │   │
│  │   - Task scheduling             │   │
│  │   - Memory management           │   │
│  │   - Fault recovery              │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Spark Execution Flow:**

```
User Code (Python/Scala/SQL)
         │
         ▼
    Spark Driver
         │
         ├─→ Parse Code
         ├─→ Create Execution Plan
         ├─→ Optimize Plan
         └─→ Distribute Tasks
              │
              ▼
         Worker Nodes
              │
              ├─→ Execute Tasks
              ├─→ Process Data Partitions
              └─→ Return Results
                   │
                   ▼
              Driver (Collect Results)
```

### 2. DataFrame

**DataFrame** ဆိုတာ distributed collection of data organized into named columns ဖြစ်ပါတယ်။ Database table တစ်ခုနဲ့ ဆင်တူပါတယ်။

**DataFrame Characteristics:**
- Immutable (ပြောင်းလဲ၍မရ)
- Lazy evaluation (action မခေါ်မချင်း execute မလုပ်)
- Distributed processing
- Schema support
- Optimization by Spark

**DataFrame Operations:**

```python
# Transformations (Lazy)
df.filter(col("age") > 25)        # Filter rows
df.select("name", "age")          # Select columns
df.groupBy("department").count()  # Aggregation
df.join(df2, "id")                # Join tables

# Actions (Trigger execution)
df.show()                         # Display data
df.count()                        # Count rows
df.collect()                      # Get all data
df.write.saveAsTable("table")     # Save to table
```

### 3. RDD vs DataFrame vs Dataset

| Feature | RDD | DataFrame | Dataset |
|---------|-----|-----------|---------|
| Type Safety | Compile-time | Runtime | Compile-time |
| Performance | Low | High | High |
| Ease of Use | Difficult | Easy | Moderate |
| Optimization | Manual | Automatic | Automatic |
| Python Support | Yes | Yes | No (Scala/Java only) |

---

## Delta Lake

### Delta Lake ဆိုတာဘာလဲ

**Delta Lake** ဆိုတာ data lake ပေါ်မှာ reliable data pipelines တွေ build လုပ်ဖို့ open-source storage layer ဖြစ်ပါတယ်။

### Delta Lake Features

| Feature | ရှင်းလင်းချက် | အကျိုးကျေးဇူး |
|---------|--------------|--------------|
| **ACID Transactions** | Atomic, Consistent, Isolated, Durable | Data reliability ရှိတယ် |
| **Time Travel** | Historical data versions ကို query လုပ်နိုင်တယ် | Audit, rollback လုပ်နိုင်တယ် |
| **Schema Evolution** | Schema ပြောင်းလဲနိုင်တယ် | Flexible data structure |
| **Upsert/Delete** | INSERT, UPDATE, DELETE, MERGE operations | Data modifications လုပ်နိုင်တယ် |
| **Compaction** | Small files တွေကို combine လုပ်တယ် | Better query performance |
| **ZORDER** | Data clustering by columns | Faster queries |

### Delta Table Structure

```
┌─────────────────────────────────────┐
│         Delta Table                  │
├─────────────────────────────────────┤
│                                     │
│  Data Files (Parquet)               │
│  ├── part-00000.parquet            │
│  ├── part-00001.parquet            │
│  └── part-00002.parquet            │
│                                     │
│  Transaction Log (_delta_log)        │
│  ├── 00000000000000000000.json     │ ← Version 0
│  ├── 00000000000000000001.json     │ ← Version 1
│  └── 00000000000000000002.json     │ ← Version 2
│                                     │
└─────────────────────────────────────┘
```

### Delta Operations

**1. Create Delta Table:**
```python
# From DataFrame
df.write.format("delta").saveAsTable("my_table")

# From path
df.write.format("delta").save("/path/to/delta/table")
```

**2. Read Delta Table:**
```python
# From table name
df = spark.read.table("my_table")

# From path
df = spark.read.format("delta").load("/path/to/delta/table")
```

**3. Update/Delete/Merge:**
```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(spark, "my_table")

# Update
delta_table.update(
    condition = "age < 18",
    set = { "category": "'minor'" }
)

# Delete
delta_table.delete("status = 'inactive'")

# Merge (Upsert)
delta_table.alias("target").merge(
    new_data.alias("source"),
    "target.id = source.id"
).whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()
```

**4. Time Travel:**
```python
# Version ကို query လုပ်တယ်
df_version_1 = spark.read.format("delta") \
    .option("versionAsOf", 1) \
    .load("/path/to/table")

# Timestamp ကို query လုပ်တယ်
df_yesterday = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-01") \
    .load("/path/to/table")
```

### Delta Lake Advantages

**Traditional Data Lake Problems:**
```
❌ No ACID transactions
❌ Small file problem
❌ Schema enforcement issues
❌ No time travel
❌ Concurrent write conflicts
```

**Delta Lake Solutions:**
```
✅ Full ACID support
✅ Auto-compaction
✅ Schema enforcement & evolution
✅ Time travel capability
✅ Optimistic concurrency control
```

---

## Unity Catalog

### Unity Catalog ဆိုတာဘာလဲ

**Unity Catalog** ဆိုတာ Azure Databricks မှာ data နဲ့ AI assets တွေကို centrally govern လုပ်ဖို့ solution ဖြစ်ပါတယ်။

### Unity Catalog Hierarchy

```
┌─────────────────────────────────────────────┐
│          UNITY CATALOG METASTORE             │
│          (Top-level governance)              │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │Catalog 1│ │Catalog 2│ │Catalog 3│
   │(Dev)    │ │(Prod)   │ │(Shared) │
   └─────────┘ └─────────┘ └─────────┘
        │
        ├── Schema (Database)
        │   ├── Tables
        │   ├── Views
        │   ├── Volumes
        │   ├── Functions
        │   └── Models
        │
        └── Schema 2
            └── ...
```

### Three-Level Namespace

Unity Catalog မှာ data objects တွေကို 3-level namespace နဲ့ address လုပ်ပါတယ်:

```
catalog.schema.object

ဥပမာ:
sales_db.orders.transactions
│         │       │
│         │       └── Table/View/Function
│         └─────────── Schema (Database)
└───────────────────── Catalog
```

### Unity Catalog Features

| Feature | ရှင်းလင်းချက် |
|---------|--------------|
| **Centralized Governance** | တစ်နေရာတည်းမှာ permissions manage လုပ်တယ် |
| **Fine-grained Access Control** | Table, row, column level permissions |
| **Data Lineage** | Data flow ကို track လုပ်တယ် |
| **Audit Logging** | Data access တွေကို log လုပ်တယ် |
| **Secure Data Sharing** | Data share လုပ်တယ် safely |
| **Credential Passthrough** | Azure AD credentials ကို use လုပ်တယ် |

### Permission Model

```
┌─────────────────────────────────────────┐
│         Permission Hierarchy             │
├─────────────────────────────────────────┤
│                                         │
│  Metastore Level                        │
│  ├── CREATE CATALOG                     │
│  └── CREATE SCHEMA                      │
│                                         │
│  Catalog Level                          │
│  ├── USE CATALOG                        │
│  ├── CREATE SCHEMA                      │
│  └── ALL PRIVILEGES                     │
│                                         │
│  Schema Level                           │
│  ├── USE SCHEMA                         │
│  ├── CREATE TABLE                       │
│  ├── CREATE FUNCTION                    │
│  └── ALL PRIVILEGES                     │
│                                         │
│  Table Level                            │
│  ├── SELECT (read)                      │
│  ├── MODIFY (write)                     │
│  └── ALL PRIVILEGES                     │
│                                         │
└─────────────────────────────────────────┘
```

### Unity Catalog Benefits

**Before Unity Catalog:**
```
❌ Per-workspace permissions
❌ No central view of data
❌ Manual permission management
❌ No data lineage tracking
❌ Difficult data sharing
```

**With Unity Catalog:**
```
✅ Single pane of glass for governance
✅ Unified permissions across workspaces
✅ Automated lineage tracking
✅ Secure data sharing
✅ Compliance & audit support
```

---

## Compute Options

### Compute Types Comparison

| Feature | Serverless | Interactive | Job Clusters |
|---------|-----------|-------------|--------------|
| **Startup Time** | < 30 seconds | 5-10 minutes | 5-10 minutes |
| **Use Case** | SQL, quick queries | Development | Production jobs |
| **Management** | Zero management | Manual setup | Auto for jobs |
| **Cost** | Higher per DBU | Running cost | Only when job runs |
| **Auto-scale** | Yes | Yes | Yes |
| **Auto-terminate** | N/A | Yes (configurable) | Yes (after job) |

### Serverless Compute

**Pros:**
- Instant startup
- No cluster management
- Auto-scaling
- Cost-effective for short queries

**Cons:**
- Higher per-DBU cost
- Limited customization
- Not for long-running jobs

**Best For:**
- SQL queries
- Dashboard queries
- Quick data exploration
- Notebooks with short computations

### Interactive Clusters

**Pros:**
- Full control over configuration
- Can install custom libraries
- Shared among team members
- Good for development

**Cons:**
- Manual setup required
- 5-10 minute startup time
- Running cost even when idle
- Need to manage termination

**Best For:**
- Development work
- Data exploration
- Testing
- Team collaboration

### Job Clusters

**Pros:**
- Created automatically for jobs
- Terminated after job completion
- Cost-effective for scheduled work
- Auto-scaling

**Cons:**
- Not for interactive use
- Recreated each run
- 5-10 minute startup

**Best For:**
- Production ETL pipelines
- Scheduled jobs
- Automated workflows
- Batch processing

### Cluster Sizing Guide

```
┌──────────────────────────────────────────────┐
│         CLUSTER SIZING MATRIX                │
├──────────────────────────────────────────────┤
│                                              │
│  Small Workload (< 10GB)                     │
│  ├── Driver: Standard_DS3_v2 (4 cores)      │
│  ├── Workers: 2-4 Standard_DS3_v2           │
│  └── Memory: 14GB per node                  │
│                                              │
│  Medium Workload (10-100GB)                  │
│  ├── Driver: Standard_DS4_v2 (8 cores)      │
│  ├── Workers: 4-8 Standard_DS4_v2           │
│  └── Memory: 28GB per node                  │
│                                              │
│  Large Workload (> 100GB)                    │
│  ├── Driver: Standard_DS5_v2 (16 cores)     │
│  ├── Workers: 8-16 Standard_DS5_v2          │
│  └── Memory: 56GB per node                  │
│                                              │
└──────────────────────────────────────────────┘
```

### Auto-scaling Configuration

```
Min Workers: 2  ──┐
                  │
                  │  Auto-scale
                  │  Range
                  │
Max Workers: 10 ──┘

Auto-scale Logic:
├── CPU/Memory utilization > 75% → Add workers
├── CPU/Memory utilization < 50% → Remove workers
└── Scale down waits 10 minutes before removing
```

### Spot Instances

**Spot Instances** ဆိုတာ Azure ရဲ့ spare computing capacity ကို discount နဲ့ ငှါးရတဲ့ VM type ဖြစ်ပါတယ်။

| Spot Instance | Regular Instance |
|---------------|------------------|
| Up to 90% cheaper | Full price |
| Can be evicted | Guaranteed availability |
| Good for stateless jobs | Required for critical jobs |
| Use for worker nodes | Use for driver node |

**Best Practices for Spot:**
- Always have on-demand workers as backup
- Use for worker nodes only (not driver)
- Implement checkpointing in jobs
- Handle graceful termination

---

## Data Pipeline Architecture

### Medallion Architecture (Bronze-Silver-Gold)

**Medallion Architecture** ဆိုတာ data ကို layer ၃ ခုဖြင့် organize လုပ်တဲ့ pattern ဖြစ်ပါတယ်။

```
┌─────────────────────────────────────────────────┐
│                BRONZE LAYER                      │
│                (Raw Data)                        │
│                                                  │
│  • Raw data from sources                         │
│  • Append-only                                   │
│  • Full history preserved                        │
│  • Minimal transformations                       │
│                                                  │
│  Tables:                                         │
│  ├── raw_customer_data                          │
│  ├── raw_transactions                           │
│  └── raw_events                                 │
└─────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│                SILVER LAYER                      │
│                (Cleansed Data)                   │
│                                                  │
│  • Data quality checks                           │
│  • Deduplication                                 │
│  • Standardization                               │
│  • Business logic applied                        │
│                                                  │
│  Tables:                                         │
│  ├── customers_clean                            │
│  ├── transactions_clean                         │
│  └── events_clean                               │
└─────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│                GOLD LAYER                        │
│                (Business-Ready)                  │
│                                                  │
│  • Aggregated metrics                            │
│  • KPIs                                          │
│  • Feature tables for ML                         │
│  • Optimized for queries                         │
│                                                  │
│  Tables:                                         │
│  ├── daily_revenue_summary                      │
│  ├── customer_360_view                          │
│  └── ml_feature_store                           │
└─────────────────────────────────────────────────┘
```

### Layer Details

**Bronze Layer:**
```
Purpose: Raw data preservation
──────────────────────────────────
✓ Ingest data as-is
✓ Add ingestion metadata (timestamp, source)
✓ No data quality filtering
✓ Append-only (no deletes/updates)
✓ Full audit trail
✓ Quick recovery from source

Example Schema:
├── <source_columns>
├── ingestion_timestamp
├── ingestion_date
├── source_file_name
└── source_system
```

**Silver Layer:**
```
Purpose: Cleansed, validated data
──────────────────────────────────
✓ Data quality checks
✓ Remove duplicates
✓ Handle null values
✓ Standardize formats
✓ Join reference data
✓ Add business columns
✓ Data validation rules

Example Transformations:
├── Remove invalid records
├── Deduplicate by primary key
├── Enrich with derived columns
└── Standardize date formats
```

**Gold Layer:**
```
Purpose: Business-ready data products
──────────────────────────────────
✓ Aggregated metrics
✓ KPI calculations
✓ Business-specific views
✓ ML feature tables
✓ Optimized for dashboards
✓ Star schema design

Example Tables:
├── Fact tables (metrics)
├── Dimension tables (descriptive)
├── Summary tables (aggregations)
└── Feature tables (ML)
```

### Data Flow Example

```
Source System (CSV Files)
         │
         ▼
    ┌────────────────┐
    │   Auto Loader  │  ← Incremental ingestion
    └────────────────┘
         │
         ▼
    Bronze Table (Raw)
         │
         ├─ Remove invalid records
         ├─ Deduplicate
         ├─ Enrich columns
         └─ Standardize formats
         │
         ▼
    Silver Table (Clean)
         │
         ├─ Aggregate metrics
         ├─ Create summaries
         └─ Build KPIs
         │
         ▼
    Gold Tables (Business-Ready)
         │
         ├─ Dashboards
         ├─ Reports
         └─ ML Models
```

### Delta Live Tables (DLT)

**Delta Live Tables (DLT)** ဆိုတာ data pipelines တွေကို declare လုပ်ဖို့ framework ဖြစ်ပါတယ်။

**DLT Benefits:**
- Declarative pipeline definition
- Automatic dependency management
- Built-in data quality checks
- Auto-scaling infrastructure
- Error handling & recovery

**DLT Example:**
```python
import dlt

# Bronze table
@dlt.table(
    comment="Raw taxi trip data"
)
def bronze_trips():
    return spark.readStream.format("cloudFiles") \
        .option("cloudFiles.format", "json") \
        .load("/path/to/raw/trips")

# Silver table with quality expectations
@dlt.table(
    comment="Cleansed trip data"
)
@dlt.expect("valid_fare", "fare_amount >= 0")
@dlt.expect("valid_distance", "trip_distance > 0")
def silver_trips():
    return dlt.read_stream("bronze_trips") \
        .filter("fare_amount > 0") \
        .filter("trip_distance > 0")

# Gold table
@dlt.table(
    comment="Daily trip metrics"
)
def gold_daily_metrics():
    return dlt.read("silver_trips") \
        .groupBy("trip_date") \
        .agg(
            count("*").alias("total_trips"),
            sum("fare_amount").alias("total_revenue")
        )
```

---

## MLflow

### MLflow ဆိုတာဘာလဲ

**MLflow** ဆိုတာ machine learning lifecycle ကို manage လုပ်ဖို့ open-source platform ဖြစ်ပါတယ်။

### MLflow Components

```
┌─────────────────────────────────────────────┐
│              MLFLOW COMPONENTS               │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  MLflow Tracking                    │   │
│  │  • Experiment tracking              │   │
│  │  • Parameter logging                │   │
│  │  • Metric logging                   │   │
│  │  • Artifact storage                 │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  MLflow Projects                    │   │
│  │  • Package data science code        │   │
│  │  • Reproducible runs                │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  MLflow Models                      │   │
│  │  • Model packaging                  │   │
│  │  • Deployment tools                 │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │  MLflow Model Registry              │   │
│  │  • Model versioning                 │   │
│  │  • Stage transitions                │   │
│  │  • Model lineage                    │   │
│  └─────────────────────────────────────┘   │
│                                             │
└─────────────────────────────────────────────┘
```

### MLflow Tracking

**Experiment tracking workflow:**

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier

# Set experiment
mlflow.set_experiment("/my_experiment")

# Start run
with mlflow.start_run():
    
    # Log parameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)
    
    # Train model
    model = RandomForestClassifier(n_estimators=100, max_depth=10)
    model.fit(X_train, y_train)
    
    # Calculate metrics
    accuracy = model.score(X_test, y_test)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    
    # Log model
    mlflow.sklearn.log_model(model, "model")
    
    # Log artifacts (plots, files)
    mlflow.log_artifact("confusion_matrix.png")
```

### MLflow Model Registry

**Model lifecycle:**

```
┌─────────────────────────────────────────────┐
│         MODEL REGISTRY WORKFLOW             │
└─────────────────────────────────────────────┘

1. Register Model
   └─→ Model registered in registry

2. Staging
   └─→ Test model in staging environment

3. Production
   └─→ Deploy to production

4. Archive
   └─→ Archive old versions


Example Commands:
──────────────────────────────────
# Register model
mlflow.register_model(
    "runs:/<run_id>/model",
    "my_model_name"
)

# Transition model stage
client = mlflow.tracking.MlflowClient()
client.transition_model_version_stage(
    name="my_model_name",
    version=1,
    stage="Production"
)

# Load production model
model = mlflow.sklearn.load_model(
    "models:/my_model_name/Production"
)
```

### MLflow in Databricks

**Databricks MLflow Benefits:**
- Pre-installed and integrated
- Automatic logging
- UI for experiment management
- Model serving capabilities
- Integration with Unity Catalog

**Autologging:**
```python
import mlflow

# Enable autologging
mlflow.autolog()

# All sklearn operations automatically logged
model = RandomForestClassifier()
model.fit(X_train, y_train)
# ↑ Parameters, metrics, model automatically logged
```

---

## Best Practices

### 1. Data Organization

```
Workspace Structure:
──────────────────────────────────
project_name/
├── 01_bronze/
│   └── ingest_raw_data.py
├── 02_silver/
│   └── transform_data.py
├── 03_gold/
│   └── create_metrics.py
├── dashboards/
│   └── executive_dashboard.py
├── ml_models/
│   └── train_model.py
└── README.md

Catalog Structure:
──────────────────────────────────
catalog_name/
├── bronze/
│   ├── raw_customers
│   └── raw_transactions
├── silver/
│   ├── customers_clean
│   └── transactions_clean
└── gold/
    ├── customer_metrics
    └── daily_revenue
```

### 2. Cluster Management

```
✓ DO:
──────────────────────────────────
• Use auto-termination (10-30 min)
• Enable auto-scaling
• Use spot instances for workers
• Use job clusters for production
• Monitor cluster metrics
• Right-size clusters

✗ DON'T:
──────────────────────────────────
• Leave interactive clusters running
• Over-provision resources
• Use interactive clusters for jobs
• Ignore cluster logs
• Skip monitoring
```

### 3. Code Best Practices

```
✓ Parameterize Everything:
──────────────────────────────────
# Bad
df = spark.read.csv("/fixed/path/data.csv")

# Good
data_path = dbutils.widgets.text("data_path", "/default/path")
df = spark.read.csv(data_path)

✓ Use Functions:
──────────────────────────────────
def process_data(df, filters):
    """Process data with given filters"""
    for col, value in filters.items():
        df = df.filter(df[col] == value)
    return df

✓ Add Documentation:
──────────────────────────────────
"""
Data Processing Pipeline

This notebook processes raw transaction data
and creates aggregated metrics for dashboards.

Author: Data Team
Date: 2024-01-01
"""

✓ Handle Errors:
──────────────────────────────────
try:
    df = spark.read.table("my_table")
except Exception as e:
    print(f"Error reading table: {e}")
    # Fallback logic
    df = spark.read.csv("/backup/data.csv")
```

### 4. Security Best Practices

```
✓ Enable Unity Catalog:
──────────────────────────────────
• Use Unity Catalog from day 1
• Implement least-privilege access
• Regular access reviews
• Use service principals for jobs

✓ Secret Management:
──────────────────────────────────
# Don't hardcode secrets
password = "my_password"  # ❌ BAD

# Use secret scopes
password = dbutils.secrets.get(
    scope="my_scope",
    key="password"
)  # ✓ GOOD

✓ Network Security:
──────────────────────────────────
• Use VNet injection
• Enable private links
• Restrict outbound access
• Use managed identity
```

### 5. Performance Optimization

```
✓ Delta Optimization:
──────────────────────────────────
# Optimize table (compaction)
spark.sql("OPTIMIZE my_table")

# Z-order by frequently queried columns
spark.sql("OPTIMIZE my_table ZORDER BY (date, category)")

✓ Caching:
──────────────────────────────────
# Cache frequently used data
df.cache()

# Unpersist when done
df.unpersist()

✓ Partitioning:
──────────────────────────────────
# Partition by date
df.write.partitionBy("date").saveAsTable("my_table")

# Don't over-partition
# Rule: Partition size > 1GB
```

### 6. Cost Optimization

```
Cost Saving Strategies:
──────────────────────────────────

1. Use Spot Instances
   └─→ Up to 90% savings on compute

2. Auto-termination
   └─→ 10-30 min idle timeout

3. Job Clusters
   └─→ Only pay when running

4. Serverless for Short Queries
   └─→ No minimum run time

5. Photon Engine
   └─→ Faster queries = less compute time

6. Right-size Clusters
   └─→ Monitor and adjust

7. Delta Cache
   └─→ Reduce data scanning

8. Partition Pruning
   └─→ Read only needed data
```

---

## Summary

### Key Takeaways

**Azure Databricks ဆိုတာ:**
- Unified analytics platform
- Built on Apache Spark
- Lakehouse architecture
- Managed by Azure

**အဓိက Components:**
- Workspace (working environment)
- Notebooks (code editors)
- Clusters (compute resources)
- Jobs (automation)
- Unity Catalog (governance)

**Data Architecture:**
- Bronze Layer (raw data)
- Silver Layer (cleansed data)
- Gold Layer (business-ready)

**Key Technologies:**
- Delta Lake (reliable storage)
- MLflow (ML lifecycle)
- Unity Catalog (governance)
- Spark (data processing)

**Best Practices:**
- Use medallion architecture
- Implement Unity Catalog early
- Optimize clusters for cost
- Follow security guidelines
- Monitor and iterate

---

## ဆက်လက်လေ့လာရန်

**Official Documentation:**
- [Azure Databricks Docs](https://learn.microsoft.com/en-us/azure/databricks/)
- [Databricks Academy](https://www.databricks.com/training)

**Free Resources:**
- Databricks Free Edition
- Azure 14-day trial
- Community forums

**Certifications:**
- Databricks Data Engineer Associate
- Databricks Data Engineer Professional
- Databricks ML Professional

---

**Document Version**: 1.0  
**Language**: Burmese (Myanmar)  
**Last Updated**: November 2026

This document provides a theoretical foundation for Azure Databricks concepts in Burmese language.
